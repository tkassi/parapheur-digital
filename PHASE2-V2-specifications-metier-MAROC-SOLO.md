# Phase 2 v2 — Spécifications métier

**Programme Parapheur Digital — Banque commerciale universelle marocaine cotée à la Bourse de Casablanca, classée OIV**

| Métadonnée | Valeur |
|---|---|
| Version | R2.0 — Avril 2026 |
| Auteur | Direction de programme |
| Statut | Pour validation Direction Crédit, Direction Risques, Direction Conformité, DAF, COPIL |
| Documents amendés | Phase 2 v1 (règles DMN EUR, hiérarchie générique) |
| Confidentialité | Interne / Direction — diffusion restreinte |
| Pages | ~80 |

---

## Note de cadrage

Cette Phase 2 v2 remplace la Phase 2 v1 pour le contexte d'une **banque marocaine cotée OIV opérant le parapheur pour ses validations et engagements internes uniquement**.

**Ce qui change vs v1** :

| Domaine | v1 (UE) | v2 (Maroc / banque) |
|---|---|---|
| Devise | EUR pivot | **MAD pivot**, multi-devise (EUR, USD) via taux BAM |
| Seuils délégation | 50 K€, 200 K€, 1 M€, 10 M€ | **550 K MAD, 2,2 M MAD, 11 M MAD, 110 M MAD** (taux 10,8) — à valider Direction Crédit |
| TVA | 20 % UE | **20 %, 14 %, 10 %, 7 %** (taux Maroc) |
| Hiérarchie | Générique (DG, Direction, Manager) | **Banque marocaine** : DG, Comité de Direction, Comités spécialisés (crédit, risques, ALM), Direction Réseau, agences, comité de crédit, hiérarchie corps inspection |
| Documents requis | UE génériques | **Maroc** : CIN/CNIE, CNSS, RC, ICE, IF, attestation fiscale |
| Niveau signature | eIDAS QES > 50 K€ | **SEA Loi 43-20** pour 100 % cas internes ; **SEQ optionnelle** > 5 M MAD si plateforme banque agréée DGSSI |
| Périmètre signature | Interne + externe (clients) | **Interne uniquement** — engagements et validations banque |
| ROI | Capital 285 M€/an | **Volume 12 K dossiers/an**, capital engagé ~3 080 M MAD/an |
| Calendrier | Calendrier UE | **Calendrier Maroc** : jours fériés Maroc (Aïd, Achoura, fête du Trône, etc.), week-end dimanche-samedi (banques marocaines : sam-dim) |

---

## Sommaire

1. Cadrage économique
2. Hiérarchie banque marocaine et organes de décision
3. Délégations et seuils
4. Catalogue des 87 règles métier (DMN-like)
5. KPIs métier
6. Business case et ROI
7. Plan d'adoption
8. Volumétrie détaillée
9. Annexes

---

## 1. Cadrage économique

### 1.1 Volumétrie et flux

| Indicateur | Valeur 2026 | Source |
|---|---|---|
| **Dossiers /an** | 12 000 | Cadrage initial confirmé |
| **Dossiers /jour ouvré** | ~50 | 12K / 240j ouvrés Maroc |
| **Capital engagé /an** | ~3 080 M MAD | Estimation banque (= 285 M€ × 10,8) |
| **Engagement moyen /dossier** | ~257 K MAD | 3 080 M / 12 K |
| **Médiane engagements** | ~80 K MAD | (distribution log-normale) |
| **Top décile** | > 1 M MAD | Comité de crédit |
| **Top centile** | > 10 M MAD | Comité de crédit + DG |

### 1.2 Répartition par type de décision

| Type | Volume /an | % | Engagement moyen |
|---|---|---|---|
| Validation RH (recrutement, mutation, départ) | 2 000 | 17 % | n/a |
| Validation dépense opérationnelle | 3 500 | 29 % | 50 K MAD |
| Engagement crédit (commissions, dérogations risque) | 2 800 | 23 % | 800 K MAD |
| Validation comité de crédit (décisions délibérées) | 800 | 7 % | 12 M MAD |
| Notes internes & politiques | 600 | 5 % | n/a |
| Délégations & procurations internes | 1 200 | 10 % | n/a |
| Validations conformité (LAB-FT, sanctions) | 700 | 6 % | n/a |
| Décisions ALM (gestion actif-passif) | 200 | 2 % | 50 M MAD |
| Autres engagements opérationnels | 200 | 1 % | varié |
| **Total** | **12 000** | **100 %** | – |

### 1.3 Saisonnalité

| Période | Volume relatif | Justification |
|---|---|---|
| Janvier (post-clôture annuelle) | 130 % | Reprise activité, dépenses Q1 |
| Février-Avril | 100 % | Régime de croisière |
| Mai-Juin | 95 % | Préparation arrêtés semestriels |
| Juillet-Août | 70 % | Vacances |
| Septembre | 110 % | Reprise rentrée |
| Octobre-Novembre | 115 % | Pré-clôture, arrêtés |
| Décembre | 90 % | Clôture, blocages temporaires |

**Pic horaire** : 10h-12h (60 % activité) puis 14h-16h (30 %).

### 1.4 Devises et conversion

| Devise | Usage | Source taux |
|---|---|---|
| **MAD** | Pivot / par défaut 95 % cas | BAM cotation officielle |
| **EUR** | Engagements internationaux ~3 % | BAM taux quotidien |
| **USD** | Engagements internationaux ~2 % | BAM taux quotidien |
| GBP, CHF, JPY | Marginal | Sur demande |

**Règle métier** : tous les seuils de délégation sont **exprimés en MAD** dans le système. Les engagements en devises étrangères sont **convertis automatiquement au taux BAM du jour** pour évaluation des règles.

```
Si engagement = 100 000 EUR et taux BAM = 10,80 :
  engagementEvalue = 1 080 000 MAD
  → règle DMN-03 : > 550 K MAD → niveau N3
```

---

## 2. Hiérarchie banque marocaine et organes de décision

### 2.1 Structure cible (banque universelle cotée typique)

```
                   ┌───────────────────────┐
                   │  CONSEIL              │
                   │  D'ADMINISTRATION     │ ← Loi 17-95 + Code gouvernance
                   └───────────┬───────────┘
                               │
             ┌─────────────────┼─────────────────┐
             ▼                 ▼                 ▼
   ┌──────────────────┐ ┌─────────────┐ ┌───────────────┐
   │ Comité d'audit   │ │ Comité      │ │ Comité de     │
   │ (3-5 admin       │ │ risques     │ │ rémunérations │
   │  indépendants)   │ │             │ │               │
   └──────────────────┘ └─────────────┘ └───────────────┘
                               │
                               ▼
                   ┌───────────────────────┐
                   │  DIRECTEUR GÉNÉRAL    │
                   │  (DG) — N1            │
                   └───────────┬───────────┘
                               │
        ┌──────────────┬───────┼───────┬─────────────┐
        ▼              ▼       ▼       ▼             ▼
┌──────────────┐ ┌──────────┐ ┌────┐ ┌────────┐ ┌──────────────┐
│ DGA Banque   │ │ DGA      │ │ DGA│ │ DGA    │ │ DGA Inspect. │
│ commerciale  │ │ Risques  │ │ SI │ │ Finance│ │ + Audit int. │
│ + Réseau     │ │+Conform. │ │+Op.│ │ + RH   │ │ (3L défense) │
│              │ │          │ │    │ │        │ │              │
│ — N2         │ │ — N2     │ │ N2 │ │ — N2   │ │ — N2         │
└──────┬───────┘ └────┬─────┘ └─┬──┘ └───┬────┘ └──────────────┘
       │              │          │       │
       ▼              ▼          ▼       ▼
   Directions   Directions   Directions Directions
   - Crédit     - Risques    - DSI      - DAF
     PRO/PME    - Conformité - Sécurité - Trésorerie
   - Crédit     - Juridique  - Org      - RH
     Particul.  - LAB-FT     - Production- Achats
   - Réseau                            - Comm. financière
   - Trade Fin.                        - Contrôle de gestion
   - Marchés
       │
       ▼
   N3 - Directeurs régionaux / Direction de zone
       │
       ▼
   N4 - Directeurs d'agence
       │
       ▼
   N5 - Chargés de clientèle / opérations
```

### 2.2 Comités (organes collégiaux)

| Comité | Composition | Fréquence | Périmètre dossiers parapheur |
|---|---|---|---|
| **Comité de Direction (CODIR)** | DG + DGA | Hebdomadaire | Décisions stratégiques, validations engagements > 50 M MAD |
| **Comité de Crédit** | DG + DGA Risques + DGA Banque commerciale + Direction Crédit + Direction Risques | 2 fois/sem | Engagements crédit > 5 M MAD ou hors politique |
| **Comité ALM** (Asset Liability Management) | DGA Risques + DAF + Trésorerie + Risques de marché | Mensuel | Décisions structure bilan, dotations, refinancement |
| **Comité Conformité / LAB-FT** | DGA Conformité + Direction Conformité + DPO + Juridique | Bimensuel | Validations LAB-FT, traitement déclarations soupçon |
| **Comité Investissements / Achats** | DG + DAF + Direction Achats | Mensuel | Engagements opérationnels > 1 M MAD |
| **Comité Risques** (Conseil) | Administrateurs indépendants + invités | Trimestriel | Reporting, stratégie risques (hors parapheur opérationnel) |

### 2.3 Niveaux d'autorité interne

| Niveau | Rôle type | Fonction parapheur |
|---|---|---|
| **N1** | Directeur Général (DG) | Validation finale > 50 M MAD ou cas exceptionnels |
| **N2** | Directeur Général Adjoint (DGA) | Validation 11-50 M MAD selon délégation conseil |
| **N3** | Directeur central / régional | Validation 2,2-11 M MAD selon délégation |
| **N4** | Directeur de département / zone | Validation 550 K - 2,2 M MAD |
| **N5** | Responsable d'agence / chef de service | Validation 100-550 K MAD |
| **N6** | Chargé de clientèle / opération | Validation < 100 K MAD ou émetteur |

### 2.4 Conseil de Conformité Charia

**Non applicable** — la banque est commerciale universelle (non participative). Si filiale participative existe, périmètre hors parapheur.

### 2.5 Inspection / Audit interne (3e ligne de défense)

L'**Inspection Générale** ou **Audit Interne** dispose d'un accès lecture seule global au parapheur pour :
- Audit a posteriori des décisions
- Investigations
- Reporting au Comité d'audit du Conseil

---

## 3. Délégations et seuils

### 3.1 Grille de délégation (proposition à valider Direction Crédit + Comité de Direction)

| Niveau | Plafond MAD | Approbateur typique | Cas d'usage |
|---|---|---|---|
| **N6** | < 100 K | Émetteur (chargé de clientèle, chef de service) | Dépenses courantes, validations RH non-cadres |
| **N5** | 100 - 550 K | Responsable d'agence / chef de service | Engagements clientèle PRO, dépenses opérationnelles |
| **N4** | 550 K - 2,2 M | Directeur de département / zone | Engagements importants, recrutements cadres, dérogations limitées |
| **N3** | 2,2 - 11 M | Directeur central / régional | Engagements crédit significatifs, dérogations risque |
| **N2** | 11 - 50 M | DGA (Directeur Général Adjoint) | Engagements majeurs |
| **N1** | > 50 M | Directeur Général | Engagements stratégiques |
| **CODIR** | > 50 M | Comité de Direction collégial | Décisions stratégiques |
| **Comité Crédit** | > 5 M (crédit) | Délibération collégiale | Tous engagements crédit > 5 M MAD ou dérogations |

### 3.2 Règles spécifiques

| Règle | Application |
|---|---|
| **Co-signature** | Tout engagement > 5 M MAD requiert **2 signatures** (règle des 4 yeux) |
| **Dérogation politique crédit** | Validation systématique Comité Crédit, indépendamment du montant |
| **Concentration risque** | Si nouveau total client > 5 % FP banque, validation N1 + Comité Risques |
| **Partie liée** | Si contrepartie = personne liée (administrateur, dirigeant), validation Comité d'Audit + déclaration AMMC art. 71 Loi 17-95 |
| **Engagement étranger** | Si contrepartie hors Maroc, validation Direction Trade Finance + Conformité |
| **Engagement urgent** | Délai < 24h : double validation hiérarchique express + ratification a posteriori |

### 3.3 Délégations temporaires (vacances)

- Procédure : déclaration via parapheur (Direction RH ou délégant)
- Contraintes :
  - Délégué de niveau **égal ou supérieur**
  - Période **bornée** (date début + date fin obligatoires)
  - **Notification** au sponsor + RH
  - **Journalisation** dans MS-4
- Auto-révocation à date fin

---

## 4. Catalogue des 87 règles métier

Les règles sont stockées en **format DMN-like JSON** dans la base de données, paramétrables sans déploiement code, versionnées.

### 4.1 Structure d'une règle

```json
{
  "id": "DMN-R-001",
  "version": 3,
  "name": "Routage par montant pour engagement crédit standard",
  "category": "routing",
  "active": true,
  "validFrom": "2026-01-01",
  "validTo": null,
  "input": [
    { "name": "type_decision", "type": "string" },
    { "name": "montant_mad", "type": "number" },
    { "name": "is_derogation_politique", "type": "boolean" }
  ],
  "output": [
    { "name": "valideurs_chain", "type": "array<role>" },
    { "name": "signature_level", "type": "enum<SES,SEA,SEQ>" }
  ],
  "rules": [
    { "when": { "type_decision": "engagement-credit", "montant_mad": "<= 100000", "is_derogation_politique": false }, "then": { "valideurs_chain": ["N5"], "signature_level": "SES" } },
    { "when": { "type_decision": "engagement-credit", "montant_mad": "> 100000 AND <= 550000", "is_derogation_politique": false }, "then": { "valideurs_chain": ["N5","N4"], "signature_level": "SEA" } },
    { "when": { "type_decision": "engagement-credit", "montant_mad": "> 550000 AND <= 2200000", "is_derogation_politique": false }, "then": { "valideurs_chain": ["N5","N4","N3"], "signature_level": "SEA" } },
    { "when": { "type_decision": "engagement-credit", "montant_mad": "> 2200000 AND <= 5000000", "is_derogation_politique": false }, "then": { "valideurs_chain": ["N4","N3","N2"], "signature_level": "SEA" } },
    { "when": { "type_decision": "engagement-credit", "montant_mad": "> 5000000", "is_derogation_politique": false }, "then": { "valideurs_chain": ["N3","N2","COMITE_CREDIT"], "signature_level": "SEQ" } },
    { "when": { "is_derogation_politique": true }, "then": { "valideurs_chain": ["COMITE_CREDIT"], "signature_level": "SEQ" } }
  ],
  "hitPolicy": "FIRST"
}
```

### 4.2 Catalogue par catégorie

#### Catégorie A — Routage workflow (15 règles)

| ID | Description | Inputs clés |
|---|---|---|
| DMN-R-001 | Routage par montant — engagement crédit standard | type, montant_mad, derogation |
| DMN-R-002 | Routage par montant — dépense opérationnelle | type, montant_mad, fournisseur_id |
| DMN-R-003 | Routage par montant — validation RH | type, niveau_poste, montant_remuneration |
| DMN-R-004 | Routage validation RH cadres dirigeants | type, niveau_poste >= "cadre-superieur" |
| DMN-R-005 | Routage engagement étranger (devises) | type, devise != MAD |
| DMN-R-006 | Routage partie liée (administrateur, dirigeant) | type, contrepartie_partie_liee = true |
| DMN-R-007 | Routage urgence | urgence_flag, delai_traitement_souhaite |
| DMN-R-008 | Routage dérogation politique crédit | derogation_flag, type |
| DMN-R-009 | Routage selon agence émettrice | agence, type |
| DMN-R-010 | Routage selon direction émettrice | direction, type |
| DMN-R-011 | Routage selon segment client (PRO/PME/Particuliers/Corporate) | segment_client |
| DMN-R-012 | Routage selon score risque | score_risque |
| DMN-R-013 | Routage validation conformité LAB-FT obligatoire | montant_mad > 100000 OR contrepartie_pep = true |
| DMN-R-014 | Routage co-signature engagements > 5 M MAD | montant_mad > 5000000 |
| DMN-R-015 | Routage selon délégation expresse active | delegation_active = true |

#### Catégorie B — Validation des montants (12 règles)

| ID | Description |
|---|---|
| DMN-V-016 | Conversion devise étrangère vers MAD au taux BAM jour |
| DMN-V-017 | Plafond engagement par contrepartie / FP banque |
| DMN-V-018 | Plafond concentration secteur (immo, auto, etc.) |
| DMN-V-019 | Vérification cohérence montant / type de décision |
| DMN-V-020 | Vérification cohérence montant / segment client |
| DMN-V-021 | Vérification présence montant si obligatoire |
| DMN-V-022 | Plafond global agence /mois |
| DMN-V-023 | Plafond global direction /trimestre |
| DMN-V-024 | Vérification budget contrepartie restant |
| DMN-V-025 | Détection montant aberrant (> 10× moyenne historique) |
| DMN-V-026 | Validation montant cohérent avec dossier précédent (suite/complément) |
| DMN-V-027 | Vérification disponibilité fonds (si applicable) |

#### Catégorie C — TVA Maroc (8 règles)

| ID | Description | Taux |
|---|---|---|
| DMN-T-028 | TVA standard 20 % | 20 % |
| DMN-T-029 | TVA réduite 14 % (transport voyageurs, fournitures travaux) | 14 % |
| DMN-T-030 | TVA réduite 10 % (hôtellerie, restauration, banques sur certains services) | 10 % |
| DMN-T-031 | TVA super réduite 7 % (eau, électricité, médicaments OTC) | 7 % |
| DMN-T-032 | TVA exonérée (services bancaires de base, intérêts) | 0 % |
| DMN-T-033 | TVA hors champ (salaires, indemnités) | n/a |
| DMN-T-034 | Calcul TTC = HT × (1 + TVA) selon type opération | dynamique |
| DMN-T-035 | Vérification cohérence TVA / type opération | – |

#### Catégorie D — Niveau de signature requis (15 règles)

| ID | Description |
|---|---|
| DMN-S-036 | SES pour validation < 100 K MAD non-engagement |
| DMN-S-037 | SES pour notes de service, communications internes |
| DMN-S-038 | SES pour délégations de pouvoir entre cadres équivalents |
| DMN-S-039 | SEA pour engagement 100 K - 5 M MAD |
| DMN-S-040 | SEA pour validation RH cadres (recrutement, mutation) |
| DMN-S-041 | SEA pour dépense opérationnelle 100 K - 5 M MAD |
| DMN-S-042 | SEA + double signature pour > 5 M MAD |
| DMN-S-043 | SEQ optionnelle pour engagement > 5 M MAD si plateforme banque agréée DGSSI |
| DMN-S-044 | SEQ pour dérogation politique crédit majeure |
| DMN-S-045 | SEQ pour acte engageant patrimonialement la banque > 50 M MAD |
| DMN-S-046 | Niveau requis pour décisions Comité de Direction collégial : SEA + signature multi-membres |
| DMN-S-047 | Niveau requis pour décisions Comité de Crédit : SEA + signatures multi-membres |
| DMN-S-048 | Détermination si signature physique ou numérique selon disponibilité plateforme e-sign |
| DMN-S-049 | Niveau pour mode dégradé (plateforme e-sign indisponible) : signature physique avec scan |
| DMN-S-050 | Niveau pour engagement urgent < 24h : SEA accélérée |

#### Catégorie E — Délais et SLA (10 règles)

| ID | Description |
|---|---|
| DMN-D-051 | Délai cible validation N5 : 4h ouvrées |
| DMN-D-052 | Délai cible validation N4 : 8h ouvrées |
| DMN-D-053 | Délai cible validation N3 : 1 jour ouvré |
| DMN-D-054 | Délai cible validation N2 : 2 jours ouvrés |
| DMN-D-055 | Délai cible validation N1 / DG : 3 jours ouvrés |
| DMN-D-056 | Délai cible Comité Crédit : selon planning comité (2/sem) |
| DMN-D-057 | Délai cible signature post-validation : 4h ouvrées |
| DMN-D-058 | Délai cible archivage post-signature : 1h |
| DMN-D-059 | Délai expiration dossier non traité : 30 jours calendaires |
| DMN-D-060 | Délai expiration demande signature : 7 jours calendaires |

#### Catégorie F — Escalade (12 règles)

| ID | Description |
|---|---|
| DMN-E-061 | Escalade N+1 si délai dépassé de 50 % |
| DMN-E-062 | Escalade N+2 si délai dépassé de 100 % |
| DMN-E-063 | Escalade DG si délai dépassé de 200 % et engagement > 5 M MAD |
| DMN-E-064 | Escalade en cas de désaccord entre valideurs successifs (ping-pong > 2 cycles) |
| DMN-E-065 | Escalade automatique si valideur absent (vacances, maladie, départ) > 24h sans réponse |
| DMN-E-066 | Escalade Comité Crédit si engagement > 5 M MAD et désaccord N3/N2 |
| DMN-E-067 | Escalade Conformité si suspicion LAB-FT (montant + pattern) |
| DMN-E-068 | Escalade Risques si dépassement plafond contrepartie |
| DMN-E-069 | Escalade Inspection si tentative de bypass détectée |
| DMN-E-070 | Notification automatique sponsor en cas d'incident technique |
| DMN-E-071 | Escalade RSSI si tentatives auth multiples échouées |
| DMN-E-072 | Escalade Direction Juridique pour cas litigieux complexes |

#### Catégorie G — Documents requis (15 règles)

| ID | Description | Documents Maroc |
|---|---|---|
| DMN-DOC-073 | Documents requis dossier client PRO | RC, ICE, IF, attestation fiscale, CIN dirigeant |
| DMN-DOC-074 | Documents requis dossier client PME | RC, ICE, IF, statuts, bilans 3 ans, CIN dirigeants, CNSS |
| DMN-DOC-075 | Documents requis dossier client Corporate | RC, ICE, IF, statuts, comptes audités 3 ans, CIN, organigramme |
| DMN-DOC-076 | Documents requis dossier client Particulier | CIN/CNIE, justificatif domicile, bulletin paie, attestation employeur |
| DMN-DOC-077 | Documents requis engagement crédit | + business plan, garanties, états financiers |
| DMN-DOC-078 | Documents requis recrutement | CIN, diplômes, CV, références, casier judiciaire |
| DMN-DOC-079 | Documents requis fournisseur | RC, ICE, IF, attestation fiscale, RIB, attestation CNSS |
| DMN-DOC-080 | Documents requis partie liée | + déclaration partie liée, autorisation Comité Audit |
| DMN-DOC-081 | Documents requis dossier LAB-FT | KYC complet, justificatif origine fonds, déclaration soupçon si applicable |
| DMN-DOC-082 | Documents requis dépense > 100 K MAD | Devis, bon de commande, facture pro forma, attestation fiscale fournisseur |
| DMN-DOC-083 | Vérification expiration documents (RC < 3 mois, IF < 1 an, etc.) |  |
| DMN-DOC-084 | Vérification authenticité (hash, signature, tampon notarié) |  |
| DMN-DOC-085 | Vérification cohérence ICE / RC / IF |  |
| DMN-DOC-086 | Documents archivage 10 ans (Code commerce 15-95 art. 22) |  |
| DMN-DOC-087 | Anonymisation post-rétention légale | RGPD-équivalent + Loi 09-08 |

### 4.3 Hit policy et combinaisons

| Hit policy | Usage |
|---|---|
| **FIRST** | Première règle qui matche (par défaut routage) |
| **UNIQUE** | Une seule règle doit matcher (sinon erreur — pour calculs cohérents) |
| **PRIORITY** | Toutes règles matchées triées par priorité |
| **COLLECT** | Toutes règles matchées collectées (pour escalades multiples) |

### 4.4 Versioning et déploiement des règles

```
Modification règle DMN-R-001
    │
    ▼
Création version v4 (draft)
    │
    ▼
Validation Direction Crédit + Direction Risques (workflow approbation interne)
    │
    ▼
Tests staging avec scénarios simulation (10 cas types)
    │
    ▼
Activation v4 (validFrom = date) + v3 inactivée (validTo = date - 1)
    │
    ▼
Workflows en cours : finis avec v3 ; nouveaux workflows : v4
```

**Effort dev moteur DMN-like** : 12 JH (déjà inclus dans MS-2 Workflow).

**Effort paramétrage 87 règles** : 8 JH (création JSON + tests).

---

## 5. KPIs métier

### 5.1 Taxonomie des indicateurs

| Famille | Indicateurs |
|---|---|
| **Volumétrie** | Dossiers créés, soumis, signés, archivés, rejetés |
| **Délais** | Cycle moyen, P50, P75, P90 par type/montant |
| **Performance** | Taux de respect SLA, taux d'escalade, taux de rejet |
| **Adoption** | Utilisateurs actifs, % digital vs papier, satisfaction NPS |
| **Risque** | Dérogations, dépassements plafonds, incidents, alertes LAB-FT |
| **Conformité** | Demandes droits CNDP traitées, audits, signatures qualifiées vs simples |

### 5.2 KPIs cibles (à 12 mois post go-live)

| KPI | Cible | Baseline (papier) |
|---|---|---|
| **Cycle moyen complet** (création → archivage) | ≤ 3 jours ouvrés | ~12 jours |
| **Taux signature digitale** | ≥ 95 % | 0 % |
| **Taux respect SLA validation N5** | ≥ 90 % | n/a |
| **Taux respect SLA validation N4** | ≥ 85 % | n/a |
| **Taux d'escalade** | ≤ 5 % | n/a |
| **Taux de rejet** | ≤ 8 % | ~5 % |
| **Utilisateurs actifs /mois** | ≥ 80 % effectif éligible | n/a |
| **Disponibilité service** | ≥ 99,5 % | n/a |
| **NPS utilisateurs internes** | ≥ +30 | n/a |
| **Délai exercice droit accès CNDP** | ≤ 25 jours | 30 jours légaux |
| **Couverture audit** (% dossiers tracés intégralement) | 100 % | ~70 % (papier) |

### 5.3 Tableau de bord COPIL trimestriel

| Section | Contenu |
|---|---|
| Activité | Volumes par type, par direction, par segment |
| Performance | Cycle moyen, SLA, top 5 directions retardataires |
| Risques | Top dérogations, escalades, incidents |
| Adoption | Taux digital, top utilisateurs, retours qualitatifs |
| Conformité | Demandes CNDP, signatures qualifiées, anomalies |
| Roadmap | Évolutions planifiées trimestre suivant |

---

## 6. Business case et ROI

### 6.1 Bénéfices quantifiés

| Bénéfice | Calcul | MAD/an |
|---|---|---|
| **Gain temps signataires** | 12 K dossiers × 0,8 h gagnée × 250 MAD/h moyen | **2 400 000** |
| **Réduction délais (cash conversion cycle)** | Capital engagé 3 080 M MAD × 0,5 % × cycle réduit | **15 400 000** |
| **Réduction litiges et pertes opérationnelles** | Estimation prudente sur incidents traçabilité | **800 000** |
| **Réduction papier / logistique / archivage physique** | 12 K dossiers × 80 MAD (papier + transport + stockage) | **960 000** |
| **Total bénéfices /an** | – | **19 560 000 MAD** |

**Hypothèse 1 (gain temps)** : un dossier mobilise typiquement 4-5 signataires × 12 minutes (papier : recherche, signature, transmission). En digital, ramené à 3-4 minutes. Gain ~0,8h par dossier, tous signataires confondus.

**Hypothèse 2 (cash conversion)** : un cycle de validation/signature plus court de 9 jours sur un capital engagé moyen permet, sur la durée de vie des engagements, une optimisation de trésorerie estimée à 0,5 % du capital. **Hypothèse principale du business case** — à valider DAF.

**Hypothèse 3 (litiges)** : traçabilité immuable = preuve robuste en cas de contestation, diminution risque juridique.

**Hypothèse 4 (papier)** : papier + impression + transport courrier interne + archivage physique 10 ans estimé à 80 MAD/dossier.

### 6.2 Coûts (rappel synthèse)

| Poste | Build (MAD) | Run /an (MAD) |
|---|---|---|
| CAPEX cash conformité (cf. P3) | 103 000 | – |
| CAPEX cash architecture & infra | 52 100 (setup cloud + outillage + Claude + buffer) | – |
| CAPEX cash divers (formation, frais, buffer) | 15 000 | – |
| **Total CAPEX cash** | **170 100** | – |
| OPEX cash conformité | – | 88 000 |
| OPEX cash architecture & infra | – | 86 300 |
| **Total OPEX cash** | – | **~174 300** |
| Effort solo Build (315 JH × 0 si ressource interne) | 0 | – |
| Effort Run (45 JH × 0) | – | 0 |

**TCO 5 ans cash** : 170 K + 5 × 174 K = **~1 042 K MAD** (cohérent avec arrondi 0,98 M MAD précédemment communiqué — l'écart provient du raffinement du chiffrage OPEX).

### 6.3 ROI

| Indicateur | Valeur |
|---|---|
| **CAPEX an 0** | 170 K MAD |
| **OPEX cumulé an 1** | 174 K MAD |
| **Bénéfices an 1** (palier 50 % adoption en moyenne) | 9 780 K MAD |
| **Bénéfices nets an 1** | +9 436 K MAD |
| **Break-even** | **< 1 mois après go-live** |
| **Bénéfices an 2-5** (palier 100 %) | 19 560 K MAD/an |
| **TCO 5 ans** | 1 042 K MAD |
| **Bénéfices cumulés 5 ans** | 88 000 K MAD |
| **NPV 5 ans (taux actualisation 8 %)** | ~71 M MAD |
| **IRR** | > 1 000 % |

→ **Conclusion** : le ROI est **trivial** sous toutes hypothèses raisonnables, même en divisant les bénéfices par 5. Le débat n'est plus financier — il est sur l'**exécution opérationnelle** et la **conformité**.

### 6.4 Sensibilité

| Hypothèse | Valeur centrale | Bornes (-50% / +50%) | Impact NPV |
|---|---|---|---|
| Gain cash conversion 0,5 % | 15,4 M MAD/an | 7,7 - 23,1 M MAD/an | NPV reste > 30 M MAD même au minimum |
| Adoption an 1 | 50 % | 25 % - 75 % | Délai break-even reste < 6 mois au pire |
| Volumétrie | 12 K dossiers/an | 6 K - 18 K | Bénéfices proportionnels |
| TCO build | 170 K MAD | 85 - 510 K MAD | Insignifiant vs bénéfices |

→ **Le business case est robuste à toutes les sensibilités plausibles.**

---

## 7. Plan d'adoption

### 7.1 Stratégie en 3 paliers

| Palier | Période (post go-live) | Cible | Volume estimé |
|---|---|---|---|
| **P1** | Mois 1 | 1 direction pilote (typ. Banque commerciale Casablanca) | 1 000 dossiers/an équivalent |
| **P2** | Mois 2-3 | Extension 4-5 directions principales | 6 000 dossiers/an équivalent |
| **P3** | Mois 4-6 | Toutes directions, toutes agences | 12 000 dossiers/an |

### 7.2 Gestion du changement

| Action | Période | Audience |
|---|---|---|
| Communication COPIL → directions | T-4 sem | Top management |
| Communication directions → équipes | T-3 sem | Tous utilisateurs |
| Sessions de formation (4 sessions × 50 personnes) | T-2 sem à T+2 sem | Utilisateurs |
| Documentation utilisateur (manuel PDF + vidéos courtes) | T-2 sem | Tous |
| Hotline / support T+0 à T+8 sem | continu | Tous |
| Champions / référents par direction (10 personnes) | T-2 sem à T+12 sem | Personnes ressources |
| Revue d'usage et ajustements | mensuel | COPIL Direction |

### 7.3 Doubles saisies (transition)

**Phase de transition** : 3 mois après go-live d'une direction, **double saisie papier + digital** pour :
- Validation par utilisateurs
- Détection bugs / écarts
- Préservation continuité en cas d'incident majeur

**Critère arrêt double saisie** : taux de cohérence digital/papier > 99 % et stabilité applicative confirmée 30 jours sans incident critique.

---

## 8. Volumétrie détaillée — données pour sizing

### 8.1 Distribution par tranche de montant (estimation)

| Tranche MAD | % dossiers | % capital | Volume an |
|---|---|---|---|
| < 100 K | 55 % | 5 % | 6 600 |
| 100 K - 550 K | 25 % | 12 % | 3 000 |
| 550 K - 2,2 M | 12 % | 18 % | 1 440 |
| 2,2 M - 11 M | 6 % | 25 % | 720 |
| 11 M - 50 M | 1,7 % | 30 % | 200 |
| > 50 M | 0,3 % | 10 % | 40 |
| **Total** | **100 %** | **100 %** | **12 000** |

### 8.2 Distribution par direction (estimation)

| Direction | % dossiers | Volume an |
|---|---|---|
| Banque commerciale (PRO/PME/Particuliers) | 40 % | 4 800 |
| Crédit / Risques | 15 % | 1 800 |
| RH | 12 % | 1 440 |
| Achats / DAF | 10 % | 1 200 |
| Réseau (agences) | 10 % | 1 200 |
| Conformité / Juridique | 5 % | 600 |
| DSI / SI | 4 % | 480 |
| Autres | 4 % | 480 |
| **Total** | **100 %** | **12 000** |

### 8.3 Profils utilisateurs

| Profil | Nombre estimé | Volume signatures /an / personne | Volume total |
|---|---|---|---|
| Émetteurs (chargés clientèle, opérationnels) | ~800 | ~15 | ~12 000 émissions |
| Valideurs N5 | ~200 | ~50 | ~10 000 validations N5 |
| Valideurs N4 | ~80 | ~70 | ~5 600 validations N4 |
| Valideurs N3 | ~30 | ~80 | ~2 400 validations N3 |
| Valideurs N2 | ~10 | ~80 | ~800 validations N2 |
| DG / DGA / Comités | ~10 | ~40 | ~400 décisions exécutives |
| Lecture / audit / contrôle | ~50 | ~100 (lectures) | – |
| **Total comptes actifs** | **~1 200** | – | – |

---

## 9. Annexes

### Annexe A — Décisions COPIL métier

| ID | Décision | Recommandation |
|---|---|---|
| **D-R10** | Devises | MAD pivot, multi-devise (EUR, USD) via taux BAM jour |
| **D-R2** | Niveau signature | SEA pour 100 % cas internes ; SEQ optionnelle > 5 M MAD si plateforme banque agréée DGSSI |
| **Met-1** | Grille de délégation | Validation Direction Crédit + Comité de Direction selon barème §3.1 |
| **Met-2** | Périmètre fonctionnel | Internes uniquement — clients externes hors périmètre |
| **Met-3** | Hiérarchie | Validation organigramme banque par DRH + DGA |
| **Met-4** | Plan d'adoption 3 paliers | Direction pilote = Banque commerciale Casablanca (à confirmer) |

### Annexe B — Mapping types de décision ↔ règles DMN

(Tableau croisé — extrait illustratif)

| Type de décision | Règles applicables |
|---|---|
| Engagement crédit | DMN-R-001, DMN-V-016, DMN-V-017, DMN-V-018, DMN-S-039/042/043, DMN-D-051-057, DMN-DOC-073-077 |
| Validation RH | DMN-R-003, DMN-R-004, DMN-S-040, DMN-DOC-078 |
| Dépense opérationnelle | DMN-R-002, DMN-V-022, DMN-V-024, DMN-T-028-035, DMN-S-041, DMN-DOC-082 |
| Validation comité crédit | DMN-R-014, DMN-S-046/047, DMN-D-056 |
| Décision ALM | DMN-R-005 (si devises), DMN-S-045 |
| Engagement étranger | DMN-R-005, DMN-V-016, DMN-DOC-077 |
| Partie liée | DMN-R-006, DMN-DOC-080 |

### Annexe C — Calendrier banque marocaine

**Jours fériés Maroc 2026 (référence)** :

| Date | Fête |
|---|---|
| 1er janvier | Nouvel An |
| 11 janvier | Manifeste de l'Indépendance |
| 1er mai | Fête du Travail |
| ~ avril (variable lunaire) | Aïd Al-Fitr (2 jours) |
| ~ juin | Aïd Al-Adha (2 jours) |
| ~ juin | Achoura (variable) |
| 30 juillet | Fête du Trône |
| 14 août | Fête Oued Ed-Dahab |
| 20 août | Révolution du Roi et du Peuple |
| 21 août | Fête de la Jeunesse |
| ~ septembre | Aïd Al-Mawlid |
| 6 novembre | Marche Verte |
| 18 novembre | Fête de l'Indépendance |

**Week-end** : le système des banques marocaines varie. La Bourse de Casablanca, la BAM et la majorité des banques universelles : **week-end samedi-dimanche**. (Quelques banques participatives pratiquent vendredi-samedi pour congruence avec finance islamique — non applicable ici.)

→ Configuration parapheur : week-end **samedi-dimanche**, jours fériés 2026 ci-dessus à charger annuellement.

### Annexe D — Glossaire métier

| Terme | Définition |
|---|---|
| **AMMC** | Autorité Marocaine du Marché des Capitaux |
| **BAM** | Bank Al-Maghrib |
| **CNIE** | Carte Nationale d'Identité Électronique |
| **CNSS** | Caisse Nationale de Sécurité Sociale |
| **CODIR** | Comité de Direction |
| **Comité Crédit** | Organe collégial décisions d'engagement |
| **Dérogation** | Décision hors politique standard, validation renforcée requise |
| **FP** | Fonds Propres |
| **ICE** | Identifiant Commun de l'Entreprise |
| **IF** | Identifiant Fiscal |
| **LAB-FT** | Lutte Anti-Blanchiment et Financement du Terrorisme |
| **PEP** | Personne Politiquement Exposée |
| **Partie liée** | Personne ayant des liens (parenté, contrôle) avec dirigeants ou actionnaires |
| **Plafond contrepartie** | Limite engagements totaux d'un client / FP banque (réglementation BAM) |
| **RC** | Registre du Commerce |
| **RIB** | Relevé d'Identité Bancaire |
| **Score risque** | Note attribuée à une contrepartie selon modèle de risque banque |
| **Segment client** | PRO (professionnels), PME, Particuliers, Corporate |
| **TVA** | Taxe sur la Valeur Ajoutée |
| **UTRF** | Unité de Traitement du Renseignement Financier |

### Annexe E — Sources de validation des règles

- Direction Crédit banque (validation seuils + grille délégation)
- Direction Risques (validation plafonds, seuils dérogation, scores)
- Direction Conformité / DPO (validation LAB-FT, parties liées, KYC)
- DRH (validation hiérarchie, niveaux d'autorité)
- DAF (validation TVA, seuils dépenses)
- Direction Juridique (validation niveau signature requis par type de décision)
- Conseil d'Administration (délégations supérieures à 50 M MAD)

---

*Fin de la Phase 2 v2 — Spécifications métier.*

**Prochaine et dernière livraison** : Phase 1 v2 — Spécifications détaillées (user stories, parcours, wireframes, i18n FR/AR).
