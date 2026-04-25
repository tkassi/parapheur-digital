# 📋 MODÉLISATION FONCTIONNELLE – PARAPHEUR DIGITAL AVEC SIGNATURE ÉLECTRONIQUE

**Document de référence pour spécifications fonctionnelles**  
*Version 1.0 - Avril 2026*

---

## 📑 TABLE DES MATIÈRES

1. [Vue d'ensemble](#1-vue-densemble)
2. [Processus métiers principaux](#2-processus-métiers-principaux)
3. [Workflow de validation à N niveaux](#3-workflow-de-validation-à-n-niveaux)
4. [Signature électronique](#4-signature-électronique)
5. [Gestion des exceptions](#5-gestion-des-exceptions)
6. [Traçabilité & Audit](#6-traçabilité--audit)
7. [Clôture & Archivage](#7-clôture--archivage)
8. [Intégration SI](#8-intégration-si)
9. [Rôles & Responsabilités (RACI)](#9-rôles--responsabilités-raci)
10. [Règles de gestion](#10-règles-de-gestion)
11. [Gestion des exceptions & cas limites](#11-gestion-des-exceptions--cas-limites)
12. [Indicateurs de performance (KPI)](#12-indicateurs-de-performance-kpi)
13. [Recommandations d'optimisation](#13-recommandations-doptimisation)

---

## 1. VUE D'ENSEMBLE

### 1.1 Périmètre fonctionnel

Le **parapheur digital** est une solution de dématérialisation complète permettant :
- ✅ Gestion de flux documentaires multi-niveaux
- ✅ Signature électronique conforme (France, Europe)
- ✅ Traçabilité intégrale et auditabilité
- ✅ Intégration ERP / Core métier
- ✅ Multi-entités, multi-filiales
- ✅ Performance et scalabilité

### 1.2 Acteurs principaux

| Rôle | Définition |
|------|-----------|
| **Initiateur** | Crée et dépose le dossier (documents + métadonnées) |
| **Validateur** | Examine et valide (visa / approche) |
| **Signataire** | Appose sa signature électronique |
| **Délégué** | Agit au nom d'un absorbé ou indisponible |
| **Administrateur** | Configure workflows, rôles, accès |
| **Auditeur** | Consulte traces et journaux (lecture seule) |

### 1.3 Entités métier clés

```
Dossier
├── Métadonnées (ref, date création, entité, statut)
├── Documents (pièces jointes, versions)
├── Circuit de validation (configuration dynamique)
├── Signatures (historique complet)
├── Audit trail (trace de chaque action)
└── Archivage (document final signé)
```

---

## 2. PROCESSUS MÉTIERS PRINCIPAUX

### 2.1 Processus P1 : Création et Dépôt de Dossier

**Acteur principal** : Initiateur  
**Durée moyenne** : 5-15 min

#### Description textuelle

L'**initiateur** accède à l'interface de création d'un nouveau dossier. Il remplit les **métadonnées obligatoires** (titre, description, entité, classification) et **charge les documents** à faire valider/signer.

Le système applique automatiquement les **règles de routage** pour générer le circuit de validation adapté à la classification et l'entité.

Un **bilan de conformité** est effectué avant acceptation du dépôt.

#### Flux détaillé

```
START
  ↓
1. Authentification utilisateur
  ↓
2. Accès à "Créer dossier"
  ↓
3. Remplir métadonnées
   ├─ Titre (obligatoire)
   ├─ Description (optionnelle)
   ├─ Classification (obligatoire) → Déclenche circuit auto
   ├─ Entité/Filiale (obligatoire)
   ├─ Bénéficiaires initiaux (optionnels)
   └─ Champ libres (si configuré)
  ↓
4. Upload documents
   ├─ Contrôle format (PDF, Word, Excel)
   ├─ Contrôle taille (max par fichier)
   ├─ Contrôle virus
   └─ Génération miniatures
  ↓
5. Sélection/confirmation circuit
   ├─ Affichage du circuit auto-généré
   ├─ Option modification manuelle (si autorisé)
   └─ Confirmation des signataires
  ↓
6. Vérification conformité
   ├─ Au moins un signataire obligatoire
   ├─ Pas de boucles infinies
   ├─ Tous les rôles requis présents
   └─ SLA réaliste
  ↓
7. Validation dépôt
  ↓
8. Création dossier (état : "EN ATTENTE")
  ↓
9. Notification validateur 1er niveau
  ↓
END
```

#### Données entrantes

| Champ | Type | Obligatoire | Contrôle |
|-------|------|-------------|----------|
| Titre | String(255) | ✓ | Non vide, < 255 car |
| Description | Text | ✗ | < 2000 car |
| Classification | Enum | ✓ | Parmi valeurs configurées |
| Entité | Select | ✓ | Selon RBAC user |
| Pièces jointes | File(s) | ✓ | Min 1, max 10, 50MB/total |
| Type de signature | Enum | ✓ | Simple/Avancée/Qualifiée |
| Urgence | Enum | ✗ | Normal / Urgent |
| Référence externe | String | ✗ | Format libre ou regex |

#### Résultat

- ✅ Dossier créé avec **ID unique** (UUID)
- ✅ État initial : `EN_ATTENTE_SIGNATURE`
- ✅ Circuit peuplé avec 1er validateur identifié
- ✅ Notifications envoyées
- ✅ Enregistrement audit

---

### 2.2 Processus P2 : Workflow de Validation à N Niveaux

**Acteur principal** : Validateurs / Signataires  
**Durée** : Variable selon SLA

#### Description textuelle générale

Chaque dossier suit un **circuit de validation** configurable, défini à la création ou modifié manuellement.

Pour chaque **étape du circuit** :

1. Le système **identifie les acteurs** pour ce niveau (rôle, délégation)
2. L'**acteur reçoit une notification** (email, dashboard)
3. L'**acteur examine** le dossier et décide :
   - ✅ **APPROUVER** → Passe au niveau suivant
   - ❌ **REJETER** → Retour à initiateur ou niveau précédent
   - ⏸️ **DÉLÉGUER** → Transfère responsabilité
   - ⚠️ **COMMENTAIRE** → Enrichit piste audit
4. L'**action est enregistrée** (audit trail complet)

Si **tous les niveaux sont validés** → Passage à **signature électronique**

#### Flux BPMN synthétique (Niveau k)

```
Dossier reçu au niveau k
        ↓
Identifier acteur pour rôle "R_k"
        ↓
Notifier acteur (email/dashboard)
        ↓
Acteur consulte dossier
        ↓
┌─────────────────────────────────────────────┐
│ Décision utilisateur                        │
└─────────────────────────────────────────────┘
     ↙           ↓            ↘          ↙
   VALIDE    REJETTE    DÉLÈGUE   COMMENTE
    (60%)      (15%)       (5%)       (20%)
    ↓           ↓            ↓          ↓
    │       RETOUR        TRANSFÈRE   ENRICH +
    │      À INIT.         À USER_X    VALIDE
    │       + NOTIF        + NOTIF      + NOTIF
    │           ↓            ↓          ↓
    └────────────┴────────────┴──────────┘
           ↓
    Enregistrement audit trail
           ↓
    k ← k+1 ?
    ↙          ↘
   NON (fin)    OUI (suite)
    ↓            ↓
  SIGNER      Suite niveau k+1
    ↓
   FIN FLUX VALIDATION
```

#### Règles de routage (conditionnement)

**Type A : Routage Séquentiel**
```
Niveau 1 (Manager) → Niveau 2 (Dir) → Niveau 3 (PDG)
Progression stricte, ordre fixe
```

**Type B : Routage Parallèle**
```
Niveaux 1A (Legal) ∥ 1B (RH) ∥ 1C (Finance)
Tous doivent valider avant passage niveau 2
```

**Type C : Routage Conditionnel**
```
if Montant < 1000€ then Niveau 1 (Manager)
else if Montant < 10000€ then Niveaux 1+2 (Manager+Dir)
else Niveaux 1+2+3 (Manager+Dir+PDG)
```

**Type D : Routage Mixte**
```
Niveau 1 (Manager) [SEQ]
  ↓
Niveaux 2A (Legal) ∥ 2B (Finance) [PAR]
  ↓
Niveau 3 (Dir Gé) [SEQ]
```

---

### 2.3 Processus P3 : Signature Électronique

**Acteur principal** : Signataire(s)  
**Durée** : 2-5 min/signature

#### Description textuelle

Une fois **tous les niveaux validés**, le dossier passe en phase de **signature électronique**.

Chaque **signataire** (selon le circuit) doit :
1. **Recevoir une notification**
2. **Consulter le document final** (synthèse validation + docs)
3. **Signer électroniquement** (simple/avancée/qualifiée)
4. **Confirmer** avec code OTP ou badge

La **signature est horodatée et scellée**. Le document devient **immuable**.

#### Flux détaillé

```
Dossier passe à "EN_SIGNATURE"
        ↓
Déterminer signataires (selon circuit)
        ↓
┌─ Boucle pour chaque signataire (mode séquentiel ou parallèle)
│  ↓
│  Envoyer notification signature
│  ↓
│  Signataire accède interface signature
│  ↓
│  Vérifier identité (OTP/Badge/Biométrique)
│  ↓
│  Signataire revoit document final + métadonnées
│  ↓
│  Signer (modes selon config)
│  ├─ Mode Simple : Click "Je signe"
│  ├─ Mode Avancé : Signature manuscrite sur tablette
│  └─ Mode Qualifié : Carte ID / Token / Service tiers
│  ↓
│  Générer certificat signature
│  ├─ Horodatage (serveur TSA)
│  ├─ Métadonnées signature (date, heure, IP, User-Agent)
│  └─ Scellement (hash document + sig)
│  ↓
│  Enregistrer signature dans dossier
│  ↓
│  Envoyer confirmation à signataire
│  ↓
└─ FIN si all(signataires validés)

Si mode PARALLÈLE : Attendre tous
Si mode SÉQUENTIEL : Passer au suivant

        ↓
Générer document final signé (PDF/XML)
        ↓
Sertifier sceau électronique global
        ↓
État dossier : "SIGNÉ"
        ↓
FIN signature
```

#### Types de signatures

| Type | Légalité | Infra | Cas d'usage |
|------|----------|-------|-----------|
| **Simple** (click) | Basique | Aucune | Visa interne, approbations |
| **Avancée** (cert+timestamp) | Légale (France) | PKI interne | Contrats, accord |
| **Qualifiée** (ID validator) | Légale (UE) | Service tiers certifié | Actes officiels, offre public |

#### Certificats et horodatage

```
Signature simple
├─ Hash document (SHA-256)
├─ Timestamp serveur
└─ Métadonnées (user, date, IP)

Signature avancée
├─ Certificat X.509 (PKI interne)
├─ Clé privée (stockée sécurisée)
├─ Horodatage TSA externe
├─ Signature + certificat encodés (CMS/PKCS#7)
└─ Preuves d'intégrité

Signature qualifiée
├─ Certificat réputé (Atos, Docusign, etc.)
├─ Infrastructure HSM (Hardware Security Module)
├─ Authentification forte (2FA)
├─ Conformité eIDAS
└─ Archivage à long terme (pérennisation)
```

---

### 2.4 Processus P4 : Clôture et Archivage

**Acteur principal** : Système (automatique)  
**Durée** : Immédiate + batch

#### Description textuelle

Une fois **toutes les signatures apposées**, le dossier entre en phase de **clôture et archivage**.

Le système :
1. **Génère le document final signé** (PDF/XML avec traces)
2. **Crée un sceau global** (prove intégrité + immuabilité)
3. **Enregistre en GED** (archivage légal)
4. **Index** le dossier (recherche, conformité)
5. **Accorde droits** d'accès en lecture seule
6. **Envoie notification** à tous les acteurs

#### Flux

```
Toutes signatures apposées
        ↓
Générer document final
├─ Fusion documents + métadonnées
├─ Intégration preuves signature
├─ Génération PDF/XML complet
└─ Horodatage global
        ↓
Créer sceau électronique
├─ Hash complet dossier
├─ Signature serveur autorité
├─ Certificat d'archivage
└─ Preuve pérennité (TimeStamp)
        ↓
Indexation pour recherche
├─ Métadonnées (classification)
├─ Contenu (OCR si image/scan)
├─ Mots-clés (tagging auto)
└─ Filiales/Entités
        ↓
Archivage GED
├─ Migration disque archivage
├─ Réplication (haute dispo)
├─ Conformité rétention légale
└─ Metadata storage
        ↓
Configuration accès
├─ Initiateur : lecture seule
├─ Signataires : lecture seule
├─ Auditeurs : lecture seule
├─ IT/Admin : backup
└─ Public (selon règles) : consultation
        ↓
Notifications finales
├─ Email à tous participants
├─ Journal audit (batch)
└─ Statistiques/KPI
        ↓
État dossier : "ARCHIVÉ"
        ↓
Déplacement zone froide (après délai légal)
```

#### Règles de rétention et conformité

| Critère | Durée | Fondement |
|---------|-------|----------|
| Dossier signé | 3 ans | Droit commercial (France) |
| Audit trail complet | 10 ans | RGPD + archivage légal |
| Versions intermédiaires | 1 an | Traçabilité |
| Métadonnées sensibles | Selon classification | Données personnelles |
| Dossiers rejetés | 6 mois | Archivage + retrouvaison |

---

## 3. WORKFLOW DE VALIDATION À N NIVEAUX

### 3.1 Architecture générale

```
Circuit dynamique
├─ N étapes (N = 1..10)
├─ Chaque étape : type (SEQ/PAR/COND)
├─ Rôle requis pour étape
├─ Délai SLA
├─ Actions autorisées (VALIDE / REJETTE / DÉLÈGUE)
└─ Règle transition vers étape suivante
```

### 3.2 Variantes par contexte métier

#### Variante V1 : Processus simple (achat < 5000€)

```
Étape 1: Manager (séquentiel)
   SLA: 48h
   Actions: Valide / Rejette
   ↓
Étape 2: Signataire (séquentiel)
   SLA: 24h
   Actions: Signe / Demande modif
   ↓
ARCHIVÉ
```

#### Variante V2 : Processus avancé (achat > 5000€)

```
Étape 1: Manager (séquentiel)
   SLA: 48h
   ↓
Étape 2A: Finance (parallèle) ∥ Étape 2B: Legal (parallèle)
   SLA: 72h (tous deux)
   ↓
Étape 3: Direction générale (séquentiel)
   SLA: 24h
   ↓
Signature Qualifiée (PDG)
   SLA: 24h
   ↓
ARCHIVÉ
```

#### Variante V3 : Processus conditionnel RH

```
IF Nature = Embauche THEN
   Niveau 1: Manager opérationnel → Validation contrat
   Niveau 2: RH → Conformité légale
   Niveau 3: Dir RH → Signature
ELSE IF Nature = Mutation THEN
   Niveau 1: Manager actuel → Accord départ
   Niveau 2: Manager futur → Accord arrivée
   Niveau 3: RH → Enregistrement
   Niveau 4: Dir RH → Signature
ELSE
   Niveau 1: Manager → Validation simple
   Niveau 2: RH → Enregistrement
```

#### Variante V4 : Processus multi-filiales (fusion)

```
Entité = Filiale A
   ↓
Étape 1: Dir Filiale A (Validation métier)
Étape 2: Finance Filiale A (Validation budget)
Étape 3: Dir Générale Groupe (Validation groupe)
   ↓
Signature Groupe
   ↓
Notification Filiale A + Groupe
```

### 3.3 Paramétrage du circuit

Chaque circuit est **défini par règle** au moment du dépôt :

```
RULE R1_ACHAT_SIMPLE
WHEN 
  Type_Document = "Bon_Achat" AND
  Montant < 5000 AND
  Entité IN ("FR", "DE", "BE")
THEN
  Étape 1: Role = "Manager", SLA = 48h, Type = SEQ
  Étape 2: Role = "Director", SLA = 24h, Type = SEQ
  Étape 3: Role = "Signataire", SLA = 24h, Type = SIGN
END

RULE R2_ACHAT_COMPLEX
WHEN 
  Type_Document = "Bon_Achat" AND
  Montant >= 5000 AND
  Montant < 50000
THEN
  Étape 1: Role = "Manager", SLA = 48h, Type = SEQ
  Étape 2A: Role = "Finance", SLA = 72h, Type = PAR
  Étape 2B: Role = "Legal", SLA = 72h, Type = PAR
  Étape 3: Role = "Director", SLA = 24h, Type = SEQ
  Étape 4: Role = "CEO_Signer", SLA = 24h, Type = SIGN
END
```

---

## 4. SIGNATURE ÉLECTRONIQUE

### 4.1 Modes de signature

#### Mode A : Signature simple (interface web)

```
Signataire se connecte
       ↓
Consultation document
       ↓
Click bouton "Je signe"
       ↓
Vérification OTP (SMS/Email) ← 2FA
       ↓
Signature enregistrée avec timestamp serveur
       ↓
Confirmation email
```

**Légalité** : Preuve écrite, mais signature "simple"  
**Certificat** : Aucun (signature au sens large)  
**Horodatage** : Serveur applicatif  
**Usage** : Visas, approbations internes

#### Mode B : Signature avancée (certificat PKI interne)

```
Signataire accède à interface avancée
       ↓
Authentification forte (LDAP + badge)
       ↓
Génération CSR (Certificate Signing Request)
       ↓
Signature avec certificat X.509 privé
       ├─ Hash SHA-256 document
       ├─ Signature asymétrique (RSA 2048 bit min)
       └─ Métadonnées (user, timestamp, IP)
       ↓
Appel TSA (TimeStamp Authority) externe
       ├─ Obtenir proof d'horodatage
       ├─ Scellement signature + timestamp
       └─ Certification audit trail
       ↓
Encodage signature (PKCS#7 / CMS)
       ↓
Stockage certificat + proof archive
       ↓
Notification confirmée
```

**Légalité** : Signature avancée (France/Europe)  
**Certificat** : PKI interne (validité 1-5 ans)  
**TSA** : Service externe certifié (ex: GlobalSign)  
**Usage** : Contrats, accords importants, audit compliance

#### Mode C : Signature qualifiée (service tiers certifié)

```
Signataire accède portail qualifié (DocuSign/Atos/Universign)
       ↓
Redirection vers service tiers
       ↓
Authentification forte (2FA, biométrique, ID)
       ├─ SMS + Mot de passe
       ├─ Badge professionnel + OTP
       └─ Reconnaissance faciale
       ↓
Consultation document via service tiers
       ↓
Signature electronique qualifiée
       ├─ Certificat réputé (HSM certifié eIDAS)
       ├─ Horodatage autorité reconnue
       ├─ Validation légale UE complète
       └─ Archivage tiers certifié
       ↓
Retour token signature au parapheur
       ↓
Intégration preuve signature à dossier
       ↓
Notification + archivage GED local
```

**Légalité** : Signature qualifiée (eIDAS - UE)  
**Certificat** : Tiers réputé certifié (ANSSI/eIDAS)  
**HSM** : Hardware Security Module (Thales/Securosys)  
**Usage** : Actes publics, offres financières, contrats valeur légale max

### 4.2 Gestion certificats et clés

#### Infrastructure PKI (recommandé)

```
Root CA (offline)
    ↓
Intermediate CA (racine opérationnelle)
    ├─ Certificat signature documents (valeur courte)
    ├─ Certificat timestamp (TSA)
    └─ Certificat archivage long-terme
       ↓
Clés privées
    ├─ Stockage HSM (Thales, Securosys)
    ├─ Sauvegarde redondante
    └─ Accès audit strict
```

#### Processus renouvellement certificat

| Phase | Durée avant expiration | Action |
|-------|------------------------|--------|
| Alert | 90 jours | Notification admin |
| Préparation | 60 jours | CSR généré, validation |
| Renouvellement | 30 jours | Cert signé, déploiement |
| Rotation | 15 jours avant exp | Tests, validation |
| Fin vie | 5 jours après exp | Archivage, audit |

### 4.3 Horodatage et immuabilité

#### Processus horodatage

```
Document signé en T0
    ↓
Hash SHA-256 généré
    ↓
Envoi à TSA (ex: RFC 3161 compliant)
    ↓
TSA réceptionne, horodate (T_TSA)
    ↓
TSA retourne TimeStamp Token
    ├─ Hash original
    ├─ Timestamp autorité
    ├─ Certificat TSA
    └─ Signature TSA
    ↓
Liaison signature + timestamp → Immuabilité
    ↓
Archivage preuve (audit trail)
```

#### Vérification intégrité

```
Document archivé (année 1)
    ↓
Récupération 5 ans plus tard
    ↓
Vérification signature
├─ Certificat signataire valide ?
├─ Hash document = hash signé ?
└─ Timestamp valide ?
    ↓
SI tous OK → Document valide légalement
SI NON → Escalade audit, notification
```

---

## 5. GESTION DES EXCEPTIONS

### 5.1 Cas d'exception (Taxonomy)

#### E1 : REJET avec demande de modification

```
Situation : Validateur rejette dossier (incomplet, erreur)
Trigger   : Click "Rejeter + commentaires"
Actions   : 
  1. Enregistrement motif rejet (catégorie + texte libre)
  2. Notification initiateur avec motif
  3. Dossier retourne état "EN_MODIFICATION"
  4. Initiateur peut corriger + relancer
  5. Retour au 1er niveau validation
Registre  : "Rejet_1er_niveau" → audit trail
```

#### E2 : DÉLÉGATION (absence utilisateur)

```
Situation : Validateur absent (congés, maladie, sabbatique)
Trigger   : Validateur pré-configure délégation
            OU Admin détecte absence > 3j
Actions   :
  1. Identifier délégué désigné
  2. Vérifier autorité de délégué (role >= rôle délégant)
  3. Transférer responsabilité
  4. Notifier délégué + stakeholders
  5. Mémoriser délégation (durée, motif)
Validité  : Délégation expire après :
            - Date fin configurée
            - Retour utilisateur
            - 30j max (sécurité)
Registre  : "Délégation_[User]_à_[Délégué]" → audit
```

#### E3 : ESCALADE automatique (dépassement SLA)

```
Situation : Validateur n'a pas réagi après SLA
Trigger   : Timer SLA + 48h buffer atteint
Actions   :
  1. Alerte email à validateur (rappel)
  2. CC manager du validateur
  3. Escalade vers N+1 hiérarchique
  4. Possibilité "Signature par délégué" activée
  5. Notification urgence à tous stakeholders
SLA :     Normal (48h) → Escalade (48h+48h) → Force signature
Registre  : "Escalade_[User]_Level_[N]" → audit
```

#### E4 : RETOUR À ÉTAPE PRÉCÉDENTE (circuit non-linéaire)

```
Situation : Validateur N+1 demande modification au niveau N
Trigger   : "Demander clarification au niveau précédent"
Actions   :
  1. Dossier retourne état "EN_RÉVISION_NIVEAU_N"
  2. Notification au validateur niveau N
  3. Métadonnées de "retour" enregistrées
  4. Validateur N peut modifier et relancer
  5. Reprise au niveau N (pas reset au début)
Limite    : Max 2 retours en arrière (sécurité infini loop)
Registre  : "Retour_Niveau_[N]_depuis_[N+1]" → audit
```

#### E5 : ANNULATION du processus

```
Situation : Initiative arrêtée, pas de suite
Trigger   : "Annuler" par initiateur ou admin
Conditions: 
  - Initiateur : avant 1ère validation
  - Admin : toujours (raison obligatoire)
  - Après signature : "Annulation + Contre-signature requise"
Actions   :
  1. État → "ANNULÉ"
  2. Notification tous participants (motif)
  3. Documents archivés zone "annulées"
  4. Historique complet conservé (audit légal)
  5. Pas de suppression physique
Registre  : "Annulation_par_[User]_motif_[Raison]"
```

#### E6 : VERSIONING (modification post-validation)

```
Situation : Besoin corriger/ajouter document après validation
Trigger   : "Créer version V2"
Conditions:
  - Avant signature (uniquement)
  - Validateurs re-notifiés
Actions   :
  1. V1 conservée (historique)
  2. V2 créée avec docs mises à jour
  3. Circuit reset ou continuité ?
     - Règle A: Full reset (prudent)
     - Règle B: Reprise depuis dernière validation (rapide)
  4. Notifications validateurs précédents
  5. Métadonnées versioning enregistrées
Registre  : "Versioning_V1→V2_[User]"
```

#### E7 : ERREUR SIGNATURE (mauvais signataire, annulation)

```
Situation : Signature apposée par erreur / révocation demandée
Trigger   : "Annuler signature" (avant archivage)
Conditions:
  - Avant archivage uniquement
  - Demandeur = signataire OU admin
  - Raison obligatoire
Actions   :
  1. Signature révoquée + marquée invalide
  2. Dossier retourne état "EN_SIGNATURE" (repeat)
  3. Notifications re-envoyées
  4. Historique complet préservé (traçabilité)
  5. Certificat révocation enregistré (PKI)
Limite    : Max 1 annulation par signataire (sécurité)
Registre  : "Révocation_Signature_[User]_motif_[Raison]"
```

#### E8 : DONNÉES SENSIBLES / REDACTION

```
Situation : Document contient PII à masquer avant archivage
Trigger   : "Masquer données" (avant signature finale)
Conditions:
  - Admin ou Manager uniquement
  - Raison obligatoire (RGPD, secret, confidentialité)
  - Audit trail conservé
Actions   :
  1. Zone redacté visuellement (bloc noir PDF)
  2. Métadonnées redaction enregistrées
  3. Version originale archivée sécurisée
  4. Version publique sans PII en GED
Registre  : "Redaction_[Zones]_par_[User]"
```

---

## 6. TRAÇABILITÉ & AUDIT

### 6.1 Audit Trail complet

Chaque action dans le parapheur est enregistrée avec :

```
{
  "timestamp": "2026-04-24T14:32:15.123Z",        // Horodatage UTC
  "user": {
    "id": "USR_001",
    "name": "Jean Dupont",
    "email": "jean.dupont@company.fr",
    "role": "Manager",
    "entity": "FR_001"
  },
  "action": "VALIDATION",                         // Type d'action
  "action_detail": {
    "type": "APPROVE",
    "step": 1,
    "level": "Manager"
  },
  "dossier": {
    "id": "DOS_2026_001",
    "title": "Achat équipement IT",
    "status_before": "EN_ATTENTE_VALIDATION",
    "status_after": "EN_ATTENTE_SIGNATURE",
    "version": 1
  },
  "metadata": {
    "ip_address": "192.168.1.100",
    "user_agent": "Mozilla/5.0...",
    "session_id": "SESS_xyz123",
    "device_id": "DEVICE_abc456"
  },
  "signature_proof": {
    "hash": "sha256:abc123...",
    "hmac": "hmac_sha256:def456..."
  },
  "comments": "Document conforme, approuvé pour suite",
  "duration": 3600,                              // Durée consultation (sec)
  "status": "SUCCESS"                             // SUCCESS / ERROR / ABORTED
}
```

### 6.2 Actions tracées par catégorie

| Action | Détail tracé | Responsable |
|--------|--------------|-------------|
| CRÉATION_DOSSIER | Métadonnées + docs | Initiateur |
| VALIDATION | Étape, décision, motif | Validateur |
| REJET | Motif catégorie + texte | Validateur |
| DÉLÉGATION | De/vers user, durée | Admin ou Delegant |
| MODIFICATION | Champs modifiés, avant/après | Initiateur |
| SIGNATURE | Certificat, horodatage, preuve | Signataire |
| ESCALADE | Niveau, trigger, vers qui | Système |
| CONSULTATION | User, durée, sections vues | Consultants |
| DOWNLOAD_EXPORT | Format, contenu, destination | User |
| ARCHIVAGE | Timestamp, zone, index | Système |
| DELETION / REDACTION | Raison, zones, audit | Admin |

### 6.3 Rapports audit

#### Rapport A1 : Traçabilité par dossier

```
Format : PDF/Excel
Contenu :
  - Timeline complète des actions
  - Acteurs impliqués + rôles
  - Décisions prise + motifs
  - Durées SLA vs réalité
  - Signataires et preuves
  - Anomalies détectées
Disponibilité : Téléchargeable par auditeur
Rétention : 10 ans
```

#### Rapport A2 : Audit de conformité

```
Format : PDF automatisé
Contenu :
  - Nombre dossiers par status (signé, annulé, rejeté)
  - Temps moyen traitement par étape
  - Taux d'escalade automatique
  - Retards SLA (liste + justification)
  - Certificats expirés / valides
  - Versioning statistics
Fréquence : Mensuel + ad-hoc
Destinataires : Dir IT, Compliance, Audit
```

#### Rapport A3 : Sécurité et détection anomalie

```
Format : Tableau de bord en temps réel
Alertes :
  ✓ Tentatives accès non-autorisé
  ✓ Signature par compte absent 30j
  ✓ Export massif documents
  ✓ Modification audit trail détectée
  ✓ Certificat révoqué encore utilisé
  ✓ Download sans consultation préalable
Seuils : Configurable par admin
Action : Notification auto à SOC
```

---

## 7. CLÔTURE & ARCHIVAGE

### 7.1 États et transitions

```
États machine :
├─ CRÉÉ (nouveau dossier)
├─ EN_ATTENTE_VALIDATION (circuit actif)
├─ EN_MODIFICATION (rejet, correction)
├─ EN_ESCALADE (SLA dépassé)
├─ EN_SIGNATURE (tous niveaux OK)
├─ SIGNÉ (toutes signatures OK)
├─ ARCHIVÉ (clôturé, immutable)
└─ ANNULÉ (arrêt processus)

Transitions valides :
CRÉÉ → EN_ATTENTE_VALIDATION
EN_ATTENTE_VALIDATION → EN_MODIFICATION (rejet)
EN_ATTENTE_VALIDATION → EN_SIGNATURE (validation complète)
EN_MODIFICATION → EN_ATTENTE_VALIDATION (relance)
EN_SIGNATURE → SIGNÉ (toutes sigs OK)
SIGNÉ → ARCHIVÉ (auto après délai 24h)
[*] → ANNULÉ (à tout moment, avec motif)
```

### 7.2 Document final signé

```
Fichier : Dossier_ID_FINAL_SIGNED.pdf
Contenu :
├─ Page 1 : Couverture (métadonnées, participants)
├─ Pages 2-N : Documents originaux + annotations
├─ Page N+1 : Résumé circuit validation (qui, quand, décision)
├─ Page N+2 : Bloc signature (nom, date, certificat)
├─ Page N+3 : Preuves numériques
│   ├─ Hash SHA-256 original
│   ├─ Certificats signataires
│   ├─ Timestamps (format RFC 3161)
│   ├─ Sceau électronique du groupe
│   └─ Chaîne de confiance complète
└─ Metadata PDF :
    ├─ Titre, auteur, création
    ├─ Signature numérique intégrée (XFA)
    └─ Rights restrictions (copy/edit prohibited)
```

### 7.3 Archivage GED

```
Destination : Système de gestion documentaire (ex: M-Files, Nuxeo)
Champs indexés :
  - Dossier ID (clé unique)
  - Titre, description
  - Classification
  - Entité / Filiale
  - Date création / signature
  - Signataires (full name)
  - Type document (Achat, RH, Contrat...)
  - Montant (si applicable)
  - Tags (OCR, custom, workflow)
  - Statut final (Signé, Annulé...)
  - Rétention jusqu'à (date légale)

Zones stockage :
  ├─ Zone chaude (1 an) : Accès rapide, réplication
  ├─ Zone tiède (2-3 ans) : Archivage tertiaire
  └─ Zone froide (4-10 ans) : Backup offline, conformité

Accès :
  ├─ Lecture : Initiateur, signataires, auditeurs
  ├─ Recherche : Tous les acteurs (permissions)
  ├─ Download : Audit trail complet
  ├─ Modification : INTERDITE (immuabilité)
  └─ Suppression : Admin only, avec raison légale
```

---

## 8. INTÉGRATION SI

### 8.1 Interactions ERP / Core métier

#### I1 : Création dossier déclenchée depuis ERP

```
Scénario : Achat approuvé dans SAP → création auto parapheur

Flux ERP :
  SAP Purchase Order créé (PO_123)
    ↓ [Event: PO.Created]
  Middleware ETL reçoit notification
    ↓
  Vérification règles (montant, entité, type...)
    ↓
  SI montant > 5000€ THEN
    Create Parapheur via API_PARAPHEUR.createDossier
      {
        external_id: "SAP:PO_123",
        title: "Achat Équipement IT - 8500€",
        classification: "Achat_Stratégique",
        entity: "FR_001",
        beneficiaries: ["mgr_fr@company.fr"],
        documents: [
          {
            url: "http://sap.internal/po/123/pdf",
            name: "PO_123.pdf"
          }
        ]
      }
    ↓
  Parapheur retourne ID: "DOS_2026_001"
    ↓
  Middleware enregistre mapping SAP:Parapheur
    ↓
  SAP notifié : "PO_123 → En circuit validation parapheur"
    ↓
  Statut SAP : "PENDING_APPROVAL"
```

#### I2 : Signature complète → Retour à ERP

```
Parapheur : Dossier "DOS_2026_001" signés avec preuves
    ↓
  Système déclenche webhook SAP
    {
      event: "dossier.signé",
      dossier_id: "DOS_2026_001",
      external_id: "SAP:PO_123",
      signataires: ["PDG_Signature"],
      timestamp: "2026-04-24T16:00:00Z",
      proof_url: "https://parapheur/api/proof/DOS_2026_001"
    }
    ↓
  SAP reçoit webhook, vérifie signature
    ↓
  SAP met à jour PO_123
    Status: "APPROVED" (PENDING → APPROVED)
    Digital_Signature_Proof: "DOS_2026_001"
    ↓
  SAP déclenche flux suivant (Bon Commande automatique)
    ↓
  Notification acheteur : "PO approuvé, suite en cours"
```

#### I3 : Synchronisation statuts temps réel

```
État Parapheur          → État ERP
EN_ATTENTE_VALIDATION   → "PENDING_APPROVAL"
EN_ESCALADE             → "PENDING_APPROVAL_ESCALATED"
EN_SIGNATURE            → "PENDING_SIGNATURE"
SIGNÉ                   → "APPROVED"
ANNULÉ                  → "CANCELLED"
ARCHIVÉ                 → "CLOSED"

API polling (ex: toutes les 5 min) :
  GET /api/parapheur/status/SAP:PO_123
  → Retourne état + métadonnées + preuves

Webhook événements (push time) :
  POST /eai/webhook/parapheur-status
  ← Notifie ERP immédiatement
```

### 8.2 Authentification et autorisation (SSO)

```
Mécanisme : SAML 2.0 ou OAuth2 (OpenID Connect)

Flow SAML :
  Utilisateur accède /parapheur/login
    ↓
  Redirection vers Okta (IdP)
    ↓
  Okta authentifie (Active Directory / LDAP)
    ↓
  Retour Assertion SAML signé
    ↓
  Parapheur vérifie signature, crée session
    ↓
  Extraction attributs (user.id, user.email, user.roles)
    ↓
  Mapping roles SAML → Roles Parapheur
    SAML Role "IT_Manager" → Parapheur Role "Manager"
    SAML Role "CEO" → Parapheur Role "Signataire_Qualifié"
    ↓
  Création JWT local (valide 8h, refresh token 30j)
    ↓
  Redirection page d'accueil parapheur
```

### 8.3 Notifications et alertes

#### N1 : Email

```
Modèle : HTML responsive (compatible mobile)
Contenu :
  - Objet descriptif (ex: "[URGENT] Document en attente votre signature")
  - Corps : Résumé dossier (titre, montant, participants)
  - Call-to-action (bouton "Consulter")
  - Deadline SLA (ex: "À faire avant demain 17h")
  - Signature mail (pied de page légal)

Envoi :
  SMTP via service d'envoi centralisé
  CC/BCC selon rules (manager, audit)
  Retry auto si échec (3x avec délai)
  Limitation : Max 1 rappel par jour si non-traité
```

#### N2 : Dashboard interne

```
Composant widget :
  - Dossiers en attente (nb + liste)
  - SLA alertes (rouge si < 24h)
  - Escalades automatiques (en cours)
  - Archivés (consultation rapide)
  
Refresh : Real-time via WebSocket
Notification badge : Red dot si tâche urgente
```

#### N3 : API webhooks (intégrations tiers)

```
Exemple : Intégration Slack
  Webhook URL: https://hooks.slack.com/services/xxx/yyy/zzz
  
  Événement : dossier.escalade
    ↓
  POST webhook avec payload :
  {
    "text": "🚨 Escalade : Achat IT (DOS_2026_001) en attente 3j",
    "attachments": [{
      "color": "danger",
      "fields": [
        {"title": "Dossier", "value": "DOS_2026_001"},
        {"title": "Responsable", "value": "@jean.dupont"},
        {"title": "Action", "value": "Consulter"}
      ]
    }]
  }
    ↓
  Message posté Slack channel #approvals
```

### 8.4 API REST parapheur

```
Endpoints clés :

POST /api/v1/dossiers
  ↓ Créer nouveau dossier
  Body: {title, classification, entity, documents[], type_signature}
  Response: {dossier_id, circuit, status}

GET /api/v1/dossiers/{id}
  ↓ Consulter dossier détail
  Response: {métadonnées, documents, circuit, signataires, audit_trail}

POST /api/v1/dossiers/{id}/valider
  ↓ Valider (rôle: validateur)
  Body: {décision: "APPROVE"|"REJECT", commentaires, step}
  Response: {status_updated, next_step, notifications_sent}

POST /api/v1/dossiers/{id}/signer
  ↓ Apposer signature
  Body: {type_signature, certificat, code_otp}
  Response: {signature_proof, horodatage, proof_url}

GET /api/v1/dossiers/{id}/audit
  ↓ Télécharger audit trail complet
  Response: {audit_trail[], export_format: "JSON|CSV"}

GET /api/v1/dossiers/{id}/proof
  ↓ Vérifier intégrité signature
  Response: {signature_valid: true|false, chain_of_trust}
```

---

## 9. RÔLES & RESPONSABILITÉS (RACI)

### Matrice RACI complète

```
Activité                     | Init | Manager | Dir | Finance | Legal | CEO | Admin | Audit
─────────────────────────────┼──────┼─────────┼─────┼─────────┼───────┼─────┼──────┼──────
Créer dossier               |  R   |    C    |     |    C    |   C   |     |      |
Valider (1er niveau)        |  I   |    R    |  C  |         |       |     |      |
Valider (2e niveau)         |  I   |         |  R  |    R    |   R   |     |      |
Signature (C-Level)         |  I   |    C    |     |         |       |  R  |      |
Approuver circuit custom    |      |         |     |         |       |     |  R   |
Modifier après rejet        |  R   |    C    |     |         |       |     |      |
Déléguer responsabilité     |      |    R    |  R  |         |       |     |      |
Escalade SLA                |  I   |    I    |  R  |         |       |     |  R   |
Archivage / GED             |      |         |     |         |       |     |  R   |  C
Audit trail consultation    |  I   |    C    |  C  |         |       |     |      |  R
Suppression / Redaction     |      |         |     |         |       |     |  R   |  C
Gestion certificats         |      |         |     |         |       |     |  R   |
Rapport conformité          |      |         |     |         |       |     |  R   |  R

R = Responsible (exécute)
A = Accountable (décisionnel final)
C = Consulted (avis requis)
I = Informed (notification)
```

### Détail responsabilités par rôle

#### Initiateur
- ✓ Créer dossier, charger documents
- ✓ Modifier avant 1ère validation
- ✓ Corriger après rejet
- ✓ Consulter statut temps réel
- ✗ Valider (autre acteur)
- ✗ Modifier après signature

#### Manager/Validateur 1er niveau
- ✓ Consulter dossier assigné
- ✓ Valider / rejeter avec motif
- ✓ Demander clarification initiateur
- ✓ Déléguer (absence)
- ✓ Consulter historique actions
- ✗ Signer document
- ✗ Archiver

#### Direction / Validateur N-niveau
- ✓ Idem manager + expertise métier
- ✓ Approuver circuits custom
- ✓ Escalader si nécessaire
- ✓ Superviser validateurs avals
- ✗ Créer dossier
- ✗ Archiver

#### Signataire (CEO / PDG)
- ✓ Consulter dossier prêt signature
- ✓ Apposer signature qualifiée
- ✓ Vérifier preuves signature
- ✓ Consulter audit trail (rôle)
- ✗ Valider intermédiaires
- ✗ Archiver

#### Administrateur
- ✓ Configurer workflows et circuits
- ✓ Gérer rôles et accès (RBAC)
- ✓ Escalader situation bloquée
- ✓ Archiver / GED
- ✓ Gérer certificats et TSA
- ✓ Auditer compliance
- ✓ Supporter utilisateurs
- ✗ Signer au nom d'autre
- ✗ Modifier audit trail post-archivage

#### Auditeur
- ✓ Consulter tous dossiers (audit)
- ✓ Télécharger audit trails
- ✓ Générer rapports conformité
- ✓ Vérifier signatures / certificats
- ✓ Consulter GED archivés
- ✗ Modifier dossiers
- ✗ Signer
- ✗ Accéder documents avant archivage (sauf accord)

---

## 10. RÈGLES DE GESTION

### RG001 : Sélection automatique du circuit

```
IF Type_Document = "Achat"
THEN
  IF Montant < 5000€ THEN
    Circuit = [Manager → Director → Signataire]
  ELSE IF Montant < 50000€ THEN
    Circuit = [Manager → Director → Finance → CEO]
  ELSE
    Circuit = [Manager → Director → Finance → Legal → CEO → Board]
  END
ELSE IF Type_Document = "RH" AND Nature = "Embauche"
THEN
  Circuit = [Manager → RH → Dir_RH → CEO]
ELSE IF Type_Document = "Contrat"
THEN
  Circuit = [Manager → Legal → Finance → CEO]
END
```

### RG002 : SLA (Service Level Agreement)

```
Niveau Manager       : 48h (business days)
Niveau Director      : 24h
Niveau Finance       : 72h
Signature C-Level    : 24h
Escalade automatique : +48h après SLA + notif

IF Urgence = "URGENT" THEN
  Multiplier SLA par 0.5 (moitié)
  Ajouter notification prioritaire (SMS + email)
END
```

### RG003 : Validation parallèle vs séquentielle

```
Définition circuit au moment dépôt :
  
Règle par défaut : Séquentiel
  Étape 1 → Étape 2 → Étape 3 (une par une)
  
Exceptions parallèles :
  IF Étapes = ["Finance", "Legal"] THEN
    Finance ∥ Legal (simultané, BOTH must OK)
  END
  
  IF Étapes = ["Dept_A_Mgr", "Dept_B_Mgr"] AND Multi_Entité THEN
    Dept_A ∥ Dept_B (valider indépendamment)
  END

Combinaison mixte :
  [Manager] → [Finance ∥ Legal] → [Director]
```

### RG004 : Escalade automatique en cas dépassement SLA

```
IF Dossier.DateDépôt + SLA + 48h < NOW THEN
  Status → "EN_ESCALADE"
  Actions :
    1. Notifier validateur (SMS + email "URGENT")
    2. Notifier manager validateur (CC email)
    3. Optionnel : Transférer délégué si configuré
    4. Dashboard alert rouge
    5. Historique : [ESCALADE_AUTO_LEVEL_1_2026_04_24]
END

Limite escalade : Max 1 fois par niveau (prévenir boucles)
```

### RG005 : Délégation de pouvoir

```
Conditions :
  IF User.Delegation_Configurée THEN
    Délégué peut actionner à la place
  ELSE IF Absence_Détectée (> 3 jours) THEN
    Admin peut forcer délégation
  END

Validité :
  Duration ≤ 30 jours (max sécurité)
  OR Date_Retour_Programmée atteinte
  → Délégation annulée automatiquement

Autorité Délégué :
  Délégué.Role >= Délégant.Role (sinon rejet)
  Exemple : Manager → peut déléguer à Director ✓
            Manager → ne peut PAS déléguer à Intern ✗

Traçabilité :
  Chaque action du délégué enregistre "[VIA DÉLÉGATION de User X]"
```

### RG006 : Rejet et retour modification

```
IF Validateur.Décision = "REJETTER" THEN
  1. Récupérer motif rejet (catégorie + texte libre)
  2. Dossier revient état "EN_MODIFICATION"
  3. Notifier Initiateur + Signature + raison
  4. Initiateur peut corriger + relancer
  5. Au relancement : Relancer depuis niveau rejet (non reset)
  6. Historique : [REJET_NIVEAU_1_MOTIF_INCOMPLET]
END

Limite rejets : Max 2 rejets par dossier (après 2 rejets → escalade admin)
```

### RG007 : Versioning document

```
IF Initiateur.Action = "Créer_Version" THEN
  1. Version actuelle → archivée (EN_VERSION_ANTÉRIEURE)
  2. Nouvelle version créée (V2)
  3. Circuit choix :
     Règle A) Full Reset : restart depuis début (sûr)
     Règle B) Continue : reprise depuis dernière validation (rapide)
  4. Validateurs re-notifiés
  5. Historique : [VERSION_V1_V2_PAR_USER_2026_04_24]
END

Limite : Max 3 versions par dossier (après 3 → escalade)
```

### RG008 : Horodatage et immuabilité

```
Signature (tous types) reçoit :
  1. Timestamp serveur (heure création)
  2. Appel TSA externe (horodatage tiers conforme RFC 3161)
  3. Scellement (hash doc + signature + timestamp)
  4. Vérification immuabilité : SHA256(doc_archivé) = SHA256_signé

Vérification post-archivage :
  IF Hash_Document_Retrievé != Hash_Signature THEN
    État → "INTÉGRITÉ_COMPROMISE"
    Alerter audit + admin (anomalie critique)
  ELSE
    État → "INTÉGRITÉ_VÉRIFIÉE"
  END
```

### RG009 : Multi-entités / Filiales

```
Chaque dossier attaché à une entité (FR_001, DE_002, etc.)

Règles routage multi-entité :
  IF Montant_Total > 100K€ AND Acteurs_Multi_Pays THEN
    Ajouter validation "Group Finance" (niveau N)
  END

  IF Entité_Cible ≠ Entité_Initiateur THEN
    Notifier Direction Entité cible (info)
  END

Archivage par entité :
  Dossier indexé et accessible au périmètre entité
  Auditeur group : accès tous dossiers toutes entités
  Manager entité : dossiers entité propre + group-level
```

### RG010 : Sécurité et conformité signature

```
Signature Simple :
  ✓ OTP (code unique SMS/email)
  ✓ Authentification LDAP
  ✓ No certificat (preuve contrat seulement)

Signature Avancée :
  ✓ Certificat X.509 PKI interne
  ✓ TSA external (horodatage tierce)
  ✓ Encodage PKCS#7
  ✓ Légal France/Europe
  
Signature Qualifiée :
  ✓ Certificat réputé (eIDAS)
  ✓ HSM fortement sécurisé
  ✓ 2FA (SMS + biométrique)
  ✓ Légal valeur maximale (UE)

Conformité RGPD :
  Dossiers contenant PII → encryption au repos
  Droit à l'oubli : Anonymisation après 30j (optionnel)
  Audit trail : 10 ans (obligation légale)
```

---

## 11. GESTION DES EXCEPTIONS & CAS LIMITES

### Tableau synthétique des exceptions

| Exception | Trigger | Impact | Escalade | Traçabilité |
|-----------|---------|--------|----------|------------|
| **E1 - Rejet** | Validateur rejette | Retour à modif | 2 rejets max | `REJET_LEVEL_N` |
| **E2 - Délégation** | User absent | Transfer pouvoir | <30j auto-expire | `DELEGATION_[User]` |
| **E3 - Escalade SLA** | SLA+48h dépassé | Urgent + transfer | Admin intervient | `ESCALADE_LEVEL_N` |
| **E4 - Retour arrière** | Demand clarif N+1 | Repeat level N | <2x max | `RETOUR_LEVEL_N` |
| **E5 - Annulation** | Stop processus | Archivage annulé | Admin raison | `ANNULATION_[Raison]` |
| **E6 - Versioning** | Doc correction pré-sig | Repeat circuit | <3 versions | `VERSION_V[N]` |
| **E7 - Révocation sig** | Cancel signature | Repeat signature | <1x par user | `REVOCATION_[User]` |
| **E8 - Redaction** | PII masquage | Dual archive | Admin raison | `REDACTION_[Zones]` |
| **E9 - Certificat expir** | Cert expiré | Bloc signature | Force renouveau | `CERT_EXPIRED` |
| **E10 - Doc non-conforme** | Format/size fail | Rejet upload | Retry user | `UPLOAD_FAIL` |

---

## 12. INDICATEURS DE PERFORMANCE (KPI)

### KPI Métier

#### KM1 : Taux de dossiers signés

```
Formule : (Dossiers_Signés / Dossiers_Créés) × 100
Cible : > 98%
Alerte : < 95%
Mesure : Mensuel + trendline

Signification : Mesure "complétude" processus
Anomalies : % rejetés élevé ? Escalades fréquentes ?
```

#### KM2 : Délai moyen de traitement (TAT)

```
Formule : Avg(DateSignature - DateDépôt)
Cible : < 5 jours business
Par niveau : 
  - Niveau 1 (Manager) : < 48h
  - Niveau 2 (Director) : < 24h
  - Signature : < 24h

Benchmark : Comparer par type doc et entité
```

#### KM3 : Taux de rejet

```
Formule : (Dossiers_Rejetés / Dossiers_Présentés) × 100
Cible : 5-10%
Alerte : > 15%

Par motif :
  - Incomplet : 40%
  - Erreur données : 35%
  - Non-conforme règle : 25%

Action : Si taux élevé → revoir process ou formation
```

#### KM4 : Taux d'escalade automatique

```
Formule : (Dossiers_Escaladés / Total_Dossiers) × 100
Cible : < 5%
Alerte : > 10%

Analyse :
  Par rôle : Manager escalade 20% → manque capacité ?
  Par type doc : Achats escaladent 12% → SLA trop court ?
```

#### KM5 : Taux de délégation

```
Formule : (Actions_Déléguées / Total_Actions) × 100
Cible : 5-15%
Alerte : > 25%

Insight : % élevé → manque ressources ou couches management ?
```

### KPI Opérationnel

#### KO1 : Disponibilité système

```
Formule : (Uptime / 24h×30j) × 100
Cible : > 99.8%
SLA : 99.5% (120 min downtime/mois toléré)

Inclut :
  - Disponibilité interface web
  - API response time < 2s (p95)
  - Signatures fonctionnelles
```

#### KO2 : Performance signature

```
Métrique : Temps moyen signature
Cible : < 5 sec (simple)
       < 15 sec (avancée + TSA)
       < 30 sec (qualifiée + HSM)

Alerte : Si > 2× cible → investigation (bottleneck TSA/HSM)
```

#### KO3 : Intégrité audit trail

```
Métrique : % actions tracées vs attendues
Cible : 100%

Contrôle : Monthly audit
  Vérifier : Hash intégrité audit logs
  Vérifier : Pas d'actions orphelines
  Vérifier : Timestamps cohérents
```

#### KO4 : Capacité stockage

```
Mesure : Volume total dossiers + documents
Croissance : +[X]% par mois

Gestion :
  Zone chaude : monitorer occupation
  Archivage : automatiser après 1 an
  Alertes : >80% capacity → action
```

### KPI Sécurité

#### KS1 : Incidents sécurité

```
Suivi : Nombre tentatives accès non-autorisé par mois
Cible : 0
Alerte : > 5 par mois → investigation

Types incidents :
  - Tentative accès dossier non-autorisé
  - Signature par compte suspecte
  - Export massif documents
  - Modification audit trail détectée
```

#### KS2 : Certificats valides

```
Formule : (Certificats_Valides / Total_Certs) × 100
Cible : 100%
Alerte : Any expired cert → escalade urgent

Dashboard :
  - Certificats expir. < 30j → liste
  - Certificats expir. < 7j → alert rouge
  - Certificats expir. → bloc imméd.
```

#### KS3 : Conformité signatures

```
Suivi : % signatures valides vs signées
Cible : 100%

Vérification :
  Chaîne certificat OK ? → Oui
  Hash intégrité OK ? → Oui
  Timestamp valide ? → Oui
  Signature pas révoquée ? → Oui

Anomalie : % < 99.9% → escalade audit + IT
```

---

## 13. RECOMMANDATIONS D'OPTIMISATION

### O1 : Automatisation workflows

**Situation actuelle** : Circuits générés manuellement pour chaque dossier  
**Problème** : Risque erreur, manque flexibilité, temps création

**Recommandation** :
```
✓ Implémenter moteur règles (ex: Drools, ARGO Workflows)
✓ Maintenance règles par métier (non IT)
✓ Versioning / rollback règles (audit)
✓ A/B testing nouveaux circuits

Impact attendu :
  - Temps création -40%
  - Erreurs circuit -80%
  - Flexibilité +500%
```

### O2 : Signature biométrique / WebAuthn

**Situation actuelle** : OTP SMS (2FA basique)  
**Problème** : SMS interception possible, UX mediocre mobile

**Recommandation** :
```
✓ Implémenter WebAuthn (FIDO2) optionnel
  → Face ID, Touch ID, USB keys
✓ Fallback SMS + email (legacy)
✓ Dashboard sécurité utilisateur

Impact attendu :
  - UX +50% satisfaction
  - Sécurité phishing -90%
  - Adoption mobile +300%
```

### O3 : OCR et extraction données documents

**Situation actuelle** : Documents chargés manuellement (titre, montant)  
**Problème** : Erreurs saisie, lenteur, pas d'indexation contenu

**Recommandation** :
```
✓ OCR intelligent documents (ex: AWS Textract, Google Vision)
✓ Extraction champs clés (montant, dates, parties)
✓ Alimentation auto métadonnées
✓ Indexation contenu pour recherche (GED)

Impact attendu :
  - Métadonnées +90% complétude
  - Recherche +400% efficacité
  - Temps préparation -30%
```

### O4 : IA pour classification et routage

**Situation actuelle** : Règles statiques basées montant/type  
**Problème** : Inflexibilité, pas adaptation risque/contexte

**Recommandation** :
```
✓ ML model (scikit-learn, XGBoost) entraîné sur historique
✓ Prédire niveau d'escalade (risque document)
✓ Recommander circuit optimal (+ confiance score)
✓ Feedback loop : validation utilisateur → re-train

Features :
  - Montant, entité, type doc
  - Historique validateur (% rejet)
  - Jours semaine (urgence contextuelle)
  - Patterns anomalie

Impact attendu :
  - Circuits optimisés -20% durée moyenne
  - Escalades anticipées +60%
  - Compliance automatique +40%
```

### O5 : Blockchain / Immutabilité distribuée

**Situation actuelle** : Audit trail centralisé (vulnérable si DB compromise)  
**Problème** : Preuve immuabilité dépend système central

**Recommandation** :
```
✓ Hash chaque action dans blockchain (privée / consortium)
✓ Horodatage distribué irrévocable
✓ Preuve légale "couche 1" + audit trail legacy "couche 2"

Technologie :
  Hyperledger Fabric (privé, performant)
  OU Corda (juridique, smart contracts)

Cas d'usage :
  - Signature qualifiée : +preuve blockchain
  - Audit trail : immuable à 100%
  - Conformité réglementaire maximale

Impact attendu :
  - Preuve légale inattaquable
  - Conformité régulateur +100%
  - Audit coûts -40% (auto-proof)
```

### O6 : Dashboard analytics en temps réel

**Situation actuelle** : Rapports mensuels, pas visualisation live  
**Problème** : Pas réactivité, données stales

**Recommandation** :
```
✓ Dashboard Power BI / Looker (real-time)
  - Dossiers en attente par manager (alertes)
  - SLA violations (détection auto)
  - Taux signatures par jour/entité
  - Anomalies sécurité (heatmap)

✓ Alertes configurables (Slack, email, SMS)
✓ Export ad-hoc (compliance, audit)

Impact attendu :
  - Réactivité SLA +500%
  - Visibilité manage +400%
  - Insights métier +300%
```

### O7 : API ecosystem et intégrations

**Situation actuelle** : Couplage fort ERP (SAP) et parapheur  
**Problème** : Difficulté évolution, autres systèmes isolés

**Recommandation** :
```
✓ API-First architecture
  - REST + GraphQL (query flexible)
  - OpenAPI/Swagger documentation
  - SDKs client (Python, Java, .NET)
  - Rate limiting, versioning

✓ Webhooks événementiels (dossier.signé, etc.)
✓ Event bus (Kafka) pour intégrations async
✓ iPaaS integration (Zapier, Make, SAP Integration Suite)

Impact attendu :
  - Intégrations 5× plus rapides
  - Maintenance -60%
  - Flexibilité métier +unlimited
```

### O8 : Mobile app dédié (offline-first)

**Situation actuelle** : Interface web responsive (pas idéal mobile)  
**Problème** : Signature sur téléphone lent, pas offline, UX desktop

**Recommandation** :
```
✓ App mobile native (iOS / Android)
  Fonctionnalités :
  - Voir dossiers assignés (push notification)
  - Signer via biométrique (Face ID, Touch ID)
  - Offline consultation (sync auto Wi-Fi)
  - Signature manuscrite (tablette compatible)

Technology : React Native (1 codebase) ou Flutter

Impact attendu :
  - Adoption +300% (sur-terrain)
  - Temps validation -50% (proximité)
  - UX satisfaction +80%
```

---

## Résumé Exécutif

Ce document fournit une **modélisation exhaustive et exploitable** des processus d'un parapheur digital, couvrant :

✅ **Processus métiers** : Création, validation N-niveaux, signature, archivage  
✅ **Variantes** : Simple, avancé, conditionnel, multi-entité  
✅ **Gestion exceptions** : Rejet, délégation, escalade, versioning  
✅ **Traçabilité audit** : 100% des actions enregistrées + preuves  
✅ **Signatures électroniques** : Simple, avancée, qualifiée (conforme légal)  
✅ **Intégrations** : ERP (SAP), SSO, webhooks, API REST  
✅ **RACI** : Rôles et responsabilités clairement définis  
✅ **Règles de gestion** : 10 règles clés + SLA  
✅ **KPI** : Métier, opérationnel, sécurité  
✅ **Optimisations** : IA, blockchain, OCR, mobile, analytics  

**Prêt pour cadrage projet et spécifications fonctionnelles détaillées.**

