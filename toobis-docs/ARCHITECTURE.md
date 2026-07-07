# ARCHITECTURE — Règles non négociables (Athena)

> Document de référence. Toute génération de code doit le respecter.
> Les règles ci-dessous sont vérifiées par ArchUnit (voir src/test/.../architecture).

## 1. Sens des dépendances
- Flux autorisé : web → application → domain ← infrastructure
- `domain` ne dépend de RIEN d'autre que la JDK et le code `domain`.
- Aucun import sous `**/domain/**` de : org.springframework, jakarta.persistence,
  com.fasterxml.jackson, lombok. (Violation = build cassé.)

## 2. Domaine
- Records ou classes finales immuables. Factory methods validantes.
- equals/hashCode sur la valeur pour tous les Value Objects.
- Identité entre aggregates par VO d'ID (UUID encapsulé), JAMAIS par référence objet.
  Ex : Property porte un ManagerId, pas un Manager.
- Deux VO de mot de passe distincts : RawPassword (transitoire, validé, jamais persisté
  ni loggé) et HashedPassword (résultat du PasswordEncoderPort, seul persisté).

## 3. Flux command/query
- Request DTO (web) → Command/Query (application) → UseCase (port in) → Domain
  → Repository Port (port out) → Persistence Adapter → Entity → Domain → Response DTO.
- Un Request DTO n'atteint JAMAIS un UseCase.
- Écritures via Command, lectures via Query.

## 4. Structure d'une feature (à cloner à l'identique)
<feature>/
  application/{port/in, port/out, command, query, dto, usecase}
  domain/{model, valueobject, repository, service, exception}
  infrastructure/{persistence, mapper, adapter, security}
  web/{controller, request, response, advice}

## 5. Inter-features
- Pas d'accès direct au repo d'une autre feature.
- Toute dépendance cross-feature passe par un port out (interface côté feature
  appelante, adapter d'implémentation côté infrastructure).

## 6. Lombok
- Autorisé UNIQUEMENT dans web et infrastructure. Interdit dans domain et application.

## 7. Pièges techniques connus
- Ordre des annotation processors : Lombok PUIS MapStruct dans maven-compiler-plugin.
- VO à factory privée : fournir explicitement les convertisseurs MapStruct
  (@Named/default), réutilisés via uses = {...}. Aucun mapper ne compile sans eux.
- OptimisticLockingFailureException → HTTP 409 dans le GlobalExceptionHandler.

## 8. Sécurité
- Interdit : User implements UserDetails. Passer par UserPrincipal (infrastructure).
- JWT HMAC HS256. Refresh token persisté + rotation + révocation au logout.
- Permissions métier converties en GrantedAuthority pour @PreAuthorize.

## 9. Schéma de base & stratégie de test (décision assumée)
- Flyway est la SOURCE DE VÉRITÉ du schéma en production (PostgreSQL),
  via db/migration/V1__baseline.sql (schéma cible complet dès le départ).
- En prod : spring.jpa.hibernate.ddl-auto=validate (Hibernate vérifie, ne modifie pas).
- Limite connue d'H2 : ne supporte pas l'index unique partiel
  (CREATE UNIQUE INDEX ... WHERE status='ACTIVE'). Donc :
  - Tests @DataJpaTest : tournent sur H2 avec ddl-auto=create-drop, Flyway DÉSACTIVÉ.
    Ils valident le MAPPING et les REQUÊTES, PAS le schéma Flyway de production.
  - Validation du schéma Flyway réel : DÉLÉGUÉE aux tests d'intégration Testcontainers
    (PostgreSQL + Flyway activé), à mettre en place en itération 3.
- CONSÉQUENCE À RETENIR : une erreur dans V1__baseline.sql ne sera PAS attrapée
  par les @DataJpaTest. Ne jamais considérer le vert des slices comme une validation
  du schéma de prod.
- TODO itération 3 : isoler la désactivation Flyway (actuellement globale dans
  src/test/resources/application.yml) pour qu'elle ne s'applique PAS au profil
  d'intégration Testcontainers.

## 10. Dette technique connue (à arbitrer)
- Génération du refresh token (SecureRandom, 32 octets) actuellement dans la couche
  application (LoginUseCaseImpl, RefreshTokenUseCaseImpl). Acceptable (JDK pur, aucune
  dépendance framework), mais le "comment générer de l'aléa" relève plutôt d'un port
  infrastructure. À refactorer derrière un TokenGeneratorPort si purisme souhaité.
  Non bloquant. ArchUnit ne le détecte pas (SecureRandom n'est pas un framework).

## 11. Convention de découpage des controllers (autorisation)

Règle de décision pour scinder ou non les controllers d'une feature :

- **Deux controllers** quand les URLs "self" et "admin" DIFFÈRENT.
  L'appelant agit sur lui-même via /me/* (id pris dans le token, jamais dans l'URL),
  un admin agit sur une cible via /{id}. Pas de collision de route possible.
  → Cas `user` : UserMeController (/api/v1/users/me/*, authenticated)
                 + UserAdminController (/api/v1/users/{id}, /api/v1/users, hasAuthority('ROLE_ADMIN')).
  Chaque controller a une politique d'autorisation HOMOGÈNE (toute la classe = même niveau).

- **Un seul controller** quand l'URL est PARTAGÉE entre admin et propriétaire.
  Admin et propriétaire visent le même chemin /{id} avec une condition d'accès différente.
  Séparer en deux classes créerait une collision de route (même chemin, deux @Mapping).
  → Cas à venir `property`/`manager`/`rentalOffer` : un controller, autorisation PAR MÉTHODE
    via @PreAuthorize combinant rôle ET ownership, ex. :
    @PreAuthorize("hasAuthority('ROLE_ADMIN') or @<feature>Security.isOwner(#id, authentication)")
  L'ownership est porté par un @Component de sécurité dédié par feature (infrastructure),
  qui charge la ressource et compare son propriétaire à l'utilisateur authentifié.

Principe directeur : on sépare par DIVERGENCE D'URL, pas par dogme "un controller par rôle".
L'autorisation reste exprimée avec hasAuthority('ROLE_...') (convention du projet, pas hasRole).

## 12. Rôle transverse de la feature `auth`

La feature `auth` joue un double rôle : c'est une feature (endpoints login/logout/refresh)
ET un fournisseur d'infrastructure d'authentification pour toute l'application.

`auth.infrastructure.security` contient SecurityConfiguration (chaîne de filtres,
requestMatchers, policy stateless), le filtre JWT, les handlers entry-point/access-denied,
UserPrincipal et DomainUserDetailsService. Ces composants sécurisent TOUS les controllers
de l'application.

Décision (analysée) : on NE déplace PAS cette config vers shared. Raisons :
- SecurityConfiguration est densément couplée aux mécanismes d'auth (filtre JWT, providers,
  handlers). La mettre dans shared forcerait shared à dépendre d'auth → inversion de
  dépendance interdite.
- Vérifié : AUCUNE feature hors auth ne référence auth.infrastructure.security dans son
  code. Les controllers SUBISSENT la sécurité (via la chaîne de filtres + @PreAuthorize)
  sans en DÉPENDRE à la compilation. L'encapsulation est donc déjà correcte.

Conclusion : auth fournit la sécurité de façon transverse, c'est légitime. La config y
reste. Une dépendance "feature X → auth" pour l'authentification est acceptable et
attendue (sens normal). L'inverse (auth → X) reste interdit.

## 13. Versioning de l'API

Toute l'API est exposée sous le préfixe `/api/v1` (endpoints `user` ET `auth`).

**Principe : préfixe CENTRALISÉ, controllers ignorants.**
- La constante `shared.web.ApiVersion.V1 = "/api/v1"` est la source de vérité.
- `shared.infrastructure.configuration.WebMvcConfiguration` applique ce préfixe à tous les
  `@RestController` du projet via `PathMatchConfigurer.addPathPrefix(ApiVersion.V1, predicate)`.
- Les `@RequestMapping` des controllers restent relatifs (ex. `/users`, `/auth`) — ils ne
  connaissent pas `/api/v1`.

**Conséquence pour les tests** : `@WebMvcTest` charge les `WebMvcConfigurer` beans, donc le
préfixe est actif dans les slices de test. Toutes les URLs de test utilisent `/api/v1/...`.

**Pour passer à v2** : changer `ApiVersion.V1` (ou ajouter `ApiVersion.V2`) et reconfigurer
`WebMvcConfiguration`. Un seul point de changement ; les controllers ne bougent pas.

## 14. Email (transverse)

L'envoi d'email est découpé en deux responsabilités distinctes.

**Transport (shared)** — `shared.application.port.out.EmailSenderPort` :
```
void send(EmailVO to, String subject, String htmlBody)
```
Interface générique, sans aucun concept métier. Implémentée par
`shared.infrastructure.email.GmailEmailAdapter` (SMTP, Spring Mail). Réutilisable par
toute feature ayant besoin d'envoyer un email (user, property, manager…).

**Composition (feature)** — chaque feature qui envoie un email est responsable de son
contenu (sujet, corps HTML, lien éventuel). Ex. : `user.application.usecase.VerificationEmailComposer`
(package-private) compose le sujet et le corps du mail de vérification ; le use case construit
le lien et appelle `EmailSenderPort.send(...)`.

**Règle** : `shared` ne dépend d'AUCUN concept métier des features. Le flux est :
```
UseCase (feature) → compose sujet + corps → EmailSenderPort (shared) → GmailEmailAdapter
```

Aucune règle ArchUnit actuelle ne vérifie que `shared.infrastructure` ne dépend pas d'une
feature. À surveiller si `shared` grandit.