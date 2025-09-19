(The file `d:\code\CAD\README.md` exists, but is empty)
# Le Cercle des Amis Disparus — SaaS Starter

Plateforme sociale pour réduire l'isolement en reliant des personnes via des activités géolocalisées et en soutenant les commerçants locaux.

## Résumé
Ce dépôt contient la base technique et les conventions pour construire « Le Cercle des Amis Disparus » : une application Next.js (App Router) en TypeScript strict, PostgreSQL + Drizzle ORM, authentification JWT via `jose`, paiements Stripe, UI avec Tailwind/shadcn et validation Zod.

Le design produit (MVP) couvre : authentification & profils, création et recherche d'activités géolocalisées, chat basique, interface commerçants et système de réservation simple.

## Contenu du dépôt
- `app/` — Next.js App Router (pages, routes, actions)
- `components/` — Composants UI réutilisables (ex: `components/ui`)
- `lib/` — Logique métier, accès DB, utils (ex: `lib/db`, `lib/auth`, `lib/payments`)
- `app/api/` — Routes API (ex: Stripe webhooks, checkout)
- `drizzle.config.ts` — Configuration Drizzle
- `.github/instructions/RULES.instructions.md` — Règles et bonnes pratiques à respecter

## Règles à respecter (extraits essentiels)
- TypeScript strict (`tsconfig.json`: `strict: true`, `noEmit: true`).
- Valider toutes les entrées serveur avec Zod.
- Auth : JWT signés via `jose`, cookie `httpOnly`, `secure`, `sameSite: 'lax'`.
- DB : Drizzle centralisé dans `lib/db/drizzle.ts`, soft-delete (`deletedAt`) partout.
- Stripe : vérifier signatures des webhooks (`STRIPE_WEBHOOK_SECRET`).
- Naming : kebab-case pour routes, PascalCase pour composants.

Consulte `.github/instructions/RULES.instructions.md` pour la liste complète.

## Tech stack
- Framework : Next.js (App Router)
- Langage : TypeScript (strict)
- DB : PostgreSQL + Drizzle ORM
- Auth : JWT (`jose`) + cookies
- Paiements : Stripe
- UI : Tailwind CSS + shadcn/ui + `cva`
- Validation : Zod
- Déploiement cible : Vercel

## Prérequis locaux
- Node.js >= 18
- pnpm (recommandé) ou npm
- Postgres (local ou cloud)

## Variables d'environnement (exigées)
Créez un fichier `.env` (ou utilisez des secrets plateforme) avec au minimum :

- `POSTGRES_URL` — URI de connexion Postgres
- `AUTH_SECRET` — clé pour signer les JWT
- `STRIPE_SECRET_KEY` — clé API Stripe
- `STRIPE_WEBHOOK_SECRET` — secret pour vérifier les webhooks
- `BASE_URL` — URL publique (ex : https://example.com)

Fichier exemple : `.env.example` (à créer)

## Scripts utiles (dans package.json)
- `pnpm dev` — démarre Next.js en mode dev
- `pnpm build` — build pour production
- `pnpm start` — démarre le serveur en production
- `pnpm db:setup` — script d'initialisation DB (lib/db/setup.ts)
- `pnpm db:seed` — remplir la DB d'exemples
- `pnpm db:migrate` — migrations Drizzle

## Quickstart local

1. Installer les dépendances :

```powershell
pnpm install
```

2. Créer `.env` à partir de `.env.example` et renseigner les variables.

3. Initialiser la base et seed (si nécessaire) :

```powershell
pnpm db:setup; pnpm db:seed
```

4. Lancer le serveur dev :

```powershell
pnpm dev
```

5. Ouvrir `http://localhost:3000`

## Structure recommandée pour démarrer le MVP
- Auth & profils (login, sign-up, session cookie)
- Activities (création, recherche géolocalisée, soft-delete)
- Chat basique (messages liés aux activités)
- Merchants (offres, deals)
- Paiements (Stripe checkout + webhooks)

## Roadmap MVP (extrait)
- Phase 1 (3 mois) : Auth, profils, création/recherche activités, chat basique, interface commerçants, réservation simple.
- Phase 2 : Module "Les Invisibles", système de cadeaux, notifications push, améliorations mobile.

## Tests & CI recommandés
- Ajouter lint, typecheck et tests unitaires pour `lib/*` et actions server.
- CI minimal proposé : GitHub Actions — job `typecheck` + `pnpm build` + `pnpm test`.

## Contribuer
- Respecter `.github/instructions/RULES.instructions.md` : conventions de code, sécurité, validation et patterns.
- Ouvrir des branches descriptives (`feature/xxx`, `fix/xxx`) et PRs claires avec description, screenshots et checklist.

## Prochaines étapes (je peux les faire pour toi)
1. Rédiger le backlog minimal et les 3 milestones (auth/profils, geo-activities, chat/merchants).
2. Scaffolder le MVP (routes de base, schéma Drizzle minimal, actions serveur, seed minimal).
3. Ajouter un workflow GitHub Actions pour typecheck/build/test.

Dis‑moi quelle étape tu veux que je lance en premier et je m'en occupe (je mettrai à jour la todo list et j'exécuterai les actions correspondantes).

