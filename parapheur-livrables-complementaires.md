# 📋 LIVRABLES COMPLÉMENTAIRES – PARAPHEUR DIGITAL

**Matrices RACI détaillées, exceptions complètes, templates implémentation**

---

## TABLE DES MATIÈRES

1. [Matrice RACI Détaillée](#matrice-raci-détaillée)
2. [Catalogue Exceptions Complet](#catalogue-exceptions-complet)
3. [Template Règles de Gestion](#template-règles-de-gestion)
4. [Checklist Implémentation](#checklist-implémentation)
5. [Schéma Données Dossier](#schéma-données-dossier)
6. [Glossaire et Acronymes](#glossaire-et-acronymes)

---

## MATRICE RACI DÉTAILLÉE

### Format détaillé avec descriptions

```
┌─────────────────────────────────────────────────────────────────┐
│ RESPONSABILITÉS ÉTENDUES – PAR PROCESSUS ET PHASE               │
└─────────────────────────────────────────────────────────────────┘

PHASE 1 : CRÉATION & DÉPÔT
──────────────────────────

Activité                      | Initia | Manager | Dir | CFO | Legal | CIO | Audit
──────────────────────────────┼────────┼─────────┼─────┼─────┼───────┼─────┼──────
Créer métadonnées dossier     |  R,A   |         |     |     |       |     |
Charger documents             |  R,A   |         |     |     |       |     |
Sélectionner classification   |  R     |  C      |     |     |       |     |
Vérifier conformité uploads   |        |         |     |     |       | R   | C
Générer circuit auto          |        |         |     |     |       | R   |
Valider dépôt (approuver)     |        |         |     |     |       | R   |
Notifier validateurs          |        |         |     |     |       | R   |


PHASE 2 : VALIDATION À N NIVEAUX
─────────────────────────────────

Activité                      | Initia | Manager | Dir | CFO | Legal | CIO | Audit
──────────────────────────────┼────────┼─────────┼─────┼─────┼───────┼─────┼──────
Consulter dossier             |        |  R      | C   |     |       |     |
Analyser document             |        |  R      | C   |     | C     |     |
Valider (signature)           |        |  R      |     |     |       |     |
Approuver (direction)         |        |  C      | R   | C   |       |     |
Contrôler montant             |        |         |     | R   | C     |     |
Valider conformité légale     |        |  C      |     |     | R     |     |
Rejeter + motif               |        |  R      | R   |     |       |     |
Demander clarification        |        |  R      | R   |     |       |     |
Déléguer responsabilité       |        |  R      | R   |     |       |     |
Escalader SLA                 |        |  I      | R   |     |       | I   |
Enregistrer audit             |        |         |     |     |       | R   | C


PHASE 3 : SIGNATURE ÉLECTRONIQUE
─────────────────────────────────

Activité                      | Initia | Manager | Dir | CFO | Legal | CIO | Audit
──────────────────────────────┼────────┼─────────┼─────┼─────┼───────┼─────┼──────
Recevoir notification         |        |         |     |     |       |     |
Consulter document            |        |         | R   |     |       |     |
Apposer signature (simple)    |        |         | R   |     |       |     |
Apposer signature (avancée)   |        |         | R   |     |       |     |
Apposer signature (qualifiée) |        |         | R   |     |       |     |
Vérifier certificat valide    |        |         |     |     |       | R   | C
Générer TSA proof             |        |         |     |     |       | R   |
Enregistrer preuve signature  |        |         |     |     |       | R   | C


PHASE 4 : CLÔTURE & ARCHIVAGE
──────────────────────────────

Activité                      | Initia | Manager | Dir | CFO | Legal | CIO | Audit
──────────────────────────────┼────────┼─────────┼─────┼─────┼───────┼─────┼──────
Générer document final        |        |         |     |     |       | R   |
Créer sceau électronique      |        |         |     |     |       | R   |
Indexation et OCR             |        |         |     |     |       | R   |
Archivage GED                 |        |         |     |     |       | R   | C
Configurer accès              |        |         |     |     |       | R   |
Envoyer notifications finales |        |         |     |     |       | R   |
Migration zone froide         |        |         |     |     |       | R   | C


GESTION & ADMINISTRATION
────────────────────────

Activité                      | Initia | Manager | Dir | CFO | Legal | CIO | Audit
──────────────────────────────┼────────┼─────────┼─────┼─────┼───────┼─────┼──────
Configurer workflows          |        |         |     |     |       | R   |
Gérer rôles et accès (RBAC)   |        |         |     |     |       | R   |
Approuver circuits custom     |        |  C      | R   |     |       |     |
Gérer certificats             |        |         |     |     |       | R   |
Gérer délégations forcées     |        |         |     |     |       | R   | C
Escalader situation bloquée   |        |         | C   |     |       | R   | I
Supporter utilisateurs        |        |  I      |     |     |       | R   |
Générer rapports compliance   |        |         |     |     |       | R   | R
Audit conformité              |        |         |     |     |       | C   | R
Gérer incidents sécurité      |        |         |     |     |       | R   | R


SÉCURITÉ & CONFORMITÉ
─────────────────────

Activité                      | Initia | Manager | Dir | CFO | Legal | CIO | Audit
──────────────────────────────┼────────┼─────────┼─────┼─────┼───────┼─────┼──────
Authentification users (SSO)  |        |         |     |     |       | R   |
Authentification forte (2FA)  |        |         |     |     |       | R   |
Signature biométrique         |        |         |     |     |       | R   |
Conformité RGPD               |        |  C      | C   |     | R     | R   | C
Conformité eIDAS              |        |         |     |     | R     | R   | C
Traçabilité audit (10 ans)    |        |         |     |     | C     | R   | R
Redaction PII                 |        |         | C   |     | C     | R   | C
Rétention légale              |        |         |     |     | R     | R   | I
Rotation certificats          |        |         |     |     |       | R   |
Détection anomalies           |        |         |     |     |       | R   | C


LÉGENDE:
─────
R = Responsible (exécute tâche)
A = Accountable (approbation finale, responsable ultim)
C = Consulted (avis requis, input obligatoire)
I = Informed (notification seule, pas input)
```

### Responsabilités par Rôle (Détail)

#### INITIATEUR
```
✓ CRÉE et GÈRE dossier avant signature
✓ Charge documents
✓ Saisit métadonnées
✓ Sélectionne/confirme circuit validation
✓ Modifie document après rejet
✓ Relance validation après correction
✓ Consulte statut temps réel
✓ Consulte audit trail (rôle)
✓ Télécharge document signé archivé
✓ Reçoit notifications

✗ Valide (sauf rôle double)
✗ Signe
✗ Accède zone froide
✗ Administre système
✗ Voit autres dossiers (sauf share)

DÉLAI ACTION: 5 jours business (correction rejet)
SLA ASSOCIÉ: 10 jours complet (dépôt à archivage)
```

#### MANAGER (1er validateur)
```
✓ Consulte dossier assigné
✓ Valide/rejette avec motif
✓ Demande clarification initiateur
✓ Délègue (absence proactive)
✓ Consulte historique actions
✓ Décide routage conditionnel (si config)
✓ Supervise escalade SLA
✓ Commente enrichissement audit

✗ Signe (sauf rôle dual)
✗ Archive
✗ Gère certificats
✗ Accède données sensibles brutes

DÉLAI ACTION: 48h (SLA standard)
DÉCISIONS: 60% approuver, 15% rejetter, 5% déléguer, 20% enrichir
```

#### DIRECTEUR / VALIDATEUR INTERMÉDIAIRE
```
✓ Idem Manager + expertise métier supérieure
✓ Approuve circuits custom (règles exception)
✓ Valide montants/budgets complexes
✓ Supervise validateurs avals (team)
✓ Escalade décisions budgétaires
✓ Consulte toute documentation support
✓ Participe COPIL conformité

✗ Créer dossier
✗ Archiver
✗ Gérer infrastructures

DÉLAI ACTION: 24h (SLA court)
ESCALADE: Si Manager ne décide pas après SLA+48h
```

#### SIGNATAIRE (C-Level / CEO / PDG)
```
✓ Consulte dossier prêt signature (résumé validations)
✓ Appose signature qualifiée
✓ Vérifier preuves signature (optionnel)
✓ Consulte audit trail (rôle)
✓ Délègue signature (absence)
✓ Reçoit notification chaque signature demandée

✗ Valide niveaux intermédiaires
✗ Rejette document (escalade à director)
✗ Archive
✗ Modifie après signature

DÉLAI ACTION: 24h (SLA critique)
FRÉQUENCE MOYENNE: 10-20 signatures/jour selon volumétrie
CERTIFICAT REQUIS: Qualifié (eIDAS) ou avancé PKI
```

#### ADMINISTRATEUR IT / OPERATIONS
```
✓ Configure tous workflows et circuits
✓ Gère rôles (RBAC) et accès utilisateurs
✓ Gère certificats (renouvellement, révocation)
✓ Gère délégations forcées (absence système détectée)
✓ Escalade manuelle situation bloquée
✓ Support utilisateurs (reset password, débug)
✓ Monitoring système (uptime, performance)
✓ Backup et disaster recovery
✓ Gestion GED et archivage
✓ Reporting technique
✓ Gestion incidents sécurité

✗ Valider au nom de quelqu'un
✗ Signer au nom de quelqu'un
✗ Modifier audit trail post-archivage
✗ Accéder contenu sensible sans raison opérationelle

PERMISSIONS SPÉCIALES: SSH, DB admin, certificat root
TRAÇABILITÉ: Toute action admin enregistrée (super-audit)
ROTATION: Minimum 2 admins (contrainte sécurité)
```

#### AUDITEUR (Compliance / Internal Audit)
```
✓ Consulter tous dossiers (lecture seule)
✓ Télécharger audit trails complets
✓ Générer rapports conformité
✓ Vérifier signatures et certificats valides
✓ Consulter GED archivés
✓ Exporter données anonymisées
✓ Participer COPIL conformité

✗ Modifier documents
✗ Signer
✗ Supprimer enregistrements
✗ Accéder avant archivage (sauf accord)
✗ Consulter dossiers sensibles (RH, confidentiels)

ACCÈS: 
  - Tous dossiers "ARCHIVÉ" : OK
  - Dossiers "EN_ATTENTE" : Cas par cas (accord)
  - Données RH/Santé : Accès conditionnel + anonymisation

RAPPORTS RÉGULIERS: Mensuel, trimestriel, annuel
```

---

## CATALOGUE EXCEPTIONS COMPLET

### Format : Exception Card

```
┌──────────────────────────────────────────────────────────┐
│          EXCEPTION CARD – Modèle réutilisable            │
├──────────────────────────────────────────────────────────┤
│
│ EXCEPTION ID : E[N]_[CODE]
│ Titre          : [Court titre]
│ Catégorie      : [Processus/Signature/Admin/Sécurité]
│ Sévérité       : [LOW / MEDIUM / HIGH / CRITICAL]
│ Probabilité    : [Rare / Occasionnel / Fréquent]
│
│ DÉCLENCHEUR (TRIGGER)
│ ─────────────────────
│ Condition       : [Condition qui déclenche exception]
│ Détection       : [Automatique / Manuel / Événement]
│
│ CONTEXTE FONCTIONNEL
│ ────────────────────
│ Qui            : [Acteurs impliqués]
│ Quand          : [Phase du processus]
│ Pourquoi       : [Impact métier]
│
│ ACTIONS CORRECTIVES
│ ───────────────────
│ Immédiate       : [Action système/user immédiate]
│ Court terme    : [Actions dans minutes/heures]
│ Long terme     : [Actions 24-48h]
│
│ IMPACT
│ ──────
│ Utilisateur    : [UX impact]
│ Métier         : [Délai, blocage]
│ Sécurité       : [Confidentialité, intégrité]
│ SLA            : [% impact SLA]
│
│ ESCALADE
│ ────────
│ Niveau 1       : [Auto-résolution attendue]
│ Niveau 2       : [Si non-résolu après X temps]
│ Niveau 3       : [Si escalade niveau 2 échoue]
│
│ TRAÇABILITÉ
│ ───────────
│ Audit entry    : [Enregistrement audit]
│ Notification   : [Qui doit être notifié]
│ Reporting      : [Métrique suivie]
│
│ CAS D'USAGE / SCÉNARIO
│ ──────────────────────
│ Exemple 1      : [Situation réelle + résolution]
│ Exemple 2      : [Autre situation]
│
└──────────────────────────────────────────────────────────┘
```

### EXCEPTION E1 : REJET – Demande Correction

```
EXCEPTION ID    : E1_REJET_CORRECTION
Titre           : Validateur rejette dossier, correction requise
Catégorie       : Processus / Validation
Sévérité        : MEDIUM
Probabilité     : Fréquent (15-20% des dossiers)

DÉCLENCHEUR
────────────
Condition       : Validateur clique "Rejeter" + saisit motif
Détection       : Manuel (click user)

CONTEXTE
─────────
Qui             : Validateur k, Initiateur
Quand           : Phase 2 (Validation)
Pourquoi        : Document incomplet / erreur données / non-conforme

ACTIONS
────────
Immédiate       : 
  1. Récupérer motif + catégorie rejet
  2. Enregistrer audit: {action: REJECT, level: k, motif}
  3. État dossier → EN_MODIFICATION
  4. Dossier → "gris" (inactive) dashboard

Court terme (< 1h) :
  1. Email à initiateur: "Dossier rejeté - Raison: [motif]"
  2. Lien consultation document rejeté
  3. Deadline correction: 5 business days
  4. Dashboard initiateur: "EN ATTENTE VOTRE ACTION"

Long terme (24-48h) :
  1. Si pas correction: Notification rappel
  2. CC Manager initiateur (escalade douceur)

IMPACT
───────
Utilisateur     : Initiateur doit corriger + relancer (ajout 5-7j)
Métier          : Dossier "stalled", respect délai à risque
Sécurité        : NONE
SLA             : +7j delai (impact 20-30% cas réel)

ESCALADE
──────────
Niveau 1        : Initiateur corrige + relance (self-service)
Niveau 2        : Manager initiateur notifié après 3 jours non-action
Niveau 3        : Admin force relance (après 5 jours limite)

TRAÇABILITÉ
─────────────
Audit           : REJET_LEVEL_[k]_[DATE]_[User]
Notification    : Email initiateur + CC manager
Reporting       : KPI: % rejets, motif stats

SCÉNARIO RÉEL
──────────────
1. Initiateur crée dossier "Achat IT 8500€"
2. Manager valide étape 1 → Passe direction
3. Direction examines → "Montant erroné, doit être 8200€"
4. Direction rejette: [ERREUR_DONNÉES - "Prix incorrect"]
5. Dossier retourne initiateur
6. Initiateur corrige montant + relance
7. Circuit reprend depuis étape direction (pas reset début)
8. Direction re-valide → OK
9. Suite signature
```

### EXCEPTION E2 : DÉLÉGATION – Absence User

```
EXCEPTION ID    : E2_DELEGATION_ABSENCE
Titre           : Délégation pouvoir (absence user)
Catégorie       : Admin / RH
Sévérité        : MEDIUM
Probabilité     : Occasionnel (3-5% des acteurs)

DÉCLENCHEUR
────────────
Condition A     : User pre-configure délégation
Condition B     : Admin détecte absence > 3 jours
Détection       : Manuel (config) + Automatique (batch)

CONTEXTE
─────────
Qui             : User validateur, Délégué, Admin, Initiateur
Quand          : Phase 2 (Validation en cours)
Pourquoi        : Congé, maladie, sabbatique, emergency

ACTIONS
────────
SCÉNARIO A (Proactif - User configure)
───────────────────────────────────────
Immédiate (Configuration) :
  1. User accès Settings → "Configurer délégation"
  2. Saisit : DATE_DÉBUT, DATE_FIN, Délégué_ID, Motif
  3. Vérifie : Délégué.role >= User.role (sécurité)
  4. SAUVEGARDER
  5. Notification auto délégué envoie

Short term (immédiat) :
  1. Délégué reçoit email: "Vous êtes délégué de User_X"
  2. Dashboard Délégué: Badge "Délégué de User_X"
  3. Tous dossiers User_X → affectés Délégué

SCÉNARIO B (Réactif - Admin détecte)
───────────────────────────────────────
Immédiate (Batch automatique) :
  1. Système vérifie : Last_login < NOW - 3 jours
  2. OUI → ALERTE admin: "User XX absent 3j"
  3. Admin peut forcer délégation

Short term :
  1. Admin interface : "Forcer délégation"
  2. Admin saisit : Délégué_ID, motif
  3. CONFIRMER
  4. Notifications envoyées User + Délégué

IMPACT
───────
Utilisateur     : Délégué prend en charge tâches
Métier          : Continuité assurée (pas blocage)
Sécurité        : Rôle délégué doit être >= (protection)
SLA             : Aucun impact (delegation = continuité)

ESCALADE
──────────
Niveau 1        : Délégation active, continues tâches
Niveau 2        : Si délégué aussi absent → Escalade N+2
Niveau 3        : Admin force signature après SLA+48h

TRAÇABILITÉ
─────────────
Audit           : DELEGATION_[User]_to_[Delegate]_[Period]
Actions         : Toute action délégué enregistre "[VIA DELÉG de User]"
Notification    : Email tous 3 (User, Délégué, Manager)
Reporting       : % délégations, fréquence

LIMITE SÉCURITÉ
────────────────
Max 30 jours    : Si > 30j, prolongation requiert approbation
Auto-expire     : À date_fin configurée
Révocation      : User peut révoquer avant date_fin

SCÉNARIO RÉEL
──────────────
1. Manager_A va en congé 3 semaines (2026-05-01 à 2026-05-21)
2. Pré-configure délégation vers Manager_B (même rôle)
3. Manager_B reçoit notification
4. Dossier arrive Manager_A le 2026-05-05
5. Système reroute vers Manager_B (délégation active)
6. Manager_B traite dossier "VIA DELEGATION Manager_A"
7. Manager_A revient 2026-05-21
8. Délégation auto-expire
9. Dossiers futurs → Manager_A directement
```

### EXCEPTION E3 : ESCALADE SLA – Timeout Auto

```
EXCEPTION ID    : E3_ESCALADE_SLA_AUTO
Titre           : Escalade automatique SLA dépassé
Catégorie       : Admin / Performance
Sévérité        : HIGH
Probabilité     : Occasionnel (5-10% des dossiers)

DÉCLENCHEUR
────────────
Condition       : DateDépôt + SLA + 48h buffer < NOW
Détection       : Automatique (batch horaire)

CONTEXTE
─────────
Qui             : Validateur non-réactif, Manager, Admin
Quand           : Phase 2 (Validation)
Pourquoi        : Validateur oublie, surchargé, absent

ACTIONS
────────
Timeline :

T0 + SLA (ex: 48h)    :
  1. Email reminder validateur: "Document en attente"
  2. Ton courtois, délai respecté

T0 + SLA + 24h       :
  1. Email + SMS urgence
  2. CC: Manager validateur

T0 + SLA + 48h       : [ESCALADE ACTIVE]
  1. État dossier → EN_ESCALADE
  2. Dashboard : Dossier rouge (URGENT)
  3. Notifications:
     - Manager validateur (SMS + EMAIL)
     - Admin notification
     - Initiateur info
  4. Audit entry: ESCALADE_LEVEL_[k]_[DATE]

  5. Options escalade:
     a) Déléguer à manager (manager signe à la place)
     b) Transférer à N+1 hiérarchique
     c) Auto-delegation configurée → apply

T0 + SLA + 96h (48h+48h) : [FORCE SIGNATURE]
  1. Si pas résolution : force signature délégué
  2. Ou escalade admin manuel
  3. Notifications tous stakeholders
  4. Dossier progresse (pas stuck)

IMPACT
───────
Utilisateur     : Validateur reçoit urgence alerts
Métier          : Dossier "débloqué" par escalade
Sécurité        : NONE (signature légitime)
SLA             : Dossier continue → respect délai global

ESCALADE
──────────
Niveau 1        : Auto-escalade sys + notifications
Niveau 2        : Manager intervient / force delegation
Niveau 3        : Admin override (rattrapage manuel)

TRAÇABILITÉ
─────────────
Audit           : ESCALADE_LEVEL_[k]_SLA_EXCEEDED
Timestamp       : Heure escalade exact
Notif log       : SMS/Email envoyés, réception confirmée
Reporting       : KPI escalade %, par validateur

CONFIGURATION
──────────────
SLA par rôle (configurable) :
  - Manager : 48h
  - Director : 24h
  - CEO : 24h
  - Signature : 24h

Buffer : +48h avant force
Urgence : Passer SLA × 0.5 si urgence dossier

SCÉNARIO RÉEL
──────────────
1. Dossier créé 2026-04-20 10h
2. Arrive Manager (SLA: 48h)
3. Manager oublie / absent
4. T0 + 48h (2026-04-22 10h) : Reminder email
5. Manager ignore
6. T0 + 72h (2026-04-23 10h) : Email + SMS urgence, CC manager
7. Toujours rien
8. T0 + 96h (2026-04-24 10h) : [ESCALADE ACTIVE]
   - État → EN_ESCALADE
   - Dossier rouge dashboard
   - Manager reçoit SMS "URGENT: Escalade dossier DOS_2026_001"
   - Admin notifié
9. Manager accepte délégation à Senior Manager
10. Senior Manager approuve le jour même
11. Dossier continue normal flow
```

### EXCEPTION E4 : RETOUR ARRIÈRE – Clarification Requise

```
EXCEPTION ID    : E4_RETOUR_ARRIERE
Titre           : Demande clarification à niveau précédent
Catégorie       : Processus
Sévérité        : LOW
Probabilité     : Occasionnel (3-5%)

DÉCLENCHEUR
────────────
Condition       : Validateur N+1 demande clarif à validateur N
Détection       : Manuel (click user)

CONTEXTE
─────────
Qui             : Validateur N+1, Validateur N, Initiateur
Quand          : Phase 2 (Validation progression)
Pourquoi        : Point non-clair, données manquantes, ambiguïté

ACTIONS
────────
Immédiate :
  1. Validateur N+1 clique "Demander clarification"
  2. Saisir message texte (min 20 char)
  3. Dossier → EN_REVISION_NIVEAU_N
  4. Validateur N reçoit notification

Short term (< 1h) :
  1. Validateur N consulte clarification demandée
  2. Répond directement (via commentaire)
  3. Dossier → EN_ATTENTE_VALIDATION (retour N+1)
  4. Validateur N+1 reçoit réponse notification

Long term :
  1. Validateur N+1 peut:
     - VALIDER (si clarification suffisante)
     - REJETTER (si réponse non-satisfait)
     - Ou DEMANDER à nouveau clarif

LIMITE
───────
Max retours arrière : 2 (prévenir ping-pong infini)
Après 2 retours : Escalade admin pour décision manuelle

IMPACT
───────
Utilisateur     : Communication indirect (moins efficace)
Métier          : Délai +1-2 jours
Sécurité        : NONE
SLA             : Pause SLA pendant clarif (ne compte pas)

TRAÇABILITÉ
─────────────
Audit           : RETOUR_LEVEL_[N]_FROM_[N+1]_[DATE]
Message         : Clarif demandée + réponse enregistrées
Notification    : Email + dashboard

SCÉNARIO RÉEL
──────────────
1. Manager approuve dossier (étape 1)
2. Director examine (étape 2)
3. Director: "Montant total pas clair, somme parties != total"
4. Director clique "Demander clarification"
5. Manager notifié: "Clarif demandée: Vérifier somme montants"
6. Manager corrige = 8500€ (pas 8200€+300€ séparé)
7. Manager répond: "Montant consolidé 8500€ vérifié"
8. Director reçoit réponse
9. Director approve → Validation OK
```

### EXCEPTION E5 : ANNULATION – Arrêt Processus

```
EXCEPTION ID    : E5_ANNULATION_PROCESS
Titre           : Annulation complète du processus
Catégorie       : Processus / Admin
Sévérité        : MEDIUM
Probabilité     : Rare (1-2%)

DÉCLENCHEUR
────────────
Condition A     : Initiateur annule avant 1ère validation
Condition B     : Admin annule (raison obligatoire)
Condition C     : Post-signature annulation (requiert contre-sig)
Détection       : Manuel (click user)

CONTEXTE
─────────
Qui             : Initiateur (si avant validation), Admin (toujours)
Quand           : Avant signature complète
Pourquoi        : Initiative annulée, pas de suite, erreur

ACTIONS
────────
CAS A (Initiateur avant validation) :
  1. Initiateur clique "Annuler dossier"
  2. Confirmer : "Êtes-vous sûr ? Non-réversible"
  3. Annuler → État: ANNULÉ
  4. Notifications : Validateurs, initiateur
  5. Archive : Dossier → zone "annulés"
  6. Historique : Complètement conservé

CAS B (Admin post-validation) :
  1. Admin accès dossier (même état avancé)
  2. Clique "Annuler dossier"
  3. Saisir motif (obligatoire) :
     - "Initiative arrêtée"
     - "Changement processus"
     - "Non-applicabilité"
     - "Erreur processus"
     - "Autre : [texte libre]"
  4. État → ANNULÉ
  5. Notifications : Tous stakeholders + motif
  6. Archive zone "annulés administratif"

CAS C (Post-signature annulation) :
  1. Très rare, seulement admin
  2. Dossier déjà signé = immutable
  3. Solution : Créer document "ANNULATION ET CONTRE-SIGNATURE"
  4. Nouvelle signature requise (PDG)
  5. Dossier original marqué "ANNULÉ_VIA_CONTRE_SIG"
  6. Document annulation signé archivé ensemble

IMPACT
───────
Utilisateur     : Aucun travail perdu (archive complet)
Métier          : Initiative suspendue / terminée
Sécurité        : Pas de suppression (traçabilité légale)
SLA             : N/A (annulé)

ESCALADE
──────────
N/A             : Décision finale (pas escalade)

TRAÇABILITÉ
─────────────
Audit           : ANNULATION_[User]_MOTIF_[Motif]
Timestamp       : Exact moment annulation
Raison          : Enregistrée complètement
Archive         : Historique dossier conservé 10 ans

SCÉNARIO RÉEL
──────────────
1. Initiateur crée dossier "Achat équipement IT 20K€"
2. Envoie validation
3. Le jour suivant : Initiative changée (achat report)
4. Initiateur clique "Annuler"
5. Confirmation : "Êtes-vous sûr ?"
6. Oui → ANNULÉ
7. Validateurs notifiés : "Dossier annulé par initiateur"
8. Archive zone "annulés"
9. Dossier peut être récupéré ultérieurement (consultation)
```

### EXCEPTION E6-E10 (RÉSUMÉ)

```
E6 : VERSIONING – Modification doc pré-signature
─────────────────────────────────────────────────
Trigger     : Initiateur corrige document avant signature
Action      : V1 archivée, V2 créée, circuit reset ou continue
Limite      : Max 3 versions (après 3 → escalade)
Impact      : +3-5 jours délai
Audit       : VERSION_[V1→V2]_[User]_[Date]


E7 : RÉVOCATION SIGNATURE – Annuler signature
───────────────────────────────────────────────
Trigger     : Signataire ou admin annule signature
Action      : Signature marquée invalide, dossier → EN_SIGNATURE
Limite      : Max 1 annulation par signataire
Impact      : +1 jour, requête re-signature
Audit       : REVOCATION_SIG_[User]_[Motif]
Condition   : Avant archivage uniquement


E8 : REDACTION PII – Masquer données sensibles
────────────────────────────────────────────────
Trigger     : Admin détecte PII avant archivage
Action      : Zones redactées visuellement (PDF noir)
Versioning  : Version originale archivée sécurisée
              Verison publique sans PII en GED
Impact      : Conformité RGPD
Audit       : REDACTION_[Zones]_[User]_[Raison]
Limite      : Avant archivage uniquement


E9 : CERTIFICAT EXPIRÉ – Blocage Signature
─────────────────────────────────────────────
Trigger     : Certificat signataire expiré
Action      : Signature bloquée, notification admin
Escalade    : Admin force renouvellement
Impact      : Blocage immédiat phase signature
Audit       : CERT_EXPIRED_[Signataire]_[Date]
Prevention  : Alert admin 90j, 60j, 30j avant expir


E10 : DOCUMENT NON-CONFORME – Erreur Upload
──────────────────────────────────────────────
Trigger     : Format/size/virus check failed
Action      : Rejet dépôt, message error utilisateur
Impact      : Retry upload, aucun dossier créé
Audit       : UPLOAD_FAIL_[Raison]_[User]_[File]
Types rejet : Format (Word→PDF), Size > 50MB, Virus detected
```

---

## TEMPLATE – RÈGLES DE GESTION

### Format Standard

```sql
-- ============================================================================
-- RULE ID: RG_[NUMBER]_[SHORT_CODE]
-- Description: [Une ligne descriptive]
-- Category: [Routing/Timing/Security/Approval]
-- Author: [Team]
-- Version: 1.0
-- Last Updated: 2026-04-24
-- ============================================================================

RULE RG_001_AUTO_CIRCUIT_SELECTION
  WHEN
    Type_Document = "Achat"
    AND Classification IN ("Strategic", "Procurement", "Capex")
  THEN
    SELECT_CIRCUIT_DYNAMICALLY(
      IF Montant < 5000 THEN
        Circuit = "SIMPLE" : [Manager(48h) → Director(24h) → Signataire(24h)]
      ELSE IF Montant < 50000 THEN
        Circuit = "STANDARD" : [
          Manager(48h) → 
          [Finance(72h) ∥ Legal(72h)] → 
          Director(24h) → 
          Signataire_Qualifié(24h)
        ]
      ELSE
        Circuit = "COMPLEX" : [
          Manager(48h) → 
          [Finance(72h) ∥ Legal(72h) ∥ Compliance(96h)] → 
          Director(24h) → 
          CFO_Approve(24h) → 
          CEO_Signature_Qualifié(24h)
      END
    )
  END
  AUDIT_ENTRY : "CIRCUIT_SELECTED_[Type]_[Amount]_[Circuit]"
  NOTIFICATION : [All participants with circuit]


RULE RG_002_SLA_ENFORCEMENT
  WHEN
    Document arrives at validation level k
  THEN
    SET_TIMER(SLA_k) {
      0h         : Silent waiting
      SLA_k      : Notif reminder (email)
      SLA_k + 24h: Notif urgent (email + SMS)
      SLA_k + 48h: ESCALADE_AUTOMATIC
                   - Status → EN_ESCALADE
                   - Notify manager
                   - Activate delegation options
      SLA_k + 96h: Force signature OR escalate admin
    }
  END
  
  EXCEPTION:
    IF Urgence = "URGENT" THEN
      Multiply_All_SLA_by(0.5) [halve timers]
    END
  
  AUDIT_ENTRY : "SLA_[Level]_[Status]_[Timestamp]"


RULE RG_003_PARALLEL_VALIDATION
  WHEN
    Circuit contains [Finance, Legal] at same step
  THEN
    EXECUTE_PARALLEL {
      Finance_Validator   : Check amounts, budget
      Legal_Validator     : Check contract, compliance
      
      Condition_Progress  : ALL validators must APPROVE before next step
      Timeout             : Individual SLA per validator (72h)
      Failure             : If one rejects → return to previous step
    }
  END
  
  AUDIT_ENTRY : "PARALLEL_VALIDATION_[Finance_Status]_[Legal_Status]"


RULE RG_004_CONDITIONAL_ROUTING
  WHEN
    Type_Document = "RH" 
    AND Nature IN ("Embauche", "Mutation", "Licenciement")
  THEN
    SELECT_CIRCUIT_BY_NATURE {
      CASE "Embauche":
        Circuit = [Manager → RH → Dir_RH → CEO]
      CASE "Mutation":
        Circuit = [Manager_From → RH → Manager_To → Dir_RH → CEO]
      CASE "Licenciement":
        Circuit = [Manager → RH → Legal → Dir_RH → CEO]
    }
  END
  
  AUDIT_ENTRY : "RH_CIRCUIT_[Nature]_SELECTED"


RULE RG_005_DELEGATION_AUTHORIZATION
  WHEN
    User.Has_Delegation = true
    AND Delegation.Valid_Until > NOW
    AND Delegation.Status = "ACTIVE"
  THEN
    VERIFY_DELEGATEE_AUTHORITY {
      IF Delegatee.Role >= User.Role THEN
        ALLOW_DELEGATION
        Notify (Delegatee, User, Manager)
        Log_Action : "[VIA_DELEGATION_from_User]"
      ELSE
        REJECT_DELEGATION (role mismatch)
      END
    }
  END
  
  EXPIRATION:
    IF NOW > Delegation.Valid_Until OR User.Return_Date reached THEN
      Auto_Disable_Delegation
    END
  
  MAXIMUM_DURATION: 30 days


RULE RG_006_REJECTION_LIMIT
  WHEN
    Validator.Decision = "REJECT"
  THEN
    INCREMENT(Dossier.Rejection_Count)
    Log_Rejection {
      rejection_category : [Enum]
      rejection_text     : [Free text]
      timestamp          : [ISO 8601]
    }
    
    IF Dossier.Rejection_Count >= 3 THEN
      Status → "ESCALADE_MANUAL"
      Notify_Admin (Alert Red)
      Dossier.Requires_Manual_Review = true
    ELSE
      Status → "EN_MODIFICATION"
      Notify_Initiator (Email with detail)
    END
  END
  
  AUDIT_ENTRY : "REJECTION_[Count]_[Category]_[Level]"


RULE RG_007_SIGNATURE_TYPE_DETERMINATION
  WHEN
    Dossier ready for signature
  THEN
    SELECT_SIGNATURE_TYPE_BY_ROLE {
      IF Role IN ["Manager", "Validateur"] THEN
        Signature_Type = "SIMPLE" (OTP, no cert)
      ELSE IF Role IN ["Director", "CFO"] THEN
        Signature_Type = "AVANCÉE" (Cert PKI + TSA)
      ELSE IF Role IN ["CEO", "PDG", "Board"] THEN
        Signature_Type = "QUALIFIÉE" (eIDAS compliant)
    }
  END
  
  VERIFICATION:
    Verify_Certificate_Valid {
      IF Cert.Valid_Until < NOW THEN
        BLOCK_SIGNATURE
        Notify_Admin : "Certificate Expired"
      END
    }
  END
  
  AUDIT_ENTRY : "SIGNATURE_TYPE_[Type]_[Signataire]"


RULE RG_008_AUDIT_TRAIL_IMMUTABILITY
  WHEN
    Audit_Entry created OR Signature recorded
  THEN
    HASH_ENTRY {
      hash = SHA256(entry_content + timestamp + previous_hash)
    }
    
    STORE_WITH_PROOF {
      entry_id : [UUID]
      timestamp: [UTC]
      user_id  : [Who performed]
      action   : [What was done]
      hash     : [Cryptographic proof]
      tsa_proof: [RFC 3161 timestamp]
    }
    
    RETENTION_POLICY {
      duration : 10 years (legal requirement)
      zone     : Initially hot, then cold after 3 years
      backup   : Replicated, encrypted
    }
  END
  
  VERIFICATION:
    ON_RETRIEVAL {
      IF HASH_MISMATCH THEN
        Alert : "AUDIT_TRAIL_INTEGRITY_COMPROMISED"
        Action: Escalate to SOC
      END
    }
  END


RULE RG_009_MULTI_ENTITY_SCOPE
  WHEN
    Dossier.Entity defined at creation
  THEN
    APPLY_ENTITY_RULES {
      IF Dossier.Entity = "FR_001" THEN
        Use_RBACRules_FR
        Routing_FR_specific
      ELSE IF Dossier.Entity = "DE_002" THEN
        Use_RBACRules_DE
        Routing_DE_specific
      END
      
      IF Montant_Global > 100K THEN
        Add_GROUP_FINANCE_validation
        Notify_Group_HQ
      END
    }
  END
  
  AUDIT_ENTRY : "MULTI_ENTITY_[Entity]_[Scope]"


RULE RG_010_COMPLIANCE_GATE
  WHEN
    Dossier.Status = "CREATED"
  THEN
    VERIFY_CONFORMITY {
      Checks = [
        minimum_1_document_attached,
        circuit_has_valid_signatories,
        no_circular_dependencies,
        all_required_fields_filled,
        sla_realistic
      ]
      
      IF ANY_check_fails THEN
        REJECT_DEPOSIT
        Notify_Initiator : "Correct errors before resubmit"
      ELSE
        ACCEPT_DEPOSIT
        Create_Dossier_ID
        Notify_First_Validator
      END
    }
  END
  
  AUDIT_ENTRY : "CONFORMITY_GATE_[PASSED/FAILED]"
```

---

## CHECKLIST IMPLÉMENTATION

### Phase 1 : Conception (Semaines 1-3)

```
PRÉREQUIS TECHNIQUES
┌─────────────────────────────────────────────────────────┐
├─ Infrastructure existante diagnostiquée
│  ├─ [ ] Architecture SI décrite
│  ├─ [ ] Capacité serveurs évaluée (volumétrie)
│  ├─ [ ] Intégration ERP mappée (SAP, Oracle, etc.)
│  └─ [ ] SSO existant (LDAP, Okta, Azure AD)
│
├─ Cadre juridique défini
│  ├─ [ ] Loi applicable confirmée (France, UE)
│  ├─ [ ] Conformité eIDAS identifiée
│  ├─ [ ] Durée conservation définie
│  ├─ [ ] RGPD data controller identifié
│  └─ [ ] Privacy impact assessment complété
│
└─ Stakeholders identifics
   ├─ [ ] COPIL conformé (Legal, IT, Métier, Audit)
   ├─ [ ] Responsable métier nommé
   ├─ [ ] Responsable IT nommé
   ├─ [ ] Budget approuvé
   └─ [ ] Planning validé
```

### Phase 2 : Éspecification (Semaines 4-6)

```
FONCTIONNEL
┌─────────────────────────────────────────────────────────┐
├─ Processus métier détaillés
│  ├─ [ ] Cas d'usage rédigés (user stories)
│  ├─ [ ] Workflows BPMN finalisés
│  ├─ [ ] Règles de gestion documentées
│  ├─ [ ] Rôles & permissions matrice RACI validée
│  └─ [ ] Exceptions et cas limites couverts
│
├─ Données et intégrations
│  ├─ [ ] Schéma données défini
│  ├─ [ ] API intégration ERP specified
│  ├─ [ ] Webhooks événementiels mapés
│  ├─ [ ] Authentification SSO configurée
│  └─ [ ] Notificationcanaux définis (email, SMS, Slack)
│
└─ Performance & conformité
   ├─ [ ] SLA defined (par niveau, par type)
   ├─ [ ] Volumétrie estimée (nb dossiers/an)
   ├─ [ ] Audit trail retention définit (10 ans)
   ├─ [ ] Chiffrage stockage GED évalué
   └─ [ ] Benchmark outils parapheur (3 vendors min)
```

### Phase 3 : Sélection Éditeur (Semaine 7)

```
SÉLECTION & CONTRAT
┌─────────────────────────────────────────────────────────┐
├─ Appel d'offres lancé
│  ├─ [ ] Cahier charges rédigé
│  ├─ [ ] Critères évaluation pondérés
│  ├─ [ ] Vendors pré-qualifiés (3-5)
│  ├─ [ ] RFI/RFP distribué
│  └─ [ ] Démo en mode réel demandé
│
├─ Évaluation commerciale
│  ├─ [ ] Licence model négocié (SaaS/On-prem/Hybrid)
│  ├─ [ ] Support & maintenance defined (SLA)
│  ├─ [ ] Formation & accompagnement included
│  ├─ [ ] Coûts infrastructure évalués
│  └─ [ ] ROI business calculé
│
├─ Contrat signé
│  ├─ [ ] Conditions générales acceptées
│  ├─ [ ] Clauses conformité/sécu ajoutées
│  ├─ [ ] Clause escalade incluse
│  ├─ [ ] Data residency garantie
│  └─ [ ] Audit trail certifié
│
└─ Vendor sélectionné & kickoff
   ├─ [ ] Contrat signé
   ├─ [ ] Responsables vendor identifiés
   ├─ [ ] Accès environnement de dev fourni
   └─ [ ] Planning implémentation établi
```

### Phase 4 : Implémentation (Semaines 8-20)

```
PLATEFORME & INTÉGRATION
┌─────────────────────────────────────────────────────────┐
├─ Installation et configuration
│  ├─ [ ] Instance pré-prod provisionnée
│  ├─ [ ] SSO intégré et testé
│  ├─ [ ] Certificats PKI installés
│  ├─ [ ] TSA (TimeStamp Authority) connectée
│  ├─ [ ] Accès GED configuré
│  └─ [ ] Email service intégré
│
├─ Workflows et règles
│  ├─ [ ] Circuits de base créés
│  ├─ [ ] Règles conditionnelles implémentées
│  ├─ [ ] SLA timers configurés
│  ├─ [ ] Notifications templates crées
│  ├─ [ ] Escalades auto-setup
│  ├─ [ ] Délégations testées
│  └─ [ ] Exceptions managées (rejet, versioning, etc.)
│
├─ Intégrations système
│  ├─ [ ] API ERP connectée (SAP, Oracle)
│  ├─ [ ] Webhooks événementiels activés
│  ├─ [ ] Synchronisation bidirectionnelle OK
│  ├─ [ ] Audit trail dans GED validé
│  ├─ [ ] Authentification forte activée (OTP, 2FA)
│  └─ [ ] Monitoring et alertes actifs
│
└─ Tests et UAT
   ├─ [ ] Tests unitaires passés (90%+ coverage)
   ├─ [ ] Tests d'intégration valides
   ├─ [ ] Tests performance (volumétrie cible)
   ├─ [ ] Audit de sécurité complété
   ├─ [ ] Conformité légale attestée
   └─ [ ] UAT sign-off des métiers
```

### Phase 5 : Déploiement Prod (Semaine 21)

```
GO-LIVE
┌─────────────────────────────────────────────────────────┐
├─ Préparation déploiement
│  ├─ [ ] Plan de migration data révisé
│  ├─ [ ] Backup stratégie active
│  ├─ [ ] Runbook de rollback préparé
│  ├─ [ ] Communication stakeholders lancée
│  ├─ [ ] Formation utilisateurs complétée
│  ├─ [ ] Support en place (24/7 si critique)
│  └─ [ ] Monitoring alerts configurés
│
├─ Déploiement lui-même
│  ├─ [ ] Migration données réalisée
│  ├─ [ ] Vérification intégrité data
│  ├─ [ ] Instances production activées
│  ├─ [ ] DNS/routing mis à jour
│  ├─ [ ] SSL certificat actif
│  └─ [ ] Health check tous services OK
│
├─ Stabilisation post-go-live
│  ├─ [ ] Incidents premiers jours traités
│  ├─ [ ] Utilisateurs accompagnés (support)
│  ├─ [ ] Statistiques de démarrage collectées
│  ├─ [ ] Ajustements mineurs appliqués
│  └─ [ ] Lessons learned documentés
│
└─ Clôture déploiement
   ├─ [ ] Rapport déploiement finalisé
   ├─ [ ] Transition support en production
   ├─ [ ] Documentations As-Built créées
   └─ [ ] Contrats vendor finalisés
```

---

## SCHÉMA DONNÉES – Dossier

### Model Entités Principales

```json
{
  "Dossier": {
    "dossier_id": "UUID (primary key)",
    "external_id": "String (SAP:PO_123, etc.)",
    "title": "String(255)",
    "description": "Text",
    "classification": "Enum (Strategic, Achat, RH, etc.)",
    "entity": "String (FR_001, DE_002, etc.)",
    "initiator_id": "FK User",
    "type_document": "Enum (Achat, Contrat, RH, etc.)",
    "type_signature": "Enum (Simple, Avancée, Qualifiée)",
    "urgence": "Enum (Normal, Urgent)",
    "montant": "Decimal (optionnel)",
    "currency": "String (EUR, USD)",
    "status": "Enum (CRÉÉ, EN_ATTENTE_VALIDATION, EN_MODIFICATION, EN_SIGNATURE, SIGNÉ, ARCHIVÉ, ANNULÉ)",
    "created_at": "DateTime (UTC)",
    "updated_at": "DateTime (UTC)",
    "signature_completed_at": "DateTime (optionnel)",
    "archived_at": "DateTime (optionnel)",
    "retention_until": "DateTime (legal requirement)",
    "version": "Integer (v1, v2, etc.)",
    
    "Documents": [
      {
        "document_id": "UUID",
        "filename": "String",
        "mime_type": "String (application/pdf, etc.)",
        "filesize_bytes": "Integer",
        "hash_sha256": "String",
        "storage_path": "String",
        "uploaded_by": "FK User",
        "uploaded_at": "DateTime",
        "virus_scan_status": "Enum (PASSED, FAILED, PENDING)"
      }
    ],
    
    "Circuit": {
      "circuit_id": "UUID",
      "type_routing": "Enum (SEQUENTIAL, PARALLEL, CONDITIONAL, MIXED)",
      "stages": [
        {
          "stage_id": "UUID",
          "order": "Integer (1, 2, 3...)",
          "role_required": "String (Manager, Director, etc.)",
          "type_action": "Enum (VISA, VALIDATION, SIGNATURE)",
          "routing_type": "Enum (SEQ, PAR, COND)",
          "sla_hours": "Integer",
          "notifications": ["email", "dashboard", "sms"]
        }
      ]
    },
    
    "Validations": [
      {
        "validation_id": "UUID",
        "stage_id": "FK Stage",
        "validator_id": "FK User",
        "delegated_from": "FK User (optionnel, if delegation)",
        "status": "Enum (PENDING, APPROVED, REJECTED, DELEGATED)",
        "decision": "Enum (VALIDE, REJETTE, DEMANDE_CLARIF)",
        "comment": "Text",
        "rejection_category": "Enum (INCOMPLET, ERREUR, NON_CONFORME, AUTRE)",
        "rejection_reason": "Text",
        "created_at": "DateTime",
        "completed_at": "DateTime",
        "duration_seconds": "Integer"
      }
    ],
    
    "Signatures": [
      {
        "signature_id": "UUID",
        "signatory_id": "FK User",
        "signature_type": "Enum (SIMPLE, AVANCÉE, QUALIFIÉE)",
        "signature_value": "String (encoded signature)",
        "certificate_chain": "JSON (X.509 certs)",
        "hash_document": "String (SHA-256)",
        "timestamp_utc": "DateTime",
        "tsa_proof": "String (RFC 3161)",
        "ip_address": "String",
        "user_agent": "String",
        "status": "Enum (VALIDE, REVOQUÉE)",
        "hmac_verification": "String (proof integrity)"
      }
    ],
    
    "Audit_Trail": [
      {
        "entry_id": "UUID",
        "action_type": "Enum (CREATE, VALIDATE, REJECT, SIGN, ARCHIVE, etc.)",
        "actor_id": "FK User",
        "timestamp": "DateTime (UTC)",
        "ip_address": "String",
        "user_agent": "String",
        "details": "JSON",
        "hash_sha256": "String (immutable proof)",
        "tsa_timestamp": "String (RFC 3161)"
      }
    ]
  },
  
  "User": {
    "user_id": "UUID",
    "email": "String (unique)",
    "first_name": "String",
    "last_name": "String",
    "role": "Enum (Manager, Director, CEO, Admin, Auditeur, etc.)",
    "entity": "FK Entity",
    "status": "Enum (ACTIF, ABSENT, DISABLED)",
    "last_login": "DateTime",
    "certificate_id": "FK Certificate (optionnel, if signature avancée)",
    
    "Delegations": [
      {
        "delegation_id": "UUID",
        "delegate_to": "FK User",
        "valid_from": "DateTime",
        "valid_until": "DateTime",
        "reason": "String (Congé, Maladie, etc.)",
        "status": "Enum (ACTIVE, EXPIRED)",
        "created_by": "FK User"
      }
    ]
  },
  
  "Certificate": {
    "certificate_id": "UUID",
    "user_id": "FK User",
    "certificate_type": "Enum (X.509, eIDAS, etc.)",
    "public_key": "String",
    "issuer": "String",
    "valid_from": "DateTime",
    "valid_until": "DateTime",
    "thumbprint": "String (SHA-1)",
    "status": "Enum (VALIDE, EXPIRÉ, RÉVOQUÉ)"
  },
  
  "Entity": {
    "entity_id": "UUID",
    "code": "String (FR_001, DE_002)",
    "name": "String (France, Allemagne)",
    "country": "String (ISO 3166)",
    "legal_structure": "String",
    "address": "String",
    "vat_number": "String"
  }
}
```

---

## GLOSSAIRE ET ACRONYMES

### Termes métier

| Terme | Définition |
|-------|-----------|
| **Parapheur** | Système dématérialisant les workflows de validation/signature |
| **Dossier** | Ensemble document + métadonnées + historique validation/signature |
| **Circuit** | Chemin de validation (niveaux, rôles, ordre) |
| **Étape** | Une partie du circuit (un validateur, un rôle) |
| **Visa** | Approbation simple (not signature) |
| **Validation** | Approbation avec responsabilité |
| **Signature** | Acte engageant légalement (electronique) |
| **Sceau** | Preuve intégrité document (hash + signature autorité) |
| **Horodatage** | Timestamp cryptographe (RFC 3161) |
| **GED** | Gestion Électronique Documents (archivage) |

### Acronymes Techniques

| Acronyme | Signification |
|----------|--------------|
| **BPMN** | Business Process Model and Notation |
| **RACI** | Responsible, Accountable, Consulted, Informed |
| **SSO** | Single Sign-On |
| **RBAC** | Role-Based Access Control |
| **PKI** | Public Key Infrastructure |
| **TSA** | TimeStamp Authority |
| **RFC 3161** | Time-Stamp Protocol standard |
| **PKCS#7** | Signature encoding standard |
| **CMS** | Cryptographic Message Syntax |
| **eIDAS** | Electronic Identification & Trust Services (UE) |
| **HSM** | Hardware Security Module |
| **OTP** | One-Time Password |
| **2FA** | Two-Factor Authentication |
| **SLA** | Service Level Agreement |
| **TAT** | Turn-Around Time |
| **KPI** | Key Performance Indicator |
| **RGPD** | Réglement Général Protection Données |
| **PII** | Personally Identifiable Information |
| **SOC** | Security Operations Center |
| **API** | Application Programming Interface |
| **JSON** | JavaScript Object Notation |
| **REST** | Representational State Transfer |

---

## Résumé Livrables Complémentaires

✅ **Matrice RACI détaillée** : Rôles & responsabilités par activité (phase par phase)  
✅ **Catalogue 10 exceptions** : Cards complètes avec triggers, actions, impacts  
✅ **Template règles de gestion** : 10 règles SQL-like réutilisables  
✅ **Checklist implémentation** : 5 phases, 100+ items (concept à go-live)  
✅ **Schéma données** : Entités principales + relations JSON  
✅ **Glossaire & acronymes** : Référence terminologie  

**Prêt pour spécifications détaillées et mise en œuvre projet.**

