# Règles de code — SaaS Starter

Ce document liste des règles concises, professionnelles et applicables au code présent dans ce dépôt (Next.js App Router, TypeScript, Drizzle ORM, Stripe, Tailwind, etc.). Elles visent à assurer cohérence, sécurité et maintenabilité.

## Format général
- Utiliser TypeScript strict (tsconfig.json : "strict": true). Toujours typer les exports publics.
- Nommer les fichiers et exports en kebab-case pour les routes/pages et en camelCase/UpperCamelCase pour les composants et types.
- Pas d'émission de JS compilé dans le dépôt (tsconfig.json : "noEmit": true).

## Architecture et organisation
- Structure : utiliser l'App Router (`app/`) ; placer routes API dans `app/api/*/route.ts`.
- Composants UI réutilisables dans `components/ui/` ; fonctions métier et accès DB dans `lib/`.
- Les fonctions serveur (actions) doivent être marquées `use server` si utilisées côté serveur et valider leurs entrées (ex. zod).

## Composants UI
- Composants présentent des props types via `React.ComponentProps<"x">` ou types explicites.
- Garder les composants purs et sans effets secondaires; appels API serveur se font via actions ou via `lib/`.
- Utiliser utilitaires partagés (ex. `cn` dans `lib/utils`) pour concaténation de classes.

## Données et DB (Drizzle)
- Centraliser la connexion DB dans `lib/db/drizzle.ts`; importer `db` et `schema`.
- Éviter les suppressions physiques ; préférer soft-delete (champ `deletedAt`) quand c'est implémenté.
- Utiliser les helpers `db.select()/insert()/update()/delete()` ou `db.query.*` selon besoins; limiter les SELECT * non filtrés et toujours `limit` quand attendu.

## Authentification et sessions
- Utiliser des tokens JWT signés via `jose` (voir `lib/auth/session.ts`) ; clé depuis `process.env.AUTH_SECRET`.
- Stocker le token en cookie HTTP-only, Secure et SameSite=lax (`cookies().set(..., { httpOnly: true, secure: true, sameSite: 'lax' })`).
- Vérifier systématiquement l'expiration du token et la présence des champs attendus avant d'accéder à l'utilisateur.

## Validation et sécurité d'entrée
- Valider toutes les entrées utilisateurs côté serveur (ex. `zod` dans `app/(login)/actions.ts`).
- Pour les actions publiques utiliser un middleware de validation/autorisation (`validatedAction`, `validatedActionWithUser`).
- Ne jamais faire confiance aux données provenant de Stripe ou d'URL sans vérification (ex. vérifier `session.customer`, `session.subscription`, signature webhook via `stripe.webhooks.constructEvent`).

## Paiements (Stripe)
- Initialiser le client Stripe une fois dans `lib/payments/stripe.ts` avec `process.env.STRIPE_SECRET_KEY` et version d'API.
- Vérifier la signature des webhooks (utiliser `STRIPE_WEBHOOK_SECRET`).
- Après un checkout réussi, enrichir la DB (customerId, subscriptionId, productId, status) et créer/mettre à jour la session utilisateur.

## Journalisation et audit
- Utiliser une table d'activité (activityLogs) pour enregistrer actions importantes (SIGN_IN, SIGN_OUT, SIGN_UP, UPDATE_PASSWORD, DELETE_ACCOUNT, etc.).
- Logguer côté serveur les erreurs critiques (ex. verification webhook, échecs Stripe) avec `console.error` et gérer les status HTTP appropriés.

## Sécurité opérationnelle
- Charger les variables sensibles via `.env` et `dotenv` uniquement côté serveur (ne pas exposer AUTH_SECRET, STRIPE_SECRET_KEY côté client).
- Cookies de session : httpOnly, secure et expiry court (1 jour dans l'implémentation actuelle).

## Patterns et bonnes pratiques SQL
- Préférer les requêtes paramétrées via Drizzle (ex. `eq`, `and`, `isNull`) — évite l'injection SQL.
- Pour les mises à jour sensibles, utiliser `returning()` quand on a besoin des entités créées.

## Tests et qualité
- Compiler avec TypeScript strict localement ; ajouter tests unitaires pour la logique métier (lib/*) et pour les actions.
- Ajouter des checks automatiques (lint, typecheck) avant merge — (recommandé : pipeline CI exécutant `pnpm build` et `pnpm test`).

## Style et conventions de code
- Respecter l'usage des utilitaires existants : `cn` pour classes, `cva` + `VariantProps` pour variantes UI.
- Préférer `async/await` et `Promise.all` pour opérations concurrentes quand c'est sûr.
- Éviter les valeurs magiques ; extraire constantes d'environnement dans `process.env` et centraliser l'usage.

## CI / Déploiement
- Les scripts npm fournis sont `dev`, `build`, `start`, et commandes Drizzle (`db:setup`, `db:seed`, `db:migrate`).
- Sur production, exiger la configuration des variables : POSTGRES_URL, AUTH_SECRET, STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET, BASE_URL.

## Règles opératoires rapides (cheat-sheet)
- Toujours valider et typer les inputs (Zod + TypeScript).
- JWT en cookie httpOnly + vérification d'expiration avant usage.
- Vérification de la signature Stripe pour les webhooks.
- Soft-delete des utilisateurs (ne pas supprimer physiquement).

---

Fichiers analysés (exemples) : `package.json`, `tsconfig.json`, `next.config.ts`, `drizzle.config.ts`, `app/layout.tsx`, `components/ui/*`, `lib/db/*`, `lib/auth/*`, `app/(login)/actions.ts`, `app/api/stripe/*`, `lib/payments/stripe.ts`.

Si tu veux, je peux :
- Ajouter des règles supplémentaires (ex : tests recommandés, coverage minimale, conventions Git commit/PR),
- Générer des scripts CI (GitHub Actions) minimaux pour lint/type/test,
- Ou traduire ce fichier en anglais.
