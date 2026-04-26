# Programme Parapheur Digital

> **Confidentiel — Interne / Direction.** Documentation de cadrage et spécifications du programme de digitalisation du parapheur (workflow + signature électronique) pour banque marocaine.

## Contexte

Programme de remplacement du parapheur papier par une solution digitale full-web couvrant :

- Workflow d'approbation et de validation
- Signature électronique (Loi 43-20, niveaux SES / SEA / SEQ)
- Archivage légal (Code de commerce 15-95, conservation 10 ans)
- Conformité Loi 09-08 (CNDP), circulaires Bank Al-Maghrib, DGSSI/DNSSI

Volumétrie cible : ~12 000 dossiers/an, ~285 M€ d'engagements/an.

## Livrables

### Cadrage initial (avril 2026)

| Document | Pages | Description |
|---|---|---|
| `parapheur-digital-processus.md` | ~50 | Cartographie des processus AS-IS / TO-BE |
| `parapheur-diagrammes-bpmn.md` | ~35 | Diagrammes BPMN 2.0 |
| `parapheur-livrables-complementaires.md` | ~50 | Annexes (matrices, glossaires, KPI) |

### Spécifications McKinsey-grade (4 phases)

| Document | Pages | Périmètre |
|---|---|---|
| `PHASE1-specifications-detaillees.md` | ~105 | User stories (47), wireframes, parcours utilisateurs |
| `PHASE2-specifications-metier.md` | ~110 | Règles métier (87 DMN), business case, ROI |
| `PHASE3-conformite-securite.md` | ~85 | RGPD/eIDAS/ISO 27001, STRIDE, plan sécurité |
| `PHASE4-architecture-integration.md` | ~95 | Architecture C4, microservices, intégrations SAP/Okta/Universign |

### Synthèse COPIL Direction

| Document | Description |
|---|---|
| `COPIL-Parapheur-Digital-Synthese.pptx` | Deck 18 slides — synthèse exécutive pour COPIL Direction (DG/DAF/DSI/RSSI/DPO/DJ) |

### Révision contextuelle (avril 2026 — branche `revision/contexte-maroc-solo`)

| Document | Description |
|---|---|
| `REVISION-CONTEXTE-MAROC-SOLO.md` | Document amendant les 4 phases pour intégrer 4 paramètres : (1) déploiement banque marocaine, (2) plateforme e-signature interne, (3) plateforme GED interne, (4) mode solo + Claude. **Recalibre l'architecture, la conformité, le budget (MAD/JH) et le planning.** |
| `COPIL-Parapheur-Digital-Synthese-MAROC-SOLO.pptx` | Deck COPIL révisé reflétant le nouveau contexte |

## Cadre réglementaire applicable (Maroc / banque)

- **Loi 09-08** — protection des données à caractère personnel (CNDP)
- **Loi 43-20** — services de confiance pour transactions électroniques (SES/SEA/SEQ)
- **Loi 05-20** — cybersécurité
- **Bank Al-Maghrib** — Circulaires 03/W/2016 (gouvernance SI), 5/W/2017 (PCA), 1/G/2010 (LAB-FT)
- **DGSSI / DNSSI** — Directive Nationale de la Sécurité des Systèmes d'Information
- **Code de commerce 15-95** + **DOC art. 417-1, 417-2** — preuve électronique
- **Loi 43-05** — lutte contre le blanchiment

## Structure du dépôt

```
.
├── README.md
├── parapheur-digital-processus.md
├── parapheur-diagrammes-bpmn.md
├── parapheur-livrables-complementaires.md
├── PHASE1-specifications-detaillees.md
├── PHASE2-specifications-metier.md
├── PHASE3-conformite-securite.md
├── PHASE4-architecture-integration.md
├── COPIL-Parapheur-Digital-Synthese.pptx
├── REVISION-CONTEXTE-MAROC-SOLO.md         # branche révision
└── COPIL-Parapheur-Digital-Synthese-MAROC-SOLO.pptx  # branche révision
```

## Workflow git

- `main` — baseline cadrage UE/équipe (avril 2026)
- `revision/contexte-maroc-solo` — révision banque marocaine + e-sign/DMS interne + mode solo

## Confidentialité

**Documents internes — diffusion restreinte Direction et équipe projet.** Ne pas pousser sur dépôt public. Repo privé GitHub recommandé ou GitLab interne banque.
