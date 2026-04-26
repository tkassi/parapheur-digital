# Parapheur Digital — Programme de spécifications

**Banque marocaine cotée | Solo Developer + Claude | Avril 2026**

Programme complet de digitalisation du circuit de signature et d'engagement bancaire.  
12 000 dossiers/an — 315 JH — 54 semaines — 170 K MAD CAPEX.

---

## Structure du repository

```
parapheur-digital/
├── Lot 1/                                          ← Livrables V1 (baseline + révision)
│   ├── REVISION-CONTEXTE-MAROC-SOLO.md             ← Audit de révision (4 paramètres)
│   ├── README.md                                   ← Notes de cadrage initial
│   ├── PHASE1-specifications-detaillees.md         ← V1 — UX/US (supersédé)
│   ├── PHASE2-specifications-metier.md             ← V1 — Métier (supersédé)
│   ├── PHASE3-conformite-securite.md               ← V1 — Conformité (supersédé)
│   ├── PHASE4-architecture-integration.md          ← V1 — Architecture (supersédé)
│   ├── COPIL-Parapheur-Digital-Synthese-MAROC-SOLO.pptx  ← Deck COPIL v2 (18 slides)
│   └── COPIL-Parapheur-Digital-Synthese.pptx       ← Deck COPIL v1 (initial)
│
├── PHASE1-V2-specifications-detaillees-MAROC-SOLO.md  ← V2 — 47 US, 6 personas, WCAG RTL
├── PHASE2-V2-specifications-metier-MAROC-SOLO.md      ← V2 — 87 règles DMN, N1-N6, ROI
├── PHASE3-V2-conformite-securite-MAROC-SOLO.md        ← V2 — Loi 43-20/09-08, ISO 27001
├── PHASE4-V2-architecture-integration-MAROC-SOLO.md   ← V2 — C4, 5 microservices, OpenAPI
│
├── GUIDE-DEVELOPPEMENT-PARAPHEUR-DIGITAL.md           ← Guide dev complet VS Code + Claude
│
├── parapheur-diagrammes-bpmn.md                       ← Diagrammes BPMN processus
├── parapheur-digital-processus.md                     ← Processus métier détaillés
├── parapheur-livrables-complementaires.md             ← Livrables complémentaires
│
└── [PDFs]  Versions impression paysage des livrables
```

---

## Livrables V2 (contexte Maroc / Solo)

| Livrable | Description | Pages |
|---------|-------------|-------|
| `PHASE1-V2` | Spécifications UX — 47 User Stories, 6 personas, 8 wireframes, WCAG 2.1 AA, i18n FR/AR + RTL | ~80 p. |
| `PHASE2-V2` | Spécifications métier — 87 règles DMN, hiérarchie N1-N6, business case 19,6 M MAD/an | ~80 p. |
| `PHASE3-V2` | Conformité & sécurité — Loi 09-08/43-20/05-20, STRIDE, 47 contrôles ISO 27001, CNDP | ~85 p. |
| `PHASE4-V2` | Architecture & intégration — C4, 5 microservices, OpenAPI e-sig/GED, sizing, CI/CD | ~85 p. |
| `GUIDE-DEV` | Guide de développement — VS Code + Claude, monorepo, Prisma, XState, React RTL | ~120 p. |
| `COPIL deck` | Présentation COPIL Direction 18 slides — synthèse exécutive, budget, décisions | 18 slides |

**Total : ~450 pages + 1 deck COPIL**

---

## Cadre de révision (4 paramètres structurants)

| Paramètre | Baseline initiale | Révision Maroc/Solo |
|-----------|------------------|---------------------|
| Réglementation | EU : RGPD + eIDAS + DORA | Maroc : Loi 09-08 + 43-20 + 05-20 + BAM |
| E-signature | Universign PSCo + HSM | Plateforme interne (adapter REST) |
| Archivage | AWS S3 + Glacier | GED interne (adapter REST, BAM 10 ans) |
| Exécution | Équipe 18 mois | Solo developer + Claude, 54 semaines |

---

## Budget (révision Maroc/Solo)

| Poste | Montant |
|-------|---------|
| CAPEX (cash) | 170 000 MAD |
| OPEX/an (cash) | 162 000 MAD |
| TCO 5 ans | 1 040 000 MAD |
| Effort | 315 JH (solo + Claude) |
| Break-even | < 1 mois |
| Bénéfices/an | 19 600 000 MAD |

**vs plan initial : -98% coût, -79% effort, break-even 40× plus rapide**

---

## Stack technique

```
Backend   : Node.js 20 + Fastify 4 + TypeScript 5 + Prisma 5 + PostgreSQL 16
Workflow  : XState 5 (state machines — Loi 43-20 compliance)
Queue     : BullMQ + Redis 7
Frontend  : React 18 + Vite + shadcn/ui + Tailwind CSS + i18next (FR/AR RTL)
Tests     : Vitest + Playwright (≥80% coverage)
CI/CD     : GitHub Actions → rolling deploy
```

---

## 12 décisions COPIL requises

| Code | Décision clé |
|------|-------------|
| D-R1 | Agrément DGSSI pour SEQ ou rester SEA (Loi 43-20) |
| D-R2 | Hébergement cloud banque interne vs DC Maroc Tier III |
| D-R3 | Audit API e-signature interne (Gate-1 avant démarrage) |
| D-R4 | Audit API GED interne (Gate-2 avant démarrage) |
| D-R5 | Désignation correspondant CNDP + pré-consultation |
| D-R6 | Budget 170K MAD CAPEX protégé + 162K MAD/an OPEX |
| D-R7 | Lettre de mission DG/DSI + mandat solo+Claude |
| D-R8 | Validation stack Node.js/React/PostgreSQL |
| D-R9 | Périmètre MVP (16 US) vs Full (47 US) |
| D-R10 | Processus recette UAT + représentants métier |
| D-R11 | Stratégie déploiement rolling vs blue/green |
| D-R12 | RPO 1h WAL PostgreSQL confirmé par DSI |

---

*Programme Parapheur Digital — Banque Marocaine Cotée — Solo Developer + Claude*  
*Avril 2026*
