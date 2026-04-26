# Workflow Claude Code + VS Code — Développement pas à pas
## Parapheur Digital : de la spec au code, session par session
**Guide pratique d'interaction — Solo Developer — Avril 2026**

---

## AVANT DE COMMENCER : Qu'est-ce que Claude Code ?

Claude Code est un **agent de développement** qui s'exécute dans votre terminal ou dans VS Code. Contrairement à ChatGPT ou Claude.ai (interface web), Claude Code :

- **Lit vos fichiers** directement (specs, code existant, tests)
- **Écrit et modifie** du code dans votre projet
- **Exécute des commandes** terminal (npm, git, prisma, vitest...)
- **Voit votre arborescence** et navigue dedans
- **Itère en autonomie** : il peut enchaîner lecture → écriture → test → correction

Vous n'avez pas besoin de copier-coller du code. Claude Code travaille *dans* votre projet.

---

## PARTIE 1 — INSTALLATION ET CONFIGURATION

### Étape 1.1 — Installer Claude Code

```bash
# Dans votre terminal
npm install -g @anthropic-ai/claude-code

# Vérifier l'installation
claude --version
```

### Étape 1.2 — Connecter votre compte Anthropic

```bash
claude login
# Un navigateur s'ouvre → connectez-vous avec votre compte Anthropic
# Le token est sauvegardé automatiquement
```

### Étape 1.3 — Ouvrir VS Code avec Claude Code

```bash
# Aller dans votre dossier de projet
mkdir parapheur-digital-app
cd parapheur-digital-app
git clone https://github.com/tkassi/parapheur-digital.git specs
# (les specs sont dans ./specs/)

# Ouvrir VS Code
code .

# Dans le terminal VS Code (Ctrl+` pour l'ouvrir), lancer Claude Code
claude
```

Vous verrez apparaître le prompt interactif :
```
Claude Code — Anthropic
Working directory: /Users/vous/parapheur-digital-app
> _
```

### Étape 1.4 — Configurer CLAUDE.md (mémoire du projet)

Claude Code lit automatiquement un fichier `CLAUDE.md` à la racine. C'est sa **mémoire permanente** du projet.

**Dans le terminal Claude Code, tapez :**
```
/init
```

Claude va générer un `CLAUDE.md` de base. Ensuite, demandez-lui de le compléter :

```
Lis le fichier specs/PHASE4-V2-architecture-integration-MAROC-SOLO.md 
et specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md, puis mets à jour 
CLAUDE.md avec :
- Le nom du projet et son contexte (banque marocaine, Loi 43-20)
- La stack technique (Node.js 20 + Fastify + TypeScript + Prisma + React)
- La structure monorepo (apps/ + packages/)
- Les 5 microservices et leurs responsabilités
- Les conventions de code (pas d'any, Zod validation, commits conventionnels)
```

Claude va lire les specs et générer un CLAUDE.md complet. **Ce fichier est lu à chaque session** — vous n'aurez plus besoin de réexpliquer le contexte.

---

## PARTIE 2 — PATTERN D'UNE SESSION DE DÉVELOPPEMENT

Chaque session suit ce pattern en 5 temps :

```
1. CONTEXTE    → Donner la spec à Claude (une seule fois par composant)
2. PLAN        → Demander le plan avant le code
3. CODE        → Faire écrire le code fichier par fichier
4. TEST        → Faire écrire et lancer les tests
5. REVIEW      → Demander une review, corriger, itérer
```

---

## PARTIE 3 — SESSION 1 : INITIALISATION DU MONOREPO (2h)

### Ouvrir Claude Code et démarrer

```bash
cd /Users/vous/parapheur-digital-app
claude
```

### Interaction 1 — Créer la structure monorepo

**Vous tapez :**
```
Lis le fichier specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md section 3 
(Initialisation du monorepo). 

Crée la structure complète :
- package.json racine avec pnpm workspaces
- pnpm-workspace.yaml
- turbo.json
- .prettierrc
- packages/shared-config/tsconfig.base.json
- packages/shared-config/eslint-config/index.js
- .gitignore approprié pour Node.js + TypeScript

Ne génère pas encore les apps/, seulement la structure de base.
```

Claude va **créer tous ces fichiers** dans votre dossier. Vous voyez les fichiers apparaître dans l'explorateur VS Code en temps réel.

### Interaction 2 — Docker Compose

```
Crée maintenant le docker-compose.yml pour l'environnement de développement 
avec PostgreSQL 16, Redis 7, et Maildev. 
Utilise les credentials du fichier .env.example dans les specs.
Crée aussi .env.example et .env.local (gitignored).
```

### Interaction 3 — Vérification

```
Lance docker-compose up -d et vérifie que les 3 services démarrent correctement.
Montre-moi le résultat de docker-compose ps.
```

Claude va **exécuter la commande** dans votre terminal et vous montrer le résultat.

### Interaction 4 — Installer les dépendances

```
Installe les dépendances racine avec pnpm install.
Vérifie qu'il n'y a pas d'erreurs.
```

**Résultat attendu de la session 1 :** structure monorepo fonctionnelle, Docker up, dépendances installées. ~2 heures.

---

## PARTIE 4 — SESSION 2 : BASE DE DONNÉES PRISMA (3h)

### Ouvrir une nouvelle session

```bash
claude
```

> **Note :** Claude Code relit CLAUDE.md automatiquement → il connaît déjà le contexte.

### Interaction 1 — Créer le package shared-db

```
Lis specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md section 4 (Base de données).

Initialise le package packages/shared-db :
1. Crée le package.json avec les dépendances prisma et @prisma/client
2. Initialise Prisma (npx prisma init)
3. Crée le schéma Prisma complet tel que décrit dans la section 4.2 du guide
   avec tous les modèles : User, Delegation, Dossier, Document, 
   WorkflowInstance, ValidationTask, WorkflowTransition, AuditLog
   et tous les enums : AuthLevel, DossierType, DossierStatus, TaskStatus, etc.
```

### Interaction 2 — Lancer la migration

```
Lance la première migration Prisma :
cd packages/shared-db && npx prisma migrate dev --name init

Ensuite crée le fichier de migration SQL pour le full-text search 
et les règles append-only sur audit_logs (section 4.3 du guide).
```

### Interaction 3 — Vérifier le schéma

```
Ouvre Prisma Studio pour que je puisse vérifier le schéma visuellement :
npx prisma studio
```

Claude lance Prisma Studio sur http://localhost:5555. Vérifiez les tables dans votre navigateur.

### Interaction 4 — Créer le seed

```
Crée le fichier packages/shared-db/prisma/seed.ts avec les 4 utilisateurs 
de test (Khalid N6, Aïcha N4, Mohammed N2, Yassir AUDITEUR) 
tels que définis dans specs/PHASE1-V2-specifications-detaillees-MAROC-SOLO.md 
section personas.

Lance ensuite le seed : npx prisma db seed
```

### Interaction 5 — Générer les types

```
Génère le client Prisma : npx prisma generate
Vérifie que les types TypeScript sont bien générés dans node_modules/@prisma/client
```

**Résultat attendu de la session 2 :** base de données PostgreSQL avec schéma complet, 4 utilisateurs de test, types TypeScript générés.

---

## PARTIE 5 — SESSION 3 : API GATEWAY + AUTHENTIFICATION (4h)

### Interaction 1 — Plan avant le code

```
Lis specs/PHASE1-V2-specifications-detaillees-MAROC-SOLO.md section 
"User Story US-01 : Authentification AD + MFA OTP".

Lis aussi specs/GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md section 5.

Donne-moi le plan de développement de l'API Gateway :
- Quels fichiers tu vas créer ?
- Dans quel ordre ?
- Quelles dépendances npm ?
Ne code rien encore, juste le plan.
```

**Attendez et lisez le plan.** Si quelque chose ne convient pas, corrigez avant de continuer. C'est le moment d'ajuster.

### Interaction 2 — Initialiser l'app

```
Plan approuvé. Commence par :
1. Créer apps/api-gateway/package.json avec toutes les dépendances
2. Créer apps/api-gateway/tsconfig.json (hérite de shared-config)
3. Créer apps/api-gateway/src/config.ts avec Zod (valide les variables d'env)
4. Créer apps/api-gateway/src/app.ts (Fastify avec plugins CORS, Helmet, 
   Rate limit, JWT)

Seulement ces 4 fichiers pour l'instant.
```

> **Conseil :** demandez toujours **4-5 fichiers maximum** par interaction. Claude est plus précis et vous pouvez vérifier au fur et à mesure.

### Interaction 3 — Route de login LDAP

```
Crée maintenant la route d'authentification :
- apps/api-gateway/src/routes/auth/login.ts
  (POST /auth/login → bind LDAP → JWT avec level)
- apps/api-gateway/src/routes/auth/totp.ts  
  (POST /auth/totp/verify → valide le code TOTP 6 chiffres)
- apps/api-gateway/src/middleware/authenticate.ts
  (hook Fastify qui vérifie le JWT sur toutes les routes protégées)

Pour le LDAP, utilise les variables LDAP_URL, LDAP_BASE_DN du .env.local.
```

### Interaction 4 — Test de la route login

```
Crée le fichier de test apps/api-gateway/src/__tests__/auth.test.ts avec Vitest.

Teste :
1. Login avec credentials valides → reçoit JWT + requireTotp: true pour N4
2. Login avec mauvais password → reçoit 401
3. Login avec username trop court → reçoit 400 (validation Zod)
4. Vérification TOTP valide → reçoit JWT final (totpVerified: true)
5. Vérification TOTP invalide → reçoit 401

Mock le client LDAP (ne pas appeler un vrai AD en test).
Lance les tests : pnpm --filter api-gateway test
```

Claude va écrire les tests ET les lancer. Vous verrez le résultat directement :

```
✓ auth > login valide → JWT (45ms)
✓ auth > mauvais password → 401 (12ms)
✓ auth > username court → 400 (8ms)
✓ auth > TOTP valide → JWT final (23ms)
✓ auth > TOTP invalide → 401 (11ms)

5 tests passed
```

### Interaction 5 — Si un test échoue

```
Le test "TOTP valide → JWT final" échoue avec cette erreur :
[coller l'erreur]

Analyse et corrige.
```

Claude va lire l'erreur, identifier la cause, modifier le code, et relancer les tests.

### Interaction 6 — Point d'entrée et démarrage

```
Crée apps/api-gateway/src/index.ts (point d'entrée).
Lance l'API : pnpm --filter api-gateway dev
Teste avec curl que GET /health répond 200.
```

**Résultat attendu session 3 :** API Gateway démarre sur port 3000, auth LDAP + TOTP fonctionnels, 5 tests verts.

---

## PARTIE 6 — SESSION 4 : MS-1 SERVICE DOSSIER (4h)

### Interaction 1 — Lire la spec métier

```
Lis specs/PHASE2-V2-specifications-metier-MAROC-SOLO.md section 
"Types de dossiers et volumétrie" et "Règles de gestion Dossier".

Lis aussi specs/PHASE1-V2-specifications-detaillees-MAROC-SOLO.md 
section "Epics B — Dossiers (US-05 à US-12)".

Résume en 10 bullets ce que MS-1 doit faire, selon ces specs.
```

Cette étape force Claude à **synthétiser les specs** avant de coder. Vous validez la compréhension.

### Interaction 2 — Repository pattern

```
Crée apps/ms-dossier/src/repositories/DossierRepository.ts

Ce repository doit exposer :
- create(data: CreateDossierInput): Promise<Dossier>
- findById(id: string): Promise<Dossier | null>
- findAll(filters: DossierFilters, pagination: Pagination): Promise<PaginatedResult<Dossier>>
- update(id: string, data: UpdateDossierInput): Promise<Dossier>
- findByReference(ref: string): Promise<Dossier | null>

Utilise le client Prisma de @parapheur/db.
Toutes les méthodes sont typées strictement (pas d'any).
```

### Interaction 3 — Service métier

```
Crée apps/ms-dossier/src/services/DossierService.ts

Ce service utilise DossierRepository et implémente :
- createDossier() : génère la référence DOS-YYYY-NNNN, convertit la devise via 
  l'API BAM (cache Redis 1h), publie un event BullMQ 'dossier.created'
- submitDossier() : vérifie que le statut est BROUILLON, publie 'dossier.submitted'
- cancelDossier() : vérifie les permissions (seul le créateur peut annuler)
- duplicateDossier() : copie un dossier sans ses documents ni son workflow

Pour la conversion devise, utilise specs/PHASE2-V2 section "Multi-devise BAM".
```

### Interaction 4 — Routes Fastify

```
Crée les routes Fastify pour MS-1 :
- POST   /dossiers          → createDossier
- GET    /dossiers          → findAll (avec pagination + filtres)
- GET    /dossiers/:id      → findById
- PATCH  /dossiers/:id      → update (brouillon seulement)
- POST   /dossiers/:id/submit   → submitDossier
- POST   /dossiers/:id/cancel   → cancelDossier
- POST   /dossiers/:id/duplicate → duplicateDossier

Chaque route a un schéma Zod pour validation du body et des params.
Codes d'erreur : 400 (validation), 401 (non auth), 403 (interdit), 
404 (not found), 422 (business rule violation).
```

### Interaction 5 — Tests du service

```
Crée apps/ms-dossier/src/__tests__/DossierService.test.ts

Teste :
1. Création dossier MAD 800 000 → référence générée DOS-2026-0001
2. Création dossier EUR → conversion MAD (mock API BAM rate 10.85)
3. Soumission dossier BROUILLON → statut SOUMIS + event publié
4. Soumission dossier déjà SOUMIS → erreur 422
5. Annulation par le créateur → OK
6. Annulation par un autre user → erreur 403
7. Duplication → nouveau dossier BROUILLON sans workflow

Coverage cible : 90% sur DossierService.
Lance : pnpm --filter ms-dossier test --coverage
```

### Interaction 6 — Corriger jusqu'au vert

```
[Si des tests échouent, copier l'output ici]
Corrige jusqu'à ce que tous les tests passent.
```

**Résultat attendu session 4 :** MS-1 complet, 7 endpoints REST, 7 tests verts, coverage ≥90%.

---

## PARTIE 7 — SESSION 5 : MS-2 WORKFLOW XSTATE (la plus complexe — 2 jours)

> **Conseil :** Prévoyez 2 sessions distinctes pour ce microservice. C'est le cœur métier.

### Session 5a — Machine XState (jour 1)

#### Interaction 1 — Lire les règles DMN

```
Lis specs/PHASE2-V2-specifications-metier-MAROC-SOLO.md section complète 
"87 règles DMN" et la section "Hiérarchie d'autorisation N1-N6".

Ensuite lis specs/PHASE3-V2-conformite-securite-MAROC-SOLO.md section 
"Loi 43-20 — Niveaux de signature SES/SEA/SEQ".

Explique-moi avec tes propres mots :
1. Comment se calcule la chaîne de validation pour un crédit de 800 000 MAD ?
2. Quel type de signature est requis ?
3. Qui valide dans quel ordre ?
```

Vous validez que Claude a bien compris les règles AVANT qu'il les code.

#### Interaction 2 — Fonction computeValidationChain

```
Crée packages/shared-schemas/src/dmnRules.ts

Cette fonction computeValidationChain(montantMad: number, type: DossierType) 
retourne un tableau de ValidationStep[] selon les règles suivantes 
[détailler les règles clés ou lui demander de les extraire des specs].

Crée aussi les tests packages/shared-schemas/src/__tests__/dmnRules.test.ts
avec 15 cas de test couvrant toutes les frontières de montant :
- 50 000 MAD (non-engagement) → SES N5
- 80 000 MAD (achat) → SES N5  
- 100 001 MAD (crédit) → SEA N5
- 549 999 MAD → SEA N5
- 550 001 MAD → SEA N4
- 2 200 001 MAD → SEA N3 + N4
- 5 000 001 MAD → double signature
- 11 000 001 MAD → N2 requis
- 50 000 001 MAD → N1 requis
... etc.

Lance les tests et assure-toi que les 15 passent.
```

#### Interaction 3 — Machine XState de base

```
Crée apps/ms-workflow/src/machines/dossierWorkflow.machine.ts

Machine XState 5 avec :
- États : brouillon, soumis, attente_valideur, en_attente_info, approuve, rejete
- Events : SUBMIT, APPROVE, REJECT, REQUEST_INFO, RESPOND_INFO, CANCEL
- Context : dossierId, montantMad, type, validationChain, currentStep, decisions

La machine doit être sérialisable en JSON pour persistence PostgreSQL.
Utilise computeValidationChain pour initialiser le context.

Pas encore les guards ni les actions, juste la structure des états et transitions.
```

#### Interaction 4 — Ajouter les guards et actions

```
Ajoute maintenant les guards à la machine :
- validationComplete : currentStep >= validationChain.length
- isCurrentValideur : l'acteur est bien le valideur attendu à cette étape
- canEscalate : le délai SLA est dépassé de 50%+

Et les actions :
- assignToCurrentValideur : crée une ValidationTask en base
- recordApproval : enregistre la décision, incrémente currentStep
- calculateSlaDeadline : N5→4h, N4→8h, N3→1j, N2→2j, N1→3j (jours ouvrés Maroc)

Utilise le calendrier marocain (weekend sam-dim + jours fériés) pour les SLA.
```

### Session 5b — Service et tests (jour 2)

#### Interaction 5 — WorkflowService

```
Crée apps/ms-workflow/src/services/WorkflowService.ts

Ce service :
1. startWorkflow(dossierId) : instancie la machine XState, persiste le snapshot JSON
2. processEvent(instanceId, event, actorId) : recharge le snapshot, 
   envoie l'event, repersiste, publie les events BullMQ appropriés
3. getWorkflowState(dossierId) : retourne l'état courant + tâches en attente
4. checkSlaBreaches() : job planifié toutes les heures, escalade automatique

Le snapshot XState est stocké dans workflow_instances.machine_snapshot (JSONB).
```

#### Interaction 6 — Tests du workflow

```
Crée apps/ms-workflow/src/__tests__/WorkflowService.test.ts

Simule le scénario complet du parcours Khalid (spec PHASE1 section P1) :
1. Khalid crée dossier crédit 800 000 MAD → workflow démarré
2. Étape 1 : assigné au N4 (Aïcha) → tâche créée avec SLA 8h
3. Aïcha approuve → étape suivante ou fin ?
4. Dossier finalement APPROUVE → état final atteint

Aussi tester :
5. Aïcha rejette → dossier REJETE directement
6. Aïcha demande info → état EN_ATTENTE_INFO
7. Khalid répond → retour ATTENTE_VALIDEUR
8. SLA dépassé → escalade déclenchée

Coverage cible : 95% (ce service est critique).
```

**Résultat attendu sessions 5a+5b :** Machine XState fonctionnelle, 87 règles DMN testées, workflow complet simulé de bout en bout.

---

## PARTIE 8 — SESSION 6 : FRONTEND REACT (3 sessions)

### Session 6a — Setup + Login (2h)

#### Interaction 1 — Initialiser Vite + Tailwind

```
Initialise le frontend apps/web avec Vite + React + TypeScript.
Installe et configure :
- Tailwind CSS avec le plugin RTL (tailwindcss-rtl)
- shadcn/ui (style default, couleurs slate)
- i18next + react-i18next
- React Router DOM
- TanStack Query v5
- React Hook Form + Zod

Configure tailwind.config.ts avec les couleurs du projet :
- primary: #1A365D (bleu marine)
- accent: #DD6B20 (orange)  
- success: #38A169 (vert)
- danger: #E53E3E (rouge)
```

#### Interaction 2 — Page de login

```
Lis specs/PHASE1-V2 section "Wireframe Screen 1 — Login AD+OTP".

Crée apps/web/src/pages/Login/LoginPage.tsx :
- Formulaire username + password (React Hook Form + Zod)
- Validation : username min 3 chars, password min 8 chars
- Appel POST /api/v1/auth/login via TanStack Query mutation
- Si requireTotp: true → rediriger vers TotpPage
- Messages d'erreur en FR (i18n)
- Design : bleu marine #1A365D, logo banque centré, formulaire carte blanche

Crée aussi TotpPage.tsx avec :
- Saisie code 6 chiffres (InputOTP shadcn)
- Compte à rebours 5 minutes
- Bouton "Renvoyer le code"
```

#### Interaction 3 — Voir le résultat

```
Lance le frontend : pnpm --filter web dev
Ouvre http://localhost:5173 dans le navigateur.
Fais un screenshot de la page de login.
```

> **Note :** Claude Code peut prendre des screenshots si vous avez un navigateur ouvert. Sinon, ouvrez-le vous-même et décrivez ce que vous voyez.

### Session 6b — Dashboard + Liste dossiers (3h)

#### Interaction 1 — Dashboard

```
Lis specs/PHASE1-V2 section "Wireframe Screen 2 — Dashboard" et 
"User Story US-29 Dashboard personnel".

Crée DashboardPage.tsx avec :
- 4 cartes statistiques : À valider (rouge si >0), Mes brouillons, 
  En cours, Risque SLA (orange)
- Widget "Mes tâches urgentes" : 3 premiers dossiers à valider triés urgence
- Bouton "Nouveau dossier" (orange, proéminent)

Utilise TanStack Query pour fetcher GET /api/v1/dashboard/stats
(mock la réponse pour l'instant si le backend n'est pas encore connecté).
```

#### Interaction 2 — Liste des dossiers

```
Crée DossierListPage.tsx avec :
- Tableau avec colonnes : Référence, Type, Contrepartie, Montant MAD, 
  Statut (badge coloré), Créé le, Actions
- Barre de recherche full-text (debounce 300ms)
- Filtres : Type, Statut, Période
- Pagination (10 par page)
- Clic sur une ligne → DossierDetailPage

Utilise les types Zod de @parapheur/shared-schemas pour typer la réponse API.
```

### Session 6c — Wizard de création dossier (3h)

```
Lis specs/PHASE1-V2 section "US-05 Créer un dossier" et 
"Wireframe Screen 3 — Wizard création étape 2".

Crée DossierCreatePage.tsx — wizard 4 étapes avec barre de progression :

Étape 1 — Type et montant :
  - Select type dossier
  - Input montant avec select devise (MAD/EUR/USD)
  - Affichage automatique équivalent MAD (appel API BAM)
  - Lookup contrepartie (input + dropdown résultats en temps réel)

Étape 2 — Détails :
  - Textarea objet (bilingue optionnel)
  - Date besoin (date picker)
  - Toggle urgence
  - Toggle dérogation (avec justification obligatoire si coché)

Étape 3 — Documents :
  - Drag & drop upload (max 50MB, PDF/DOCX/XLSX/PNG)
  - Liste des documents requis selon type dossier (spec PHASE2 DMN R-DOC-*)
  - Indicateur : requis vs optionnel

Étape 4 — Récapitulatif :
  - Résumé de toutes les informations
  - Affichage de la chaîne de validation qui sera appliquée
  - Bouton "Soumettre" (appel POST /dossiers puis POST /dossiers/:id/submit)

Validation Zod à chaque étape. Le bouton "Suivant" est désactivé si l'étape est invalide.
```

---

## PARTIE 9 — PATTERN POUR LES SESSIONS SUIVANTES

Pour chaque composant restant (MS-3 Notifications, MS-4 Audit, MS-5/6 Adapters, 
page de validation, i18n arabe, tests e2e...), répétez toujours ce pattern :

### Template de prompt d'ouverture de session

```
Contexte (Claude relit CLAUDE.md automatiquement).

Aujourd'hui je travaille sur : [NOM DU COMPOSANT]

Spec de référence : lis [FICHIER SPEC] section [SECTION PRÉCISE]

État actuel du code :
- Ce qui est déjà fait : [liste]
- Ce qui manque : [liste]

Objectif de cette session :
- Livrable 1 : [fichier/fonctionnalité]
- Livrable 2 : [fichier/fonctionnalité]

Donne-moi le plan avant de commencer.
```

### Template pour demander un fichier précis

```
Crée [chemin/fichier.ts] qui :
1. [fonctionnalité 1]
2. [fonctionnalité 2]
3. [fonctionnalité 3]

Contraintes :
- Pas d'any TypeScript
- Gestion d'erreurs avec codes HTTP appropriés
- [contrainte métier spécifique]

Ensuite crée le test correspondant et lance-le.
```

### Template pour corriger une erreur

```
Cette commande [commande] produit cette erreur :

[coller l'erreur complète avec stacktrace]

Analyse la cause racine (pas juste le symptôme) et propose la correction.
Après correction, relance la commande pour vérifier.
```

### Template pour une code review

```
Fais une code review de [fichier.ts].
Critères : sécurité (OWASP), performance, lisibilité TypeScript, 
gestion d'erreurs, couverture des cas limites.
Note chaque problème : CRITIQUE / MAJEUR / MINEUR.
Propose les corrections pour les CRITIQUE et MAJEUR.
```

---

## PARTIE 10 — COMMANDES CLAUDE CODE UTILES

### Commandes slash disponibles dans Claude Code

| Commande | Usage |
|----------|-------|
| `/init` | Génère CLAUDE.md pour le projet |
| `/clear` | Efface la conversation (garde CLAUDE.md) |
| `/compact` | Compresse le contexte (utile pour longues sessions) |
| `/cost` | Affiche le coût de la session en cours |
| `/doctor` | Vérifie la configuration Claude Code |

### Référencer des fichiers dans votre prompt

```
# Lire un fichier spécifique
Lis @apps/ms-dossier/src/services/DossierService.ts et explique ce qu'il fait.

# Lire plusieurs fichiers
Lis @packages/shared-db/prisma/schema.prisma et @apps/ms-workflow/src/machines/
et explique comment le workflow persiste son état.

# Lire un dossier entier
Lis tous les fichiers dans @apps/api-gateway/src/routes/auth/
```

### Exécuter des commandes terminal

```
# Claude peut exécuter n'importe quelle commande
Lance : pnpm --filter ms-dossier test --coverage --reporter=verbose

# Il voit le résultat et peut l'analyser
Lance : pnpm --filter ms-dossier build && echo "Build OK" || echo "Build FAILED"

# Git
Lance : git diff HEAD~1 --stat
```

---

## PARTIE 11 — ORGANISATION DES 315 JH EN SESSIONS CLAUDE CODE

| Phase | Sessions | Durée | Livrables |
|-------|----------|-------|-----------|
| **P5 Setup** | 2 sessions | 1 sem | Monorepo + DB + Docker |
| **P6 Backend L1** | 6 sessions | 3 sem | MS-1 Dossier + MS-4 Audit |
| **P6 Backend L2** | 8 sessions | 4 sem | MS-2 Workflow XState (sessions longues) |
| **P7 Frontend L1** | 6 sessions | 3 sem | Auth + Dashboard + Liste + Création |
| **P7 Frontend L2** | 4 sessions | 2 sem | Validation queue + Signature modal |
| **P8 Intégrations** | 6 sessions | 3 sem | MS-3 Notifs + MS-5/6 Adapters |
| **P8 i18n + RTL** | 3 sessions | 1,5 sem | Traductions FR/AR + layout RTL |
| **P8 Mobile PWA** | 2 sessions | 1 sem | Vue mobile + offline |
| **P8 Admin** | 3 sessions | 1,5 sem | Dashboard RSSI + Audit logs + Config DMN |
| **P9 Tests e2e** | 4 sessions | 2 sem | Playwright : 5 parcours complets |
| **P9 Sécurité** | 3 sessions | 1,5 sem | OWASP ZAP + SAST + pentest basique |
| **P9 Conformité** | 2 sessions | 1 sem | CNDP + BAM audit trail |
| **P10 CI/CD** | 2 sessions | 1 sem | GitHub Actions + Docker prod |
| **P10 Monitoring** | 2 sessions | 1 sem | Prometheus + Grafana + alertes |
| **Total** | **~53 sessions** | **~27 semaines** | Application complète |

> **Rythme recommandé :** 2 sessions/jour (matin + après-midi), 5 jours/semaine.  
> Une "session" = 2 à 4 heures de travail avec Claude Code.

---

## PARTIE 12 — RÉSOLUTION DES PROBLÈMES COURANTS

### Problème : Claude génère du code qui ne compile pas

```
Ce code ne compile pas. Voici l'erreur TypeScript :
[erreur tsc]

Corrige le problème de types. N'utilise pas 'any' comme solution de facilité.
```

### Problème : Claude "oublie" le contexte après une longue session

```
# Option 1 : compresser le contexte
/compact

# Option 2 : rappeler le contexte clé
Pour rappel, le projet utilise :
- Fastify 4 + TypeScript (pas Express)
- Prisma 5 + PostgreSQL (pas de SQL raw)  
- Zod pour toutes les validations (pas de joi, pas de yup)
- Pas d'any TypeScript nulle part
```

### Problème : Les tests passent mais le comportement en dev est différent

```
Les tests passent mais quand je teste manuellement avec curl :
curl -X POST http://localhost:3001/dossiers -H "Content-Type: application/json" \
  -d '{"type":"ENGAGEMENT_CREDIT","montant":800000,"devise":"MAD",...}'

J'obtiens : [erreur]
Mais mon test mocke [ce composant]. Le problème vient probablement de [X].
Analyse et corrige.
```

### Problème : Performance lente sur une route

```
La route GET /dossiers est lente (>2s) quand il y a 1000 dossiers.
Analyse la requête Prisma générée (active le query logging : 
log: ['query'] dans PrismaClient).
Propose les index manquants et optimise la requête.
```

---

## RÉCAPITULATIF — LES 5 RÈGLES D'OR AVEC CLAUDE CODE

**1. Toujours lire la spec avant de coder**
```
Lis specs/[FICHIER] section [X] → résume → valide → code
```

**2. Plan avant code pour les composants complexes**
```
Donne-moi le plan → je valide → tu codes
```

**3. Petits incréments (4-5 fichiers max par interaction)**
```
Crée ces 4 fichiers → teste → itère → 4 fichiers suivants
```

**4. Tests systématiques après chaque composant**
```
Crée le composant → crée les tests → lance les tests → corrige → green
```

**5. Commits après chaque session réussie**
```
Lance : git add -p && git commit -m "feat(ms-dossier): add DossierService with tests"
```

---

*Guide Claude Code Workflow — Parapheur Digital — Avril 2026*  
*Solo Developer + Claude Code → Application professionnelle en production*
