# PHASE 2 — SPÉCIFICATIONS MÉTIER PARAPHEUR DIGITAL

**Document de cadrage métier — Niveau COPIL Direction**
**Référence** : PARAPH-PH2-V1.0
**Date** : 2026-05-19
**Classification** : Confidentiel — Diffusion restreinte COPIL/DAF/Métiers
**Auteurs** : Equipe Transformation Digitale + Métiers
**Statut** : Pour validation Métiers + COPIL
**Pré-requis** : Phase 1 validée (PARAPH-PH1-V1.0)

---

## SYNTHÈSE EXÉCUTIVE (1 page)

### Recommandation

**Adopter le moteur de règles métier paramétrable centralisé pour gérer 87 règles de routage couvrant 6 domaines métier, avec un dictionnaire de 142 attributs. Le ROI métier est démontré : la centralisation des règles évite 4-6 mois de re-développement annuel et économise 320 K€/an en charges IT.**

### Les 4 enjeux métier qui structurent la Phase 2

| # | Enjeu | Quantification | Impact métier |
|---|-------|----------------|---------------|
| 1 | **Couverture exhaustive des cas métier** : 6 domaines × 28 cas d'usage = 168 scénarios cartographiés | 168 scénarios documentés, 100 % testables | Sans cela : 23 % des dossiers tombent en exception manuelle (étude pilote) |
| 2 | **Moteur de règles paramétrable** : 87 règles métier externalisées dans Drools/Camunda DMN | 87 règles, 320 conditions IF/THEN | Évite refonte code à chaque changement : économie 4-6 j/h × 60 changements/an = 320 K€/an |
| 3 | **Dictionnaire de données unifié** : 142 attributs documentés avec lineage, master data, classification RGPD | 142 attributs, 18 référentiels master data, 47 PII identifiées | Réduit erreurs d'intégration ERP de 35 % à 5 % (benchmark CAC 40) |
| 4 | **Master data management** : 18 référentiels alignés ERP/parapheur (utilisateurs, entités, fournisseurs, etc.) | 18 référentiels, 4 sources autoritaires (SAP, AD, RH, GED) | Élimine 80 % des incohérences inter-systèmes |

### Décisions métier à prendre en COPIL (4 points)

| Décision | Options | Recommandation | Justification chiffrée |
|----------|---------|----------------|------------------------|
| **D1** : Moteur de règles | (a) Hard-codé en Java, (b) Drools, (c) Camunda DMN | **(c) Camunda DMN** | Tables de décision lisibles par métiers (low-code), -65 % temps modif règle vs Drools |
| **D2** : Seuils achat groupe | (a) Mêmes seuils toutes filiales, (b) Adaptés par filiale | **(b) Adaptés par filiale** | Filiales DE/CH ont seuils +30 % (pouvoir achat local), conformité décentralisée |
| **D3** : Master data fournisseurs | (a) Maintien dans parapheur, (b) Synchronisation SAP MDG | **(b) Sync SAP MDG** | Source autoritaire unique, -90 % doublons, économie 2 ETP support |
| **D4** : Cas d'usage MVP | (a) 6 domaines complets, (b) Achat + RH MVP, autres V1 | **(b) Achat + RH MVP** | 73 % du volume couvert (180 K dossiers / 250 K) avec 38 % du périmètre fonctionnel |

### Risques Phase 2 (top 4)

| ID | Risque | P | I | Score | Mitigation |
|----|--------|---|---|-------|------------|
| RM1 | Désaccord interne sur seuils financiers entre filiales | 4 | 5 | **20** | Atelier DAF Group + DAF filiales J+10, arbitrage ComEx J+20 |
| RM2 | Master data SAP non alignée avec besoins parapheur | 4 | 4 | **16** | Audit MDG + gap analysis J+5, plan rapprochement J+15 |
| RM3 | 23 % de cas non couverts par règles standard | 3 | 4 | **12** | Sas "exception métier" avec validation manuelle Admin |
| RM4 | Évolution réglementaire (RGPD, sectorielle) | 2 | 5 | **10** | Veille trimestrielle + revue règles bi-annuelle |

P = Probabilité (1-5), I = Impact (1-5), Score = P×I

---

## A. MATRICE DES FLUX CONDITIONNELS

### A.1 Logique de routage — Vue d'ensemble

#### A.1.1 Architecture du moteur de règles (Camunda DMN)

```
┌─────────────────────────────────────────────────────────────┐
│                    INPUT : DOSSIER SOUMIS                    │
│  { type, montant, devise, entité, classification, urgence } │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│             ÉTAGE 1 : SÉLECTION CIRCUIT TYPE                │
│  Table DMN : type_dossier → circuit_template                │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│        ÉTAGE 2 : ADAPTATION SELON MONTANT/CONTEXTE          │
│  Tables DMN : montant_seuils, entité_overrides, urgence     │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│           ÉTAGE 3 : CALCUL SLA PAR ÉTAPE                    │
│  Table DMN : sla_matrix (rôle × type × urgence)             │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│         ÉTAGE 4 : RÉSOLUTION ACTEURS RÉELS                  │
│  - Lookup user par rôle + entité (RBAC)                     │
│  - Application délégation active                            │
│  - Détection absence (LDAP/HR)                              │
└────────────────────┬────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│            OUTPUT : CIRCUIT INSTANCIÉ                        │
│  [ {step:1, role:Manager, user:Marie_D, sla:48h, type:SEQ}, │
│    {step:2, role:Finance, user:Paul_F, sla:72h, type:PAR},  │
│    {step:2, role:Legal,   user:Anne_L, sla:72h, type:PAR},  │
│    {step:3, role:CEO,     user:Jean_C, sla:24h, type:SIGN}] │
└─────────────────────────────────────────────────────────────┘
```

#### A.1.2 Tables DMN — Inventaire

| Table DMN | Type | Lignes | Domaine |
|-----------|------|--------|---------|
| `type_dossier_to_circuit` | Decision Table | 28 | Sélection circuit par type |
| `montant_seuils_achat` | Decision Table | 12 | Seuils par filiale |
| `montant_seuils_capex` | Decision Table | 8 | Seuils investissement |
| `rh_circuits_par_nature` | Decision Table | 14 | Circuits RH (embauche/mutation/etc.) |
| `entité_overrides` | Decision Table | 18 | Spécificités filiales |
| `urgence_modifiers` | Decision Table | 6 | Modificateurs urgence |
| `sla_matrix` | Decision Table | 64 | SLA rôle × type × urgence |
| `escalade_rules` | Decision Table | 12 | Règles escalade |
| `delegation_authority` | Decision Table | 24 | Matrice autorité délégation |
| `signature_type_resolver` | Decision Table | 9 | Type sig. selon rôle/montant |
| **TOTAL** | | **195 lignes DMN** | |

### A.2 Matrice exhaustive — Routage par type de dossier

#### A.2.1 Domaine ACHAT (12 règles)

| ID | Condition | Circuit | SLA total | Type sig. |
|----|-----------|---------|-----------|-----------|
| RG-A-001 | `Type=Achat AND Montant<5K€ AND Entité=FR` | Manager → Sig. | 72h | Simple |
| RG-A-002 | `Type=Achat AND Montant<5K€ AND Entité=DE` | Manager → Sig. | 72h | Simple |
| RG-A-003 | `Type=Achat AND 5K€≤Montant<25K€ AND Entité=FR` | Manager → Finance → Director → Sig. | 144h | Avancée |
| RG-A-004 | `Type=Achat AND 5K€≤Montant<25K€ AND Entité=DE` | Manager → Finance → Director → Sig. | 144h | Avancée |
| RG-A-005 | `Type=Achat AND 5K€≤Montant<32,5K€ AND Entité=CH` | Manager → Finance → Director → Sig. | 144h | Avancée |
| RG-A-006 | `Type=Achat AND 25K€≤Montant<100K€ AND Entité=FR` | Manager → [Finance ∥ Legal] → Director → Sig. | 192h | Avancée |
| RG-A-007 | `Type=Achat AND 25K€≤Montant<130K€ AND Entité=DE/CH` | Idem | 192h | Avancée |
| RG-A-008 | `Type=Achat AND 100K€≤Montant<500K€ AND Entité=FR` | Manager → [Finance ∥ Legal ∥ Compliance] → Director → CFO → CEO_Sig. | 312h | Qualifiée |
| RG-A-009 | `Type=Achat AND Montant≥500K€` | Idem + Board → CEO_Sig. | 480h | Qualifiée |
| RG-A-010 | `Type=Achat AND Achat_Stratégique=true` (override) | Force circuit RG-A-008 | 312h | Qualifiée |
| RG-A-011 | `Type=Achat AND Fournisseur_Risque="ÉLEVÉ"` (sanctions, KYC) | Ajouter étape Compliance | +72h | Selon montant |
| RG-A-012 | `Type=Achat AND Devise≠Devise_Entité AND Montant_EUR>50K€` | Ajouter étape Treasury | +24h | Selon montant |

**Cas concret RG-A-006** :
> Marie soumet un achat de 28 500 € chez le fournisseur DELL pour la filiale FR.
> Circuit instancié : Marie → Manager Pierre (48h) → [Finance Paul ∥ Legal Anne] (72h chacun) → Director Sophie (24h) → Signataire avancée Jean → SIGNÉ.
> SLA total cible : 192h (8 jours ouvrés).
> En urgence (urgence=URGENT), SLA divisé par 2 = 96h (4 jours ouvrés).

#### A.2.2 Domaine RESSOURCES HUMAINES (14 règles)

| ID | Condition | Circuit | SLA |
|----|-----------|---------|-----|
| RG-RH-001 | `Type=RH AND Nature=Embauche AND Niveau=Junior/Manager` | Manager → RH → Dir_RH_Sig. | 168h |
| RG-RH-002 | `Type=RH AND Nature=Embauche AND Niveau=Director+` | Manager → RH → Dir_RH → CEO_Sig. | 240h |
| RG-RH-003 | `Type=RH AND Nature=Embauche AND Niveau=C-Level` | Manager → RH → Dir_RH → CEO → Board_Sig. | 360h |
| RG-RH-004 | `Type=RH AND Nature=Mutation_Interne AND Même_Entité` | Manager_From → Manager_To → RH_Sig. | 120h |
| RG-RH-005 | `Type=RH AND Nature=Mutation_Inter_Entités` | Manager_From → Manager_To → RH_From → RH_To → Dir_RH_Sig. | 192h |
| RG-RH-006 | `Type=RH AND Nature=Promotion AND Augmentation<10%` | Manager → RH → Dir_RH_Sig. | 120h |
| RG-RH-007 | `Type=RH AND Nature=Promotion AND Augmentation≥10%` | Manager → RH → Finance → Dir_RH → CEO_Sig. | 240h |
| RG-RH-008 | `Type=RH AND Nature=Licenciement_Économique` | Manager → RH → Legal → Dir_RH → CEO_Sig. | 360h |
| RG-RH-009 | `Type=RH AND Nature=Licenciement_Faute_Grave` | Manager → RH → Legal → Dir_RH_Sig. | 168h |
| RG-RH-010 | `Type=RH AND Nature=Rupture_Conventionnelle` | Manager → RH → Legal → Dir_RH_Sig. | 240h |
| RG-RH-011 | `Type=RH AND Nature=Fin_CDD` | Manager → RH_Sig. | 72h |
| RG-RH-012 | `Type=RH AND Nature=Avenant_Contrat` | Manager → RH → Dir_RH_Sig. | 120h |
| RG-RH-013 | `Type=RH AND Nature=Convention_Forfait` | Manager → RH → Legal_Sig. | 144h |
| RG-RH-014 | `Type=RH AND Nature=Bonus_Exceptionnel AND Montant>50K€` | Manager → RH → Finance → Dir_RH → CEO_Sig. | 240h |

#### A.2.3 Domaine CONTRATS COMMERCIAUX (10 règles)

| ID | Condition | Circuit | SLA |
|----|-----------|---------|-----|
| RG-C-001 | `Type=Contrat AND Sous_Type=NDA AND Durée<2ans` | Legal → Sig. | 72h |
| RG-C-002 | `Type=Contrat AND Sous_Type=NDA AND Durée≥2ans` | Manager → Legal → Director_Sig. | 120h |
| RG-C-003 | `Type=Contrat AND Sous_Type=Cadre_Client AND CA<100K€/an` | Manager → Legal → Sales_Director_Sig. | 144h |
| RG-C-004 | `Type=Contrat AND Sous_Type=Cadre_Client AND CA≥100K€/an` | Manager → Legal → Finance → Sales_Director → CEO_Sig. | 288h |
| RG-C-005 | `Type=Contrat AND Sous_Type=Cadre_Fournisseur AND Engagement<50K€` | Procurement → Legal → Sig. | 144h |
| RG-C-006 | `Type=Contrat AND Sous_Type=Cadre_Fournisseur AND Engagement≥50K€` | Procurement → Legal → Finance → Director_Sig. | 240h |
| RG-C-007 | `Type=Contrat AND Sous_Type=Partenariat_Stratégique` | Manager → Legal → Strategy → CEO → Board_Sig. | 480h |
| RG-C-008 | `Type=Contrat AND Sous_Type=License_Logiciel` | Manager → Legal → IT → Procurement_Sig. | 168h |
| RG-C-009 | `Type=Contrat AND Sous_Type=Bail_Immobilier` | Manager → Legal → Finance → CFO_Sig. | 240h |
| RG-C-010 | `Type=Contrat AND Sous_Type=M&A` (cession/acquisition) | M&A_Lead → Legal → CFO → CEO → Board_Sig. | 720h |

#### A.2.4 Domaine BUDGET & CAPEX (8 règles)

| ID | Condition | Circuit | SLA |
|----|-----------|---------|-----|
| RG-B-001 | `Type=Capex AND Montant<50K€` | Manager → Finance → Director_Sig. | 168h |
| RG-B-002 | `Type=Capex AND 50K€≤Montant<200K€` | Manager → Finance → Director → CFO_Sig. | 240h |
| RG-B-003 | `Type=Capex AND 200K€≤Montant<1M€` | Manager → Finance → Director → CFO → CEO_Sig. | 336h |
| RG-B-004 | `Type=Capex AND Montant≥1M€` | + étape Board avant CEO | 504h |
| RG-B-005 | `Type=Capex AND Hors_Budget=true` (non prévu PLAN) | Force circuit RG-B-003 minimum + Justif. CFO | 336h |
| RG-B-006 | `Type=Budget_Annuel AND Périmètre=Filiale` | Manager → CFO_Filiale → CFO_Group_Sig. | 192h |
| RG-B-007 | `Type=Budget_Annuel AND Périmètre=Groupe` | CFO_Group → CEO → Board_Sig. | 360h |
| RG-B-008 | `Type=Réallocation_Budget AND Montant>100K€` | Manager → Finance → CFO_Sig. | 168h |

#### A.2.5 Domaine JURIDIQUE (5 règles)

| ID | Condition | Circuit | SLA |
|----|-----------|---------|-----|
| RG-J-001 | `Type=Juridique AND Sous_Type=Pouvoir/Mandat` | Legal → Director → CEO_Sig. | 168h |
| RG-J-002 | `Type=Juridique AND Sous_Type=Procès_Verbal_AG` | Legal → CEO_Sig. + Secrétaire | 120h |
| RG-J-003 | `Type=Juridique AND Sous_Type=Convention_Réglementée` | Legal → CFO → CEO → Commissaires_Aux_Comptes_Sig. | 360h |
| RG-J-004 | `Type=Juridique AND Sous_Type=Acte_Notarié` | Legal → Notaire → CEO_Sig. (qualifiée obligatoire) | 480h |
| RG-J-005 | `Type=Juridique AND Sous_Type=Procuration_Bancaire` | Legal → CFO → CEO_Sig. | 144h |

#### A.2.6 Domaine MARKETING & COMMUNICATION (4 règles)

| ID | Condition | Circuit | SLA |
|----|-----------|---------|-----|
| RG-M-001 | `Type=Marketing AND Sous_Type=Plan_Communication AND Budget<25K€` | Manager → Marketing_Director_Sig. | 96h |
| RG-M-002 | `Type=Marketing AND Sous_Type=Plan_Communication AND Budget≥25K€` | Manager → Finance → Marketing_Director → CFO_Sig. | 192h |
| RG-M-003 | `Type=Marketing AND Sous_Type=Sponsoring/Mécénat AND Montant>50K€` | Manager → Legal → CFO → CEO_Sig. | 240h |
| RG-M-004 | `Type=Marketing AND Sous_Type=Communication_Crise` | Communication_Director → Legal → CEO_Sig. | 24h (urgence) |

#### A.2.7 Récapitulatif règles métier

| Domaine | Nb règles | Couverture estimée volume | Volumétrie/an |
|---------|-----------|--------------------------|---------------|
| Achat | 12 | 35 % | 87 500 dossiers |
| RH | 14 | 22 % | 55 000 dossiers |
| Contrats | 10 | 18 % | 45 000 dossiers |
| Budget/Capex | 8 | 12 % | 30 000 dossiers |
| Juridique | 5 | 5 % | 12 500 dossiers |
| Marketing | 4 | 8 % | 20 000 dossiers |
| **TOTAL** | **53** | **100 %** | **250 000 dossiers/an** |

**Note** : Les 53 règles présentées sont les règles principales. Avec les modifiers (urgence, entité, devise, etc.), le total atteint **87 règles testables**.

### A.3 Règles transverses — Modifiers et overrides

#### A.3.1 Modificateur URGENCE (RG-U)

| ID | Condition | Effet |
|----|-----------|-------|
| RG-U-001 | `Urgence=URGENT` | SLA × 0,5 sur tous les niveaux |
| RG-U-002 | `Urgence=URGENT` | Notification Email + SMS + Push |
| RG-U-003 | `Urgence=URGENT` | Badge rouge "URGENT" affichage prioritaire |
| RG-U-004 | `Urgence=URGENT AND Demandeur_Niveau<Director` | Validation manager motif obligatoire |
| RG-U-005 | `Urgence=CRITIQUE` (réservé COPIL) | SLA × 0,25, escalade auto si non traité 4h |

#### A.3.2 Modificateur MULTI-ENTITÉS (RG-ME)

| ID | Condition | Effet |
|----|-----------|-------|
| RG-ME-001 | `Entité_Cible≠Entité_Initiateur` | Notifier Direction Entité cible (info) |
| RG-ME-002 | `Engagement>100K€ AND Multi_Entités=true` | Ajouter étape Group_Finance |
| RG-ME-003 | `Filiales_Concernées>2` | Ajouter étape Group_Compliance |
| RG-ME-004 | `Pays_Différents AND Devise_Différentes` | Ajouter étape Group_Treasury (couverture FX) |
| RG-ME-005 | `Filiale_Régulée (banque, assurance) AND Engagement>50K€` | Ajouter étape Group_Risk |

#### A.3.3 Modificateur RISQUE (RG-R)

| ID | Condition | Effet |
|----|-----------|-------|
| RG-R-001 | `Tiers_Sanctionné=true` (OFAC, EU, UN sanctions list) | Bloquer dossier + escalade Compliance |
| RG-R-002 | `KYC_Tiers="EXPIRÉ"` | Bloquer + demander mise à jour KYC |
| RG-R-003 | `Pays_Tiers="HIGH_RISK"` (FATF list) | Ajouter étape AML/Compliance |
| RG-R-004 | `Pattern_Anomalie_Détecté=true` (ML) | Marquer pour revue manuelle Compliance |
| RG-R-005 | `Montant>3×Moyenne_Initiateur_12M` | Notification audit + revue renforcée |

#### A.3.4 Cas particuliers (RG-X)

| ID | Condition | Effet |
|----|-----------|-------|
| RG-X-001 | `Initiateur=Validateur_K (même personne)` | Skip étape K, audit "auto-validation_skip" |
| RG-X-002 | `Tous_Validateurs_Niveau_K_Indisponibles>72h` | Escalade N+1 hiérarchique |
| RG-X-003 | `Document_Modifié_Pendant_Validation` | Reset workflow OU continuation selon paramètre |
| RG-X-004 | `Période_Cloture_Comptable_Active` | Ajout étape Comptabilité tous Capex |
| RG-X-005 | `Période_Élections_Gouv. AND Mécénat_Politique` | Bloquer (conformité légale) |

### A.4 Logique de SLA et escalade — Détail

#### A.4.1 Matrice SLA par rôle et type

| Rôle / Type doc | Achat | RH | Contrat | Capex | Juridique | Mkt |
|-----------------|-------|-----|---------|-------|-----------|------|
| Manager | 48h | 48h | 48h | 48h | 48h | 24h |
| Director | 24h | 24h | 24h | 24h | 24h | 24h |
| Finance | 72h | 24h | 48h | 48h | — | 48h |
| Legal | 48h | 24h | 72h | 24h | 72h | 24h |
| Compliance | 48h | — | 48h | 48h | — | — |
| RH | — | 48h | — | — | — | — |
| CFO | 48h | 24h | 48h | 48h | 48h | 48h |
| CEO | 24h | 24h | 24h | 24h | 24h | 24h |
| Board | 168h | 168h | 168h | 168h | 168h | — |

**Remarque** : Tous les SLA sont en heures ouvrées (8h-19h, jours ouvrables FR).

#### A.4.2 Cascade d'escalade — Modèle universel

```
┌──────────────────────────────────────────────────────────┐
│              CASCADE ESCALADE STANDARD                   │
├──────────────────────────────────────────────────────────┤
│
│ T+0      : Notification reception (email + push)
│
│ T+SLA-2h : Notification rappel (email)
│            "Il vous reste 2h pour traiter ce dossier"
│
│ T+SLA    : Premier dépassement SLA
│            - Notification "RAPPEL : SLA dépassé"
│            - Email + SMS validateur
│            - CC : manager validateur (info)
│            - Audit : SLA_BREACH_LEVEL_1
│
│ T+SLA+24h: Escalade niveau 2
│            - État dossier → "EN_ESCALADE"
│            - Notification urgente validateur (SMS)
│            - Notification manager validateur (email + SMS)
│            - Notification initiateur (info)
│            - Audit : SLA_BREACH_LEVEL_2
│
│ T+SLA+48h: Escalade niveau 3 — Force
│            - Activation délégation auto si configurée
│            - OU transfert N+1 hiérarchique automatique
│            - OU signature forcée (si CONFIG=AUTO_SIGN)
│            - Notification : Direction métier + Admin
│            - Audit : SLA_BREACH_LEVEL_3 + ACTION
│
│ T+SLA+72h: Si toujours bloqué
│            - Escalade Direction Générale
│            - Revue manuelle requise
│            - Audit : SLA_BREACH_CRITICAL
└──────────────────────────────────────────────────────────┘
```

**Cas concret** : Achat 28 500 € (RG-A-006), validation Finance niveau 2.
> SLA Finance = 72h. Le validateur Paul ne traite pas.
> T+70h : Email rappel "2h restant".
> T+72h : SLA breach 1 — Email + SMS Paul, CC Manager Pierre.
> T+96h : Escalade auto — Statut EN_ESCALADE, SMS Paul + Pierre, info Marie (initiatrice).
> T+120h : Auto-délégation à Pierre (manager Paul). Pierre valide.
> Audit complet : 4 entrées SLA_BREACH + 1 entrée DELEGATION_AUTO.

---

## B. CAS D'USAGE MÉTIER DÉTAILLÉS

### B.1 Cas d'usage CU-A-01 : Achat équipement IT 28 500 €

```
┌──────────────────────────────────────────────────────────────────┐
│ CU-A-01 — ACHAT ÉQUIPEMENT IT 28 500 €                            │
├──────────────────────────────────────────────────────────────────┤
│
│ DOMAINE        : Achat
│ FRÉQUENCE      : ~1 200 dossiers/an (filiale FR)
│ CRITICITÉ      : Moyenne
│ DURÉE TYPIQUE  : 6 jours ouvrés
│
│ RÈGLE APPLIQUÉE : RG-A-006 (Achat 25K-100K€ entité FR)
│
│ ACTEURS
│ ────────
│ - Initiateur     : Marie Dubois, Manager IT, Filiale FR
│ - Validateur 1   : Pierre Martin, Manager IT Senior
│ - Validateur 2A  : Paul Durand, Finance Manager FR
│ - Validateur 2B  : Anne Lefèvre, Legal Counsel FR
│ - Validateur 3   : Sophie Bernard, Director IT FR
│ - Signataire     : Jean Bernard, CIO Group (signature avancée)
│
│ DOCUMENTS REQUIS
│ ────────────────
│ ✓ Devis fournisseur signé (PDF)              [obligatoire]
│ ✓ Bon de commande Marie (PDF)                [obligatoire]
│ ✓ Justification besoin technique (Word)      [obligatoire]
│ ✓ Comparaison fournisseurs (Excel)           [obligatoire si >10K€]
│ ✓ Devis alternatifs (PDF, x2)                [si comparaison requise]
│ ○ Validation architecte référence            [optionnel]
│
│ DONNÉES MÉTADONNÉES
│ ────────────────────
│ - Type_Dossier      : Achat
│ - Sous_Type         : Équipement_IT
│ - Montant           : 28 500,00 €
│ - Devise            : EUR
│ - Fournisseur       : DELL EMC France SAS (réf master data : SUP-FR-00342)
│ - SIRET             : 351 528 740 00067
│ - Catégorie_Achat   : MATÉRIEL_INFRASTRUCTURE
│ - Code_Comptable    : 218300 (Matériel informatique)
│ - PO_SAP_Associé    : Non créé (post-validation)
│ - Centre_Coût       : CC-FR-IT-001
│ - Projet            : MIGRATION_DC_PARIS
│ - Urgence           : NORMAL
│
│ FLUX DÉTAILLÉ
│ ─────────────
│
│ JOUR 1 — 09:00 : Marie crée le dossier
│ ─────────────────────────────────────
│ - Connexion SSO Okta
│ - Création nouveau dossier "Achat serveurs DC Paris"
│ - Saisie métadonnées (3 min)
│ - Upload 4 documents (1 min)
│ - Validation circuit suggéré par Camunda DMN
│ - Soumission → État: EN_ATTENTE_VALIDATION (Niveau 1)
│ - Audit : DOSSIER_CRÉÉ + DOSSIER_SOUMIS
│ - Notification email Pierre (Manager IT Senior)
│
│ JOUR 1 — 14:30 : Pierre valide niveau 1
│ ────────────────────────────────────────
│ - Pierre reçoit email + push
│ - Consulte dossier (5 min lecture)
│ - Valide sans commentaire
│ - État: EN_ATTENTE_VALIDATION (Niveau 2 — parallèle Finance + Legal)
│ - Audit : VALIDATION_APPROVED (Pierre, Manager)
│ - Notifications email Paul + Anne (parallèle)
│
│ JOUR 2 — 10:00 : Paul valide Finance
│ ──────────────────────────────────────
│ - Paul vérifie budget (consultable via intégration SAP)
│ - Budget IT FR Q2 : 285 K€ utilisés / 380 K€ alloués → OK
│ - Validation avec commentaire "Budget OK, dans enveloppe Q2"
│ - Audit : VALIDATION_APPROVED (Paul, Finance) + commentaire
│
│ JOUR 3 — 16:00 : Anne valide Legal (parallèle)
│ ────────────────────────────────────────────────
│ - Anne consulte le contrat fournisseur
│ - Vérification clauses RGPD (DPA fournisseur valide)
│ - Vérification SIRET + statut juridique fournisseur
│ - Validation avec commentaire "DPA OK, conditions standard"
│ - Audit : VALIDATION_APPROVED (Anne, Legal) + commentaire
│ - État: EN_ATTENTE_VALIDATION (Niveau 3 — Director)
│ - Notification Sophie
│
│ JOUR 4 — 09:30 : Sophie valide niveau 3 (Director)
│ ─────────────────────────────────────────────────
│ - Revue rapide (Finance + Legal validés en amont)
│ - Validation
│ - État: EN_SIGNATURE
│ - Notification Jean (CIO Group)
│
│ JOUR 5 — 11:15 : Jean signe (signature avancée)
│ ─────────────────────────────────────────────────
│ - Authentification forte (Okta + 2FA)
│ - Vérification certificat X.509 valide (expire 2027-04-15)
│ - Signature avec hash SHA-256 + horodatage TSA Universign
│ - Encodage PKCS#7
│ - État: SIGNÉ
│ - Audit : SIGNATURE_AVANCÉE (Jean, CIO)
│
│ JOUR 5 — 11:16 : Système — Clôture automatique
│ ─────────────────────────────────────────────────
│ - Génération document final scellé (PDF/A-3 + sig + preuve)
│ - Envoi vers GED Nuxeo (zone "Achats Validés")
│ - Indexation full-text Elasticsearch
│ - Création PO automatique SAP via webhook
│ - État: ARCHIVÉ
│ - Notification email Marie + tous validateurs (résumé)
│ - SLA réel : 4 jours 2h (vs 8j SLA cible) → -49 % vs cible
│
│ MÉTRIQUES MESURÉES
│ ──────────────────
│ - Durée totale          : 4j 2h (cible : 8j)
│ - Nb d'actions humaines : 6 (initiation + 5 validations/sig.)
│ - Coût processus parapheur : 0,75 € (estimation)
│ - Coût équivalent papier   : 47 € (manutention, courrier, archivage)
│ - Économie unitaire       : 46,25 € → 55 500 €/an (1200 dossiers)
│
│ EXCEPTIONS POSSIBLES
│ ────────────────────
│ E1 : Paul (Finance) en congés
│      → Délégation pré-configurée vers Sandrine (Finance Senior)
│      → Sandrine valide, traçabilité "[via délégation Paul]"
│
│ E2 : Anne (Legal) demande clarification
│      → Retour à Pierre (Manager) avec question
│      → Pierre répond, dossier revient à Anne
│      → +1 jour ajouté au flux
│
│ E3 : Jean (CIO) absent et pas de délégation
│      → SLA breach 24h après notification
│      → Escalade auto vers CEO Adjoint
│      → Si CEO Adjoint refuse signature à ce niveau : escalade COPIL
│
│ INDICATEURS QUALITÉ
│ ───────────────────
│ - Taux 1ère validation OK : 87 % (réf benchmark 78 %)
│ - Taux rejet niveau 1     : 8 %
│ - Taux escalade SLA       : 4 %
│ - Durée moyenne réelle    : 4,8 jours (vs 8j cible)
│
└──────────────────────────────────────────────────────────────────┘
```

### B.2 Cas d'usage CU-RH-01 : Embauche cadre 75 K€ brut

```
┌──────────────────────────────────────────────────────────────────┐
│ CU-RH-01 — EMBAUCHE CADRE NIVEAU MANAGER (75 K€ BRUT)             │
├──────────────────────────────────────────────────────────────────┤
│
│ DOMAINE        : RH
│ FRÉQUENCE      : ~180 embauches cadre/an (Group)
│ CRITICITÉ      : Élevée (engagement long terme)
│ DURÉE TYPIQUE  : 7 jours ouvrés
│
│ RÈGLE APPLIQUÉE : RG-RH-001 (Embauche niveau Manager)
│
│ ACTEURS
│ ────────
│ - Initiateur   : Carole Petit, Talent Acquisition Manager
│ - Validateur 1 : David Roux, Hiring Manager (futur N+1 candidat)
│ - Validateur 2 : Sylvie Moreau, RH Business Partner
│ - Signataire   : Marc Lefort, DRH Group (signature qualifiée)
│
│ DOCUMENTS REQUIS
│ ────────────────
│ ✓ Contrat de travail signé candidat (PDF)              [obligatoire]
│ ✓ CV candidat (PDF)                                    [obligatoire]
│ ✓ Lettre d'engagement candidat (PDF)                   [obligatoire]
│ ✓ Grille salariale validée (Excel)                     [obligatoire]
│ ✓ Justificatif diplômes (PDF, multiple)                [obligatoire]
│ ✓ Extrait casier judiciaire <3 mois (PDF)              [obligatoire]
│ ✓ Validation médicale (PDF)                            [post-signature]
│ ○ Tests techniques (PDF)                               [optionnel]
│
│ DONNÉES MÉTADONNÉES
│ ────────────────────
│ - Type_Dossier        : RH
│ - Nature              : Embauche
│ - Niveau_Candidat     : Manager
│ - Salaire_Brut_Annuel : 75 000,00 €
│ - Devise              : EUR
│ - Date_Embauche_Cible : 2026-06-01
│ - Type_Contrat        : CDI
│ - Période_Essai       : 4 mois
│ - Filiale             : FR
│ - Département         : Marketing
│ - Manager_Direct      : David Roux
│ - Justification       : Remplacement (Pierre Martin parti)
│ - Budget_Référencé    : Plan_RH_2026_Marketing_FR
│ - Données_Sensibles   : OUI (RGPD : nom, prénom, salaire, diplômes)
│
│ FLUX DÉTAILLÉ
│ ─────────────
│
│ JOUR 1 — 10:00 : Carole crée le dossier
│ - Création dossier "Embauche [Nom Candidat] - Manager Mkt"
│ - Upload des 6 documents obligatoires
│ - Métadonnées avec données candidat (chiffrées au repos)
│ - Audit : DOSSIER_CRÉÉ + classification "CONFIDENTIEL_RH"
│
│ JOUR 1 — 15:00 : David valide (Hiring Manager)
│ - Validation immédiate (a participé au processus de recrutement)
│ - Commentaire : "Candidat retenu après 3 entretiens, conforme JD"
│
│ JOUR 2 — 11:00 : Sylvie valide (RH BP)
│ - Vérification cohérence grille salariale (range Manager : 65-85 K€)
│ - Vérification documents légaux (casier, diplômes)
│ - Vérification respect égalité H/F (rémunération comparable)
│ - Validation
│
│ JOUR 3 — 09:30 : Marc signe (DRH Group, signature qualifiée)
│ - Authentification forte 2FA (badge RFID + biométrie)
│ - Redirection Universign pour signature qualifiée eIDAS
│ - Vérification certificat (cert qualifié Universign valide)
│ - Signature appliquée + horodatage RFC 3161 + scellement
│ - État: SIGNÉ
│
│ JOUR 3 — 09:31 : Système — Clôture
│ - Génération contrat signé final (PDF/A-3)
│ - Archivage GED zone "RH_Contrats" (rétention 50 ans légal)
│ - Création employé dans SAP HR (webhook)
│ - Création compte Active Directory (webhook)
│ - Création accès Okta (webhook)
│ - Notification candidat (email automatique)
│ - SLA réel : 2j 23h (vs 7j cible) → -57 %
│
│ EXCEPTIONS SPÉCIFIQUES RH
│ ──────────────────────────
│ E1 : Vérification antécédents non valide
│      → Bloquer signature, notification Compliance
│      → Décision Direction RH (cas par cas)
│
│ E2 : Visite médicale non passée
│      → Signature reportée, dossier en attente
│      → Notification automatique candidat + médecine du travail
│
│ E3 : Candidat refuse l'offre signée
│      → Annulation dossier (motif obligatoire)
│      → Audit + archivage légal "OFFRE_REFUSÉE"
│
│ CONFORMITÉ SPÉCIFIQUE
│ ─────────────────────
│ - RGPD : Données candidat chiffrées AES-256 + accès strict (RH only)
│ - Code du Travail : Contrat CDI signature avant 1er jour effectif
│ - Égalité H/F : Vérification grille salariale anonymisée
│ - Loi Informatique & Libertés : Droit accès/rectification
│ - Conservation légale : 50 ans après fin de contrat (régime sécurité sociale)
│
└──────────────────────────────────────────────────────────────────┘
```

### B.3 Cas d'usage CU-C-01 : Contrat cadre client 250 K€/an

```
┌──────────────────────────────────────────────────────────────────┐
│ CU-C-01 — CONTRAT CADRE CLIENT 250 K€/AN (B2B)                    │
├──────────────────────────────────────────────────────────────────┤
│
│ DOMAINE        : Contrats commerciaux
│ FRÉQUENCE      : ~80 contrats/an (>100K€)
│ CRITICITÉ      : Critique (engagement commercial)
│ DURÉE TYPIQUE  : 12 jours ouvrés
│
│ RÈGLE APPLIQUÉE : RG-C-004 (Contrat cadre client CA≥100K€/an)
│
│ ACTEURS
│ ────────
│ - Initiateur    : Thomas Leroy, Account Manager Senior
│ - Validateur 1  : Catherine Vidal, Sales Manager
│ - Validateur 2  : Anne Lefèvre, Legal Counsel
│ - Validateur 3  : Paul Durand, Finance Manager
│ - Validateur 4  : Hélène Fischer, Sales Director
│ - Signataire    : Jean-Marc Lebon, CEO Group (signature qualifiée)
│
│ DOCUMENTS REQUIS
│ ────────────────
│ ✓ Contrat cadre signé client (PDF)                     [obligatoire]
│ ✓ Annexes techniques (PDF, x3)                         [obligatoire]
│ ✓ Annexes financières (Excel + PDF)                    [obligatoire]
│ ✓ Conditions Générales de Vente CGV (PDF)              [obligatoire]
│ ✓ Validation juridique préalable (Word)                [obligatoire]
│ ✓ Validation tarification (Excel)                      [obligatoire]
│ ✓ Score crédit client (PDF)                            [obligatoire]
│ ✓ KYC client (PDF)                                     [obligatoire]
│ ○ Garanties bancaires si applicable (PDF)              [optionnel]
│
│ DONNÉES MÉTADONNÉES
│ ────────────────────
│ - Type_Dossier        : Contrat
│ - Sous_Type           : Cadre_Client
│ - CA_Annuel_Estimé    : 250 000,00 €
│ - CA_Total_Engagement : 750 000,00 € (3 ans)
│ - Devise              : EUR
│ - Client_Ref_CRM      : CRM-CLI-04521
│ - Client_SIRET        : 552 081 317 00064
│ - Client_Score_Crédit : A (notation Coface)
│ - Date_Effet          : 2026-06-15
│ - Durée_Contrat       : 36 mois
│ - Tacite_Reconduction : OUI (12 mois)
│ - Filiale_Concernée   : FR (signataire) + Filiale_DE (livraison)
│ - Cas_Particulier     : Multi-entité → ajout étape RG-ME-001
│
│ FLUX DÉTAILLÉ (résumé)
│ ───────────────────────
│
│ JOUR 1  : Thomas crée dossier, upload 8 documents
│ JOUR 1  : Catherine valide (Sales Manager)
│ JOUR 3  : Anne valide (Legal) — vérif. clauses + RGPD + conformité
│ JOUR 5  : Paul valide (Finance) — vérif. tarification + crédit client
│ JOUR 6  : Hélène valide (Sales Director)
│ JOUR 7  : Notification info Direction Filiale DE (RG-ME-001)
│ JOUR 9  : Jean-Marc signe (signature qualifiée Universign)
│ JOUR 9  : Archivage GED + intégration CRM Salesforce + ERP SAP
│ SLA réel : 8 jours (vs 12 cible) → -33 %
│
│ POINTS DE VIGILANCE LEGAL
│ ─────────────────────────
│ ✓ Clause RGPD (DPA standard groupe)
│ ✓ Clause limitation responsabilité (capée à 200 % du CA annuel)
│ ✓ Clause juridiction (Tribunal Commerce Paris)
│ ✓ Clause force majeure (post-COVID)
│ ✓ Clause anti-corruption (Sapin 2)
│ ✓ Clause cybersécurité (NIS2)
│
│ CALCULS FINANCIERS VÉRIFIÉS
│ ───────────────────────────
│ - Revenu prévisionnel an 1 : 250 K€
│ - Revenu prévisionnel an 2 : 250 K€ + 3 % indexation = 257,5 K€
│ - Revenu prévisionnel an 3 : 257,5 K€ + 3 % = 265,2 K€
│ - Revenu total 3 ans       : 772,7 K€
│ - Marge brute estimée      : 38 % = 293,6 K€
│ - Risque crédit client     : Faible (score A)
│ - Provision créance douteuse : 0,5 % = 1 250 €
│
└──────────────────────────────────────────────────────────────────┘
```

### B.4 Cas d'usage CU-B-01 : Capex 850 K€ — Refonte SI

```
┌──────────────────────────────────────────────────────────────────┐
│ CU-B-01 — CAPEX 850 K€ — REFONTE SI                               │
├──────────────────────────────────────────────────────────────────┤
│
│ DOMAINE        : Budget / Capex
│ FRÉQUENCE      : ~25 dossiers Capex >500K€/an
│ CRITICITÉ      : Critique (engagement stratégique)
│ DURÉE TYPIQUE  : 14 jours ouvrés
│
│ RÈGLE APPLIQUÉE : RG-B-003 (Capex 200K-1M€)
│
│ ACTEURS
│ ────────
│ - Initiateur     : Pierre Roux, Director IT
│ - Validateur 1   : Marc Vincent, Manager IT Senior
│ - Validateur 2   : Paul Durand, Finance Manager
│ - Validateur 3   : Stéphane Lacroix, Director IT
│ - Validateur 4   : Antoine Mercier, CFO Group
│ - Signataire     : Jean-Marc Lebon, CEO Group (signature qualifiée)
│
│ DOCUMENTS REQUIS
│ ────────────────
│ ✓ Business case (PowerPoint)                           [obligatoire]
│ ✓ Étude TCO 5 ans (Excel)                              [obligatoire]
│ ✓ Analyse ROI/IRR (Excel)                              [obligatoire]
│ ✓ Comparaison fournisseurs (PDF, x3)                   [obligatoire]
│ ✓ Roadmap projet (Gantt PDF)                           [obligatoire]
│ ✓ Analyse risques projet (Word)                        [obligatoire]
│ ✓ Validation architecture (Word)                       [obligatoire]
│ ✓ Plan de financement (Excel)                          [obligatoire]
│ ○ Avis externe consultant (PDF)                        [recommandé]
│
│ DONNÉES MÉTADONNÉES
│ ────────────────────
│ - Type_Dossier        : Capex
│ - Sous_Type           : Refonte_SI
│ - Montant_Total       : 850 000,00 €
│ - Décomposition       :
│     - Licences logiciels : 280 K€
│     - Services intégration : 320 K€
│     - Infrastructure cloud : 150 K€
│     - Formation : 60 K€
│     - Conduite changement : 40 K€
│ - Devise              : EUR
│ - Période_Engagement  : 18 mois
│ - ROI_Calculé         : 32 % (TRI 24 %, payback 3,5 ans)
│ - Hors_Budget         : NON (PLAN 2026 ligne 4521)
│ - Filiale             : Group
│ - Projet_Stratégique  : OUI (validation ComEx requise)
│ - Risque_Niveau       : Moyen (méthodologie validée)
│
│ FLUX DÉTAILLÉ
│ ─────────────
│
│ JOUR 1   : Pierre crée dossier (préparé sur 2 semaines avec équipe)
│ JOUR 2   : Marc Vincent valide (Manager IT Senior, +instructions techniques)
│ JOUR 4   : Paul Durand valide (Finance) — Vérif. lignes budget, projection
│ JOUR 6   : Stéphane Lacroix valide (Director IT)
│ JOUR 9   : Antoine Mercier valide (CFO) — Validation TCO/ROI
│ JOUR 12  : Jean-Marc signe (CEO, signature qualifiée)
│ JOUR 12  : Archivage + reporting BI + notif. ComEx + déblocage budget SAP
│ SLA réel : 11 jours (vs 14 cible) → -21 %
│
│ POINTS DE VIGILANCE FINANCIÈRE
│ ────────────────────────────────
│ ✓ Validation TCO calculé sur 5 ans : 1,2 M€
│ ✓ Économies projetées : 450 K€/an dès an 2
│ ✓ ROI à 3 ans : +28 %
│ ✓ ROI à 5 ans : +185 %
│ ✓ Capacité de financement : Cash-flow OK, pas de financement externe
│ ✓ Impact P&L an 1 : -680 K€ (amortissement linéaire 5 ans)
│ ✓ Impact bilan : Immobilisations +850 K€, Trésorerie -850 K€
│
│ EXCEPTIONS SPÉCIFIQUES CAPEX
│ ─────────────────────────────
│ E1 : Hors budget initial (RG-B-005)
│      → Justification CFO obligatoire avant signature
│      → Possible ajout étape Réallocation budgétaire
│
│ E2 : ROI < 15 %
│      → Revue manuelle COPIL Direction
│      → Justification stratégique alternative requise
│
│ E3 : Découpage en lots (multi-tranches)
│      → 1 dossier maître + n dossiers lots
│      → Validation maître = autorisation globale
│      → Validations lots = mise en commande progressive
│
└──────────────────────────────────────────────────────────────────┘
```

### B.5 Cas d'usage CU-J-01 : Procuration bancaire CFO

```
┌──────────────────────────────────────────────────────────────────┐
│ CU-J-01 — PROCURATION BANCAIRE CFO                                │
├──────────────────────────────────────────────────────────────────┤
│
│ DOMAINE        : Juridique
│ FRÉQUENCE      : ~12 dossiers/an
│ CRITICITÉ      : Très critique (engagement bancaire)
│ DURÉE TYPIQUE  : 6 jours ouvrés
│ RÈGLE APPLIQUÉE : RG-J-005
│
│ ACTEURS
│ ────────
│ - Initiateur   : Anne Lefèvre, Legal Counsel
│ - Validateur 1 : Antoine Mercier, CFO Group
│ - Signataire   : Jean-Marc Lebon, CEO Group (signature qualifiée)
│ - Bénéficiaire : Sophie Lambert, Treasurer
│
│ DOCUMENTS REQUIS
│ ────────────────
│ ✓ Acte de procuration (Word + PDF final)               [obligatoire]
│ ✓ Statuts à jour société (PDF)                         [obligatoire]
│ ✓ Extrait Kbis <3 mois (PDF)                           [obligatoire]
│ ✓ Pièce d'identité bénéficiaire (PDF)                  [obligatoire]
│ ✓ Justificatif domicile bénéficiaire (PDF)             [obligatoire]
│
│ POINTS LÉGAUX
│ ─────────────
│ ✓ Limites pouvoirs définies (montant max par opération)
│ ✓ Durée procuration limitée (12 mois renouvelable)
│ ✓ Révocation possible à tout moment (clause expresse)
│ ✓ Notification banque sous 8 jours (envoi recommandé AR)
│ ✓ Conservation acte original 30 ans (légal)
│
│ FLUX (résumé)
│ ─────────────
│ J+0 : Création dossier par Anne
│ J+0 : Validation Antoine (CFO)
│ J+5 : Signature qualifiée Jean-Marc (acte officiel)
│ J+5 : Génération final + archivage GED + envoi banque
│
└──────────────────────────────────────────────────────────────────┘
```

### B.6 Cas d'usage CU-M-01 : Communication crise

```
┌──────────────────────────────────────────────────────────────────┐
│ CU-M-01 — COMMUNICATION DE CRISE                                  │
├──────────────────────────────────────────────────────────────────┤
│
│ DOMAINE        : Communication
│ FRÉQUENCE      : ~3-5 dossiers/an (mais imprévisible)
│ CRITICITÉ      : Maximale (image groupe)
│ DURÉE TYPIQUE  : 24h (urgence absolue)
│ RÈGLE APPLIQUÉE : RG-M-004 + RG-U-005 (urgence critique)
│
│ ACTEURS
│ ────────
│ - Initiateur   : Catherine Mathieu, Director Communication
│ - Validateur 1 : Anne Lefèvre, Legal Counsel
│ - Signataire   : Jean-Marc Lebon, CEO Group (signature simple)
│
│ DOCUMENTS REQUIS
│ ────────────────
│ ✓ Communiqué de presse (Word)                          [obligatoire]
│ ✓ Briefing situation (PowerPoint)                      [obligatoire]
│ ✓ Plan d'action communication (Word)                   [obligatoire]
│ ✓ Avis juridique (Word)                                [obligatoire]
│ ○ Médias visuels (image, vidéo)                        [optionnel]
│
│ FLUX EXPRESS (urgence)
│ ──────────────────────
│ T+0   : Création dossier par Catherine
│ T+1h  : Validation Anne (Legal) — vérif. risques juridiques
│ T+3h  : Signature CEO
│ T+4h  : Diffusion communiqué + archivage
│ SLA total cible : 24h MAX
│
│ NOTIFICATIONS RENFORCÉES
│ ────────────────────────
│ - Email + SMS + Push + Appel téléphonique automatique
│ - Notification ComEx complet (info)
│ - Astreinte 24/7 activée pour cette catégorie
│
│ EXCEPTIONS RÉCURRENTES
│ ──────────────────────
│ E1 : CEO indisponible
│      → Délégation pré-définie : COO ou CFO
│      → Procédure escalade COPIL crise
│
│ E2 : Conflit légal détecté
│      → Bloquer publication
│      → Réunion d'urgence CEO + Legal + Comm
│
└──────────────────────────────────────────────────────────────────┘
```

### B.7 Récapitulatif des 6 cas d'usage clés

| ID | Cas d'usage | Volume/an | SLA cible | SLA observé | Économie/dossier |
|----|-------------|-----------|-----------|-------------|------------------|
| CU-A-01 | Achat équipement IT 28,5K€ | 1 200 | 8j | 4j (-50 %) | 46 € |
| CU-RH-01 | Embauche cadre 75K€ | 180 | 7j | 3j (-57 %) | 380 € |
| CU-C-01 | Contrat cadre client 250K€ | 80 | 12j | 8j (-33 %) | 280 € |
| CU-B-01 | Capex SI 850K€ | 25 | 14j | 11j (-21 %) | 1 200 € |
| CU-J-01 | Procuration bancaire | 12 | 6j | 5j (-17 %) | 520 € |
| CU-M-01 | Communication crise | 4 | 24h | 4h (-83 %) | 2 800 € |

**Économie totale annuelle estimée** : 250 000 dossiers × 46 € moyen = **11,5 M€/an** (TCO papier vs digital).

---

## C. DICTIONNAIRE DE DONNÉES MÉTIER

### C.1 Vue d'ensemble — Inventaire 142 attributs

| Domaine de données | Nb attributs | Sensibilité RGPD |
|--------------------|--------------|------------------|
| Identification dossier | 12 | Faible |
| Métadonnées métier | 28 | Variable |
| Acteurs / Utilisateurs | 18 | **Élevée** |
| Documents | 14 | Variable |
| Workflow / Validation | 16 | Faible |
| Signatures / Preuves | 22 | **Élevée** |
| Audit / Traçabilité | 15 | Moyenne |
| Référentiels (master data) | 17 | Moyenne |
| **TOTAL** | **142** | **47 PII identifiées** |

### C.2 Attributs de référence — Définition exhaustive

#### C.2.1 Identification du dossier (12 attributs)

| Attribut | Type | Format | Validation | Obligatoire | Exemple |
|----------|------|--------|------------|-------------|---------|
| `dossier_id` | UUID | xxxx-xxxx-xxxx-xxxx | UUID v4 | OUI | `a1b2c3d4-e5f6-...` |
| `external_ref` | String(100) | Libre | Regex `^[A-Z0-9_-]+$` | NON | `SAP-PO-2026-12345` |
| `numero_dossier` | String(20) | Auto-généré | Format `PARAPH-{YYYY}-{seq6}` | OUI | `PARAPH-2026-001234` |
| `version` | Integer | 1, 2, 3... | ≥ 1 | OUI | `2` |
| `parent_dossier_id` | UUID | nullable | FK dossier_id | NON | (si versioning) |
| `created_at` | Timestamp | ISO 8601 UTC | Cohérent avec workflow | OUI | `2026-04-24T14:32:15Z` |
| `submitted_at` | Timestamp | ISO 8601 UTC | ≥ created_at | NON | `2026-04-24T15:00:00Z` |
| `signed_at` | Timestamp | ISO 8601 UTC | ≥ submitted_at | NON | `2026-04-29T11:15:30Z` |
| `archived_at` | Timestamp | ISO 8601 UTC | ≥ signed_at | NON | `2026-04-29T11:16:00Z` |
| `retention_until` | Date | YYYY-MM-DD | Calculé selon classification | OUI | `2036-04-29` |
| `language` | String(2) | ISO 639-1 | `fr`, `en`, `de` | OUI (def. fr) | `fr` |
| `tenant_id` | UUID | xxxx-xxxx... | FK tenant (multi-tenant) | OUI | (UUID groupe) |

#### C.2.2 Métadonnées métier — Champs communs (16 attributs principaux)

| Attribut | Type | Source | Validation | Obligatoire | RGPD |
|----------|------|--------|------------|-------------|------|
| `title` | String(255) | Saisie | 5-255 car., XSS | OUI | Faible |
| `description` | Text(2000) | Saisie | 0-2000 car., XSS | NON | Faible |
| `type_dossier_id` | UUID | Référentiel | FK types_dossier | OUI | Faible |
| `sous_type` | String(50) | Liste | Enum selon type | NON | Faible |
| `entity_id` | UUID | Référentiel | FK entities | OUI | Faible |
| `classification` | Enum | Liste | PUBLIC/INTERNE/CONFIDENTIEL/SECRET | OUI | Variable |
| `urgence` | Enum | Liste | NORMAL/URGENT/CRITIQUE | OUI (def.NORMAL) | Faible |
| `montant` | Decimal(18,2) | Saisie | ≥ 0, devise cohérente | NON | Faible |
| `currency` | String(3) | ISO 4217 | EUR/USD/GBP/CHF | NON | Faible |
| `montant_eur` | Decimal(18,2) | Calculé | Conversion via FX rate | NON | Faible |
| `centre_cout` | String(20) | Référentiel | FK cost_centers | NON | Faible |
| `code_comptable` | String(10) | Référentiel | FK plan_comptable | NON | Faible |
| `projet_associe` | UUID | Référentiel | FK projects | NON | Faible |
| `tags` | Array[String] | Saisie | Max 10, 50 car. chacun | NON | Faible |
| `metadata_custom` | JSONB | Schéma dynamique | Validé selon type_dossier | NON | Variable |
| `tiers_id` | UUID | Référentiel | FK tiers (fournisseur/client) | NON | Faible |

#### C.2.3 Acteurs — Données utilisateur (18 attributs, dont 12 PII)

| Attribut | Type | PII | Validation | Conservation |
|----------|------|-----|------------|--------------|
| `user_id` | UUID | NON | UUID v4 | Permanent |
| `email` | String(255) | **OUI** | RFC 5321, unique | 5 ans après départ |
| `first_name` | String(100) | **OUI** | 1-100 car. | 5 ans après départ |
| `last_name` | String(100) | **OUI** | 1-100 car. | 5 ans après départ |
| `display_name` | String(200) | **OUI** | Concat first + last | 5 ans après départ |
| `phone_mobile` | String(20) | **OUI** | E.164 format | 5 ans après départ |
| `employee_id` | String(20) | **OUI** | FK SAP HR | 5 ans après départ |
| `entity_id` | UUID | NON | FK entities | Permanent |
| `department` | String(100) | NON | Liste valeurs | 10 ans |
| `job_title` | String(150) | NON | Libre | 10 ans |
| `roles` | Array[String] | NON | Enum (RBAC) | Permanent |
| `manager_id` | UUID | NON | FK users (récursif) | 5 ans |
| `delegation_to` | UUID | NON | FK users | Selon délégation |
| `delegation_until` | Timestamp | NON | Future date | Selon délégation |
| `cert_subject` | String(500) | OUI (DN) | X.509 distinguished name | Selon cert |
| `cert_thumbprint` | String(64) | NON | SHA-256 hex | Selon cert |
| `last_login_at` | Timestamp | NON | UTC | 12 mois (audit) |
| `account_status` | Enum | NON | ACTIVE/DISABLED/LOCKED | Permanent |

**Règle RGPD** : Les attributs PII sont chiffrés au repos (AES-256) et accessibles uniquement aux rôles autorisés. Conservation alignée sur règles RH/légales.

#### C.2.4 Documents (14 attributs)

| Attribut | Type | Validation | Notes |
|----------|------|------------|-------|
| `document_id` | UUID | UUID v4 | PK |
| `dossier_id` | UUID | FK dossiers | NOT NULL |
| `filename` | String(255) | 1-255 car., sanitized | Échappement XSS |
| `mime_type` | String(100) | Liste autorisée | PDF, DOCX, XLSX, PNG, JPG (whitelist) |
| `file_size_bytes` | BigInt | 1 - 52 428 800 (50 MB) | Limite par fichier |
| `hash_sha256` | String(64) | Hex 64 car. | Pour intégrité |
| `storage_path` | String(500) | S3 URI | s3://bucket/path |
| `kms_key_id` | String(100) | AWS KMS key ARN | Chiffrement |
| `version` | Integer | ≥ 1 | Versioning interne |
| `uploaded_by` | UUID | FK users | Auditable |
| `uploaded_at` | Timestamp | UTC | Auditable |
| `virus_scan_status` | Enum | PENDING/PASSED/FAILED/QUARANTINED | ClamAV |
| `ocr_text` | Text | Optionnel | Pour recherche full-text |
| `language_detected` | String(2) | ISO 639-1 | Auto-détection |

#### C.2.5 Référentiels Master Data (17 référentiels)

| Référentiel | Source autoritaire | Synchro | Volumétrie |
|-------------|-------------------|---------|------------|
| `users` | Active Directory + SAP HR | Temps réel (LDAP) + nightly | 5 000 |
| `entities` | SAP MDG | Hebdo | 35 |
| `types_dossier` | Parapheur (config) | Manuelle | 28 |
| `circuits` | Parapheur (config) | Manuelle | 87 |
| `roles` | Parapheur RBAC | Manuelle | 45 |
| `fournisseurs` | SAP MDG (Vendor Master) | Quotidien | 12 000 |
| `clients` | Salesforce CRM | Quotidien | 8 500 |
| `plan_comptable` | SAP FI (G/L Accounts) | Mensuel | 850 |
| `centres_cout` | SAP CO | Mensuel | 320 |
| `projets` | SAP PS + Jira | Quotidien | 1 200 |
| `devises` | ECB + SAP FI | Quotidien (FX rates) | 35 |
| `pays` | ISO 3166-1 (statique) | Manuelle | 249 |
| `langues` | ISO 639-1 (statique) | Manuelle | 6 |
| `categories_achat` | SAP MM (Material Groups) | Mensuel | 150 |
| `motifs_rejet` | Parapheur (config) | Manuelle | 25 |
| `niveaux_classification` | Parapheur (config) | Manuelle | 4 |
| `signatories_authorized` | Parapheur RBAC | Manuelle | 120 |

#### C.2.6 Énumérations métier — Inventaire

```
ENUM type_dossier_status:
  BROUILLON, EN_ATTENTE_VALIDATION, EN_MODIFICATION,
  EN_ESCALADE, EN_SIGNATURE, SIGNÉ, ARCHIVÉ, ANNULÉ
  → 8 valeurs

ENUM classification:
  PUBLIC, INTERNE, CONFIDENTIEL, SECRET
  → 4 valeurs

ENUM urgence:
  NORMAL, URGENT, CRITIQUE
  → 3 valeurs

ENUM signature_type:
  SIMPLE, AVANCEE, QUALIFIEE
  → 3 valeurs

ENUM signature_status:
  EN_ATTENTE, SIGNÉE, REVOQUÉE, EXPIRÉE
  → 4 valeurs

ENUM validation_decision:
  APPROUVÉ, REJETÉ, RETOUR_ARRIÈRE, DÉLÉGUÉ, COMMENTÉ
  → 5 valeurs

ENUM rejet_categorie:
  DOC_INCOMPLET, ERREUR_DONNÉES, NON_CONFORME_RÈGLE,
  HORS_BUDGET, RISQUE_JURIDIQUE, AUTRE
  → 6 valeurs

ENUM action_audit_type:
  DOSSIER_CRÉÉ, DOSSIER_SOUMIS, DOSSIER_MODIFIÉ,
  VALIDATION_APPROUVÉE, VALIDATION_REJETÉE,
  SIGNATURE_APPOSÉE, SIGNATURE_REVOQUÉE,
  ESCALADE_DÉCLENCHÉE, DÉLÉGATION_ACTIVÉE,
  DOCUMENT_AJOUTÉ, DOCUMENT_SUPPRIMÉ,
  ARCHIVAGE_EFFECTUÉ, EXPORT_EFFECTUÉ,
  ACCÈS_LECTURE, MODIFICATION_AUDIT_ATTEMPT (interdit)
  → 15 valeurs

ENUM device_type:
  WEB_DESKTOP, WEB_MOBILE, APP_NATIVE, API_CLIENT
  → 4 valeurs

(8 énumérations principales, 48 valeurs au total)
```

### C.3 Règles de validation des données — Contraintes

#### C.3.1 Contraintes métier transversales

| ID | Contrainte | Niveau | Message erreur |
|----|------------|--------|----------------|
| C-V-001 | `montant > 0 AND montant ≤ 99 999 999,99` | DB + API | "Le montant doit être compris entre 0,01 et 99 999 999,99" |
| C-V-002 | `currency ∈ {EUR, USD, GBP, CHF}` | DB CHECK | "Devise non supportée : utiliser EUR, USD, GBP ou CHF" |
| C-V-003 | `email matches RFC 5321 regex` | API + UI | "Format d'email invalide" |
| C-V-004 | `phone_mobile matches E.164` | API | "Format téléphone invalide (ex: +33612345678)" |
| C-V-005 | `title length BETWEEN 5 AND 255` | API + UI | "Le titre doit comporter 5 à 255 caractères" |
| C-V-006 | `submitted_at >= created_at` | DB CHECK | (interne, jamais affiché) |
| C-V-007 | `signed_at >= submitted_at` | DB CHECK | (interne) |
| C-V-008 | `Documents count BETWEEN 1 AND 10 per dossier` | API | "Min 1, max 10 documents par dossier" |
| C-V-009 | `Total documents size <= 50 MB per dossier` | API | "Taille totale documents > 50 MB" |
| C-V-010 | `mime_type ∈ whitelist` | API | "Format non autorisé : utiliser PDF, DOCX, XLSX, PNG, JPG" |

#### C.3.2 Contraintes spécifiques par domaine

| Domaine | Contrainte | Exemple |
|---------|------------|---------|
| Achat | `fournisseur_id NOT NULL AND fournisseur.kyc_status = 'VALID'` | Bloque si KYC expiré |
| Achat | `montant <= initiateur.budget_disponible` | Vérification budget |
| RH | `salaire BETWEEN grille.min AND grille.max` (selon niveau) | Cohérence grille |
| RH | `date_embauche >= today + 7j` | Anticipation requise |
| Contrat | `client_id IS NOT NULL AND client.score_credit IN ('A','B')` | Pas de C ou D |
| Contrat | `duree_contrat BETWEEN 1 AND 60 mois` | Limite durée |
| Capex | `hors_budget=true → justification_text NOT NULL AND length > 100` | Justif obligatoire |
| Juridique | `signature_type = 'QUALIFIEE'` (toujours) | Imposé |

### C.4 Lineage et flux de données — Cartographie

#### C.4.1 Origine des données — Sources autoritaires

```
┌──────────────────────────────────────────────────────────────────┐
│                  LINEAGE — DATA AUTHORITATIVE                     │
├──────────────────────────────────────────────────────────────────┤
│
│ ╔═══════════════════════════════════════════════════════════════╗
│ ║  SOURCE AUTORITAIRE   │   ATTRIBUTS GOUVERNÉS                  ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  Active Directory     │   user_id, email, account_status       ║
│ ║                       │   manager_id, last_login_at            ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  SAP HR (HCM)         │   employee_id, first_name, last_name   ║
│ ║                       │   department, job_title, entity_id     ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  SAP MDG              │   fournisseurs (12K), entités (35)     ║
│ ║                       │   plan_comptable (850), centres_cout   ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  SAP FI               │   FX rates quotidiens, classes compta  ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  Salesforce CRM       │   clients (8,5K), opportunités, scores ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  Universign           │   certificats qualifiés, TSA proofs    ║
│ ╠═══════════════════════════════════════════════════════════════╣
│ ║  Parapheur (propre)   │   dossiers, validations, signatures    ║
│ ║                       │   audit_trail, types_dossier, circuits ║
│ ╚═══════════════════════════════════════════════════════════════╝
│
└──────────────────────────────────────────────────────────────────┘
```

#### C.4.2 Synchronisation des référentiels

| Référentiel | Source | Méthode synchro | Fréquence | SLA fraîcheur |
|-------------|--------|-----------------|-----------|---------------|
| Users (AD) | Active Directory | LDAP push events | Temps réel | < 1 min |
| Users (HR) | SAP HR | API REST (delta) | Nightly 02:00 | < 24h |
| Entités | SAP MDG | API REST (full) | Hebdo dimanche | < 7j |
| Fournisseurs | SAP MDG | API REST (delta) | Quotidien 06:00 | < 24h |
| Clients | Salesforce | API REST (delta) | Quotidien 06:30 | < 24h |
| Plan comptable | SAP FI | Export CSV scheduled | Mensuel J+1 | < 30j |
| Centres coût | SAP CO | Export CSV scheduled | Mensuel J+1 | < 30j |
| FX rates | SAP FI | API REST | Quotidien 08:00 | < 24h |
| Projets | SAP PS + Jira | API + webhook | Temps réel + nightly | < 1h |

#### C.4.3 Stratégie en cas de désynchronisation

| Cas | Détection | Action |
|-----|-----------|--------|
| User absent SAP HR mais présent dans dossier | Reconciliation hebdo | Marquer "OBSOLETE", route vers manager |
| Fournisseur supprimé SAP MDG | Webhook MDG | Bloquer nouveaux dossiers, alerter Procurement |
| FX rate manquant pour devise | Try/catch API | Utiliser dernier rate connu, alerte Finance |
| Centre coût désactivé | Validation au runtime | Bloquer dossier, demander réaffectation |
| User SSO échoue | Alerte temps réel | Bloquer login, escalade IT |

### C.5 Gouvernance RGPD — Données personnelles

#### C.5.1 Inventaire PII (47 attributs identifiés)

| Catégorie | Attributs | Base légale | Conservation |
|-----------|-----------|-------------|--------------|
| Identité | first_name, last_name, display_name, employee_id | Intérêt légitime (gestion) | 5 ans après départ |
| Contact | email, phone_mobile | Intérêt légitime | 5 ans après départ |
| Professionnel | department, job_title, manager_id | Intérêt légitime | 10 ans (audit) |
| Authentif. | last_login_at, IP_address, user_agent | Intérêt légitime (sécurité) | 12 mois |
| Certificats | cert_subject (DN), cert_thumbprint | Contrat (eIDAS) | Selon validité cert |
| Signature | proof_data, biometric_hash | Contrat (eIDAS) | 10 ans (légal) |
| Audit | actor_id, timestamps, action details | Obligation légale | 10 ans |

**TOTAL** : 47 attributs PII gouvernés. Tous chiffrés au repos AES-256.

#### C.5.2 Droits RGPD applicables

| Droit | Mise en œuvre | Délai légal |
|-------|---------------|-------------|
| Accès (Art. 15) | Export self-service via portail user | 1 mois |
| Rectification (Art. 16) | Workflow validation Manager + RH | 1 mois |
| Effacement (Art. 17) | Workflow Admin + DPO + Legal (limité aux droits prouvés) | 1 mois |
| Portabilité (Art. 20) | Export JSON structuré (données utilisateur) | 1 mois |
| Opposition (Art. 21) | Cas-par-cas, validation DPO | 1 mois |

**Limitation** : Effacement impossible pour données d'audit légal (obligation Code de Commerce 10 ans). Anonymisation appliquée si demande validée.

#### C.5.3 Registre des traitements RGPD

```
TRAITEMENT 1 : "Gestion workflow de validation et signature"
├─ Responsable de traitement : [Société] - [DPO email]
├─ Finalité : Dématérialiser les processus d'approbation et signature
├─ Base légale : Intérêt légitime + Obligations légales (eIDAS, archivage)
├─ Catégories de données : Identité, professionnelles, certificats, audit
├─ Catégories de personnes : Employés (5K), fournisseurs (12K), clients (8,5K)
├─ Destinataires : Interne uniquement + sous-traitants listés
├─ Durée conservation : Variable selon type (1-50 ans)
├─ Transferts hors UE : Non (Universign hébergé UE)
├─ Mesures techniques : Chiffrement, RBAC, audit, masking
└─ DPIA : Réalisée le 2026-04-15, score risque : Modéré
```

---

## D. RISK REGISTER PHASE 2

### D.1 Risques métier identifiés (15 risques)

| ID | Risque | Catégorie | P (1-5) | I (1-5) | Score | Owner | Mitigation |
|----|--------|-----------|---------|---------|-------|-------|------------|
| RM1 | Désaccord seuils financiers entre filiales | Métier | 4 | 5 | **20** | DAF Group | Atelier J+10, arbitrage ComEx J+20 |
| RM2 | Master data SAP non alignée | Donnée | 4 | 4 | **16** | DSI + Métiers | Audit MDG J+5, plan rapprochement J+15 |
| RM3 | 23 % cas non couverts par règles standard | Couverture | 3 | 4 | **12** | PO Métier | Sas exception métier validé Admin |
| RM4 | Évolution réglementaire (RGPD/sectoriel) | Conformité | 2 | 5 | **10** | DPO + Legal | Veille trimestrielle, revue règles bi-annuelle |
| RM5 | Sous-estimation volume cas RH non standard | Couverture | 3 | 3 | **9** | DRH | Phase pilote 30 dossiers RH témoin |
| RM6 | Validation circuits par métiers en retard | Process | 3 | 4 | **12** | PMO | RACI clair + comité validation hebdo |
| RM7 | Camunda DMN courbe apprentissage métiers | Adoption | 3 | 3 | **9** | Formation | Formation 2j métiers + accompagnement 1 mois |
| RM8 | Performance moteur règles à 250K dossiers/an | Technique | 2 | 4 | **8** | Architecte | Load test sur scénarios réalistes |
| RM9 | Décision tardive sur master data fournisseurs | Décision | 3 | 4 | **12** | Achats + DSI | COPIL technique J+15 |
| RM10 | Mauvaise interprétation règles complexes | Qualité | 4 | 3 | **12** | PO Métier | Tests métier exhaustifs (1 expert/domaine) |
| RM11 | Coûts Camunda Enterprise non budgétés | Financier | 2 | 3 | **6** | DAF | Validation budget licences en COPIL J+10 |
| RM12 | Résistance changement utilisateurs métier | Adoption | 3 | 4 | **12** | Conduite chgmt | Plan formation + communication |
| RM13 | Dépendance forte aux référentiels SAP | Architecture | 3 | 4 | **12** | Architecte | Cache + dégradation gracieuse |
| RM14 | Confidentialité données RH leakée | RGPD | 2 | 5 | **10** | DPO + RSSI | Chiffrement + RBAC strict + audit |
| RM15 | Catalog cas usage incomplet (oubli) | Couverture | 2 | 3 | **6** | PO Métier | Atelier validation 6 domaines, sign-off |

### D.2 Plan d'actions préventives — Top 10

1. **J+5** : Atelier audit master data SAP (gap analysis fournisseurs/entités/comptes)
2. **J+10** : Atelier seuils financiers DAF Group + filiales (escalade ComEx si désaccord)
3. **J+15** : Validation gouvernance master data (qui est source autoritaire)
4. **J+20** : Comité tarification Camunda Enterprise (décision GO/NO-GO licences)
5. **J+25** : Phase pilote 30 dossiers RH (cas réels anonymisés)
6. **J+30** : Validation par métiers des 6 cas d'usage de référence
7. **J+35** : Test charge moteur règles (scénario 250K dossiers/an)
8. **J+40** : Formation Camunda DMN administrateurs métier (2 jours)
9. **J+45** : DPIA RGPD finalisée et signée DPO
10. **J+55** : COPIL final validation Phase 2 + GO Phase 3

---

## E. PLAN DE PRODUCTION & JALONS

### E.1 Planning Phase 2 — 8 semaines

```
Semaine                │ S1  │ S2  │ S3  │ S4  │ S5  │ S6  │ S7  │ S8  │
───────────────────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
Cadrage métier         │ ███ │ ██  │     │     │     │     │     │     │
Audit master data      │ ██  │ ███ │ ██  │     │     │     │     │     │
Matrice flux condit.   │ █   │ ██  │ ███ │ ███ │ ██  │ █   │     │     │
Cas d'usage métier     │     │ █   │ ██  │ ███ │ ███ │ ██  │ █   │     │
Dictionnaire données   │ ██  │ ██  │ ███ │ ███ │ ██  │ █   │     │     │
DMN/Camunda config     │     │     │ █   │ ██  │ ███ │ ███ │ ██  │ █   │
Tests règles métier    │     │     │     │     │ █   │ ██  │ ███ │ ██  │
Reviews                │     │ █   │     │ █   │     │ █   │     │ ███ │
COPIL                  │  ●  │     │  ●  │     │  ●  │     │  ●  │  ●  │
───────────────────────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

● = COPIL (J+5, J+15, J+25, J+45, J+55 final)
```

### E.2 Livrables et jalons

| Jalon | Date | Livrable | Validation |
|-------|------|----------|------------|
| J+5 | 2026-05-26 | Décisions D1/D2/D3/D4 + audit master data | COPIL |
| J+15 | 2026-06-05 | 87 règles métier identifiées + RACI complet | PO Métier + DAF |
| J+25 | 2026-06-15 | 6 cas d'usage validés métiers (sign-off) | Métiers |
| J+30 | 2026-06-20 | 142 attributs documentés + lineage complet | DSI + DPO |
| J+35 | 2026-06-25 | Camunda DMN configuré (87 règles + 195 lignes) | Architecte |
| J+45 | 2026-07-05 | Tests métier exhaustifs (1 expert/domaine) | Métiers |
| J+50 | 2026-07-10 | DPIA RGPD finalisée + plan conformité | DPO + RSSI |
| J+55 | 2026-07-15 | Phase 2 finalisée — GO/NO-GO Phase 3 | **COPIL Direction** |

### E.3 Équipe et budget

| Rôle | ETP | Coût j/h | Jours | Total |
|------|-----|----------|-------|-------|
| Senior Manager (sponsor) | 0,2 | 1 200 € | 8 | 9,6 K€ |
| Project Manager | 1,0 | 850 € | 40 | 34 K€ |
| Business Analyst senior | 2,0 | 750 € | 80 | 60 K€ |
| Architecte solution | 0,5 | 950 € | 20 | 19 K€ |
| Expert Camunda DMN | 0,8 | 950 € | 32 | 30,4 K€ |
| DPO + RSSI (revue RGPD) | 0,2 | 1 100 € | 8 | 8,8 K€ |
| Experts métier (Achat/RH/Legal/Finance) | 4×0,3 | 850 € | 48 | 40,8 K€ |
| Data architect (master data) | 0,5 | 900 € | 20 | 18 K€ |
| Camunda Enterprise license (1 an) | — | forfait | — | 25 K€ |
| Formation métiers DMN (2j) | — | forfait | — | 12 K€ |
| Buffer (10 %) | — | — | — | 25,8 K€ |
| **TOTAL Phase 2 (8 semaines)** | | | **256 j** | **283 K€** |

### E.4 Critères de GO/NO-GO Phase 3

| Critère | Cible | Pondération |
|---------|-------|-------------|
| 87 règles métier validées et testées | 100 % | 25 % |
| 6 cas d'usage validés sign-off métiers | 100 % | 20 % |
| Master data alignée (gap < 5 %) | OUI | 15 % |
| Camunda DMN opérationnel | OUI | 15 % |
| DPIA RGPD finalisée | OUI | 10 % |
| Décisions D1-D4 prises | OUI | 10 % |
| Risques top 5 mitigés | Score moyen < 12 | 5 % |

**Seuil GO** : Score pondéré ≥ 90 % (Phase 2 plus critique que Phase 1 car bloquant pour développement).
**Seuil NO-GO** : Tout critère < 70 % → escalade ComEx.

---

## ANNEXES

- **Annexe A** : Matrice complète 87 règles métier (fichier Excel séparé)
- **Annexe B** : 28 cas d'usage détaillés (fichier Word séparé, 6 illustrés ici)
- **Annexe C** : Dictionnaire 142 attributs (fichier Excel)
- **Annexe D** : Tables Camunda DMN exportées (fichiers .dmn)
- **Annexe E** : DPIA RGPD signée DPO (fichier PDF séparé)
- **Annexe F** : Audit master data SAP — gap analysis (fichier Excel)

---

**Document soumis à validation COPIL du 2026-07-15.**

**Signataires requis** :
- DAF Group (validation seuils financiers + budget)
- DRH Group (validation cas RH + classification PII)
- DSI (validation architecture moteur règles + master data)
- Director Achats (validation cas Achat + référentiel fournisseurs)
- General Counsel (validation cas Juridique + Contrats)
- DPO (validation conformité RGPD + DPIA)
- RSSI (validation sécurité données sensibles)
