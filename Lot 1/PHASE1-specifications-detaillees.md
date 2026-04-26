# PHASE 1 — SPÉCIFICATIONS DÉTAILLÉES PARAPHEUR DIGITAL

**Document de cadrage projet — Niveau COPIL Direction**
**Référence** : PARAPH-PH1-V1.0
**Date** : 2026-04-24
**Classification** : Confidentiel — Diffusion restreinte COPIL/DSI/DAF
**Auteurs** : Equipe Transformation Digitale
**Statut** : Pour validation COPIL

---

## SYNTHÈSE EXÉCUTIVE (1 page)

### Recommandation

**Lancer immédiatement la production des 3 livrables Phase 1 sur 8 semaines avec un budget de 280 K€, pour un démarrage du développement le 2026-06-22.** Le ROI est démontré : chaque semaine de retard sur la Phase 1 décale le ROI projet de 1,4 semaine et coûte 47 K€ en charges projet improductives.

### Les 3 enjeux qui structurent la Phase 1

| # | Enjeu | Impact business | Niveau d'effort Phase 1 |
|---|-------|-----------------|------------------------|
| 1 | **Couverture fonctionnelle exhaustive** : 47 user stories réparties en 6 domaines, dont 12 critiques pour la conformité eIDAS | Sans cela : 18-24 mois de retravaux post go-live, coût estimé 1,2 M€ | 35 j/h Business Analyst |
| 2 | **Architecture cible robuste** : choix microservices vs monolithe modulaire, intégration SAP S/4HANA, infrastructure HSM pour signature qualifiée | Sans cela : refonte architecturale T+18 mois, coût 800 K€ + dette technique | 25 j/h Architecte |
| 3 | **Expérience utilisateur validée** : 12 parcours utilisateurs prototypés et testés sur 30 utilisateurs représentatifs (3 entités, 4 rôles) | Sans cela : adoption < 60 % au lieu de 90 %, perte ROI 2,1 M€/an | 20 j/h UX Designer |

### Décisions à prendre en COPIL (3 points)

| Décision | Options | Recommandation | Raison |
|----------|---------|----------------|--------|
| **D1** : Architecture cible | (a) Monolithe modulaire, (b) Microservices | **(b) Microservices** | Volumétrie cible 250 K dossiers/an, multi-entités → besoin scalabilité indépendante par bounded context |
| **D2** : Mode signature qualifiée | (a) Service tiers (DocuSign/Universign), (b) PKI interne + HSM | **(a) Service tiers Universign** | Coût TCO 3 ans : 380 K€ vs 920 K€, conformité eIDAS native, time-to-market -4 mois |
| **D3** : Stratégie UX research | (a) Wireframes → dev, (b) Prototypage Figma + tests usabilité 30 users | **(b) Tests usabilité** | Investissement 28 K€ → évite 340 K€ de refonte UI post go-live (ratio 1:12) |

### Risques Phase 1 (top 3)

| ID | Risque | P | I | Score | Mitigation |
|----|--------|---|---|-------|------------|
| R1 | Indisponibilité experts métier (RH/Legal/Finance) pour ateliers SFD | 4 | 5 | **20** | Sécuriser 2 j/sem dédiés sur 8 sem dès J0 + nominer un Sponsor Direction |
| R2 | Dérive fonctionnelle (scope creep) → +30 % périmètre, +6 sem | 4 | 4 | **16** | Gel scope à J+15, comité de changement avec arbitrage DAF/DSI hebdomadaire |
| R3 | Décision tardive D1/D2/D3 ci-dessus | 3 | 5 | **15** | COPIL J+5 dédié décisions architecturales, sinon escalade ComEx |

P = Probabilité (1-5), I = Impact (1-5), Score = P×I

---

## A. SPÉCIFICATIONS FONCTIONNELLES DÉTAILLÉES (SFD)

### A.1 Cartographie fonctionnelle — Vue d'ensemble

```
┌──────────────────────────────────────────────────────────────────┐
│                  PARAPHEUR DIGITAL — 6 DOMAINES                  │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  D1. Gestion       D2. Workflow      D3. Signature              │
│  Dossier           Validation        Électronique               │
│  (8 US)            (12 US)           (7 US)                     │
│                                                                  │
│  D4. Exception     D5. Audit &       D6. Administration         │
│  Management        Archivage         & Configuration            │
│  (9 US)            (5 US)            (6 US)                     │
│                                                                  │
│  TOTAL : 47 User Stories                                        │
└──────────────────────────────────────────────────────────────────┘
```

### A.2 Backlog détaillé — Inventaire exhaustif des 47 User Stories

#### Domaine D1 : Gestion du Dossier (8 US)

| ID | User Story | Priorité MoSCoW | Complexité (SP) | Dépendances |
|----|-----------|-----------------|-----------------|-------------|
| US-D1-001 | Créer un nouveau dossier avec métadonnées | **Must** | 5 | — |
| US-D1-002 | Charger 1 à 10 documents (max 50 MB total) | **Must** | 8 | US-D1-001 |
| US-D1-003 | Sauvegarder un dossier en brouillon | **Must** | 3 | US-D1-001 |
| US-D1-004 | Modifier un dossier brouillon | **Must** | 5 | US-D1-003 |
| US-D1-005 | Sélectionner ou personnaliser le circuit | **Must** | 13 | US-D1-001, US-D6-002 |
| US-D1-006 | Soumettre le dossier (lance le workflow) | **Must** | 8 | US-D1-005 |
| US-D1-007 | Cloner un dossier existant (template) | Should | 5 | US-D1-001 |
| US-D1-008 | Importer un dossier depuis SAP (API) | Could | 13 | Intégration ERP |

**Sous-total D1** : 60 Story Points

#### Domaine D2 : Workflow de Validation (12 US)

| ID | User Story | Priorité | SP | Dépendances |
|----|-----------|----------|-----|-------------|
| US-D2-001 | Recevoir une notification de tâche à valider | **Must** | 5 | US-D1-006 |
| US-D2-002 | Visualiser ma liste de tâches en attente | **Must** | 5 | US-D2-001 |
| US-D2-003 | Consulter un dossier (lecture) | **Must** | 8 | US-D2-001 |
| US-D2-004 | Approuver un dossier | **Must** | 5 | US-D2-003 |
| US-D2-005 | Rejeter un dossier avec motif catégorisé | **Must** | 8 | US-D2-003 |
| US-D2-006 | Demander une clarification (retour arrière) | Should | 8 | US-D2-003 |
| US-D2-007 | Annoter un document (commentaires inline) | Should | 13 | US-D2-003 |
| US-D2-008 | Validation parallèle (n acteurs simultanés) | **Must** | 13 | US-D2-004 |
| US-D2-009 | Validation conditionnelle (règles métier) | **Must** | 13 | US-D2-004, US-D6-002 |
| US-D2-010 | Réassigner une tâche à un collègue | Should | 8 | US-D2-002 |
| US-D2-011 | Suivre l'état d'un dossier en temps réel | **Must** | 5 | US-D2-001 |
| US-D2-012 | Dashboard "Mes dossiers en cours" (initiateur) | **Must** | 8 | US-D2-011 |

**Sous-total D2** : 99 Story Points

#### Domaine D3 : Signature Électronique (7 US)

| ID | User Story | Priorité | SP | Dépendances |
|----|-----------|----------|-----|-------------|
| US-D3-001 | Apposer une signature simple (OTP) | **Must** | 13 | Service auth |
| US-D3-002 | Apposer une signature avancée (PKI) | **Must** | 21 | Infrastructure PKI |
| US-D3-003 | Apposer une signature qualifiée (eIDAS) | **Must** | 21 | Service tiers Universign |
| US-D3-004 | Vérifier l'intégrité d'une signature | **Must** | 8 | US-D3-001 à 003 |
| US-D3-005 | Télécharger le document signé final | **Must** | 5 | US-D3-001 à 003 |
| US-D3-006 | Télécharger le dossier de preuve | **Must** | 8 | US-D5-002 |
| US-D3-007 | Signature mobile (app native) | Could | 21 | Phase 2 envisagée |

**Sous-total D3** : 97 Story Points (dont 21 reportés Phase 2)

#### Domaine D4 : Exception Management (9 US)

| ID | User Story | Priorité | SP | Dépendances |
|----|-----------|----------|-----|-------------|
| US-D4-001 | Configurer une délégation proactive | **Must** | 8 | US-D6-001 |
| US-D4-002 | Détection automatique d'absence (>3j) | Should | 13 | Intégration RH/AD |
| US-D4-003 | Escalade automatique sur dépassement SLA | **Must** | 13 | Moteur SLA |
| US-D4-004 | Forcer une signature (admin override) | **Must** | 5 | Rôles élevés |
| US-D4-005 | Annuler un dossier (avant clôture) | **Must** | 5 | US-D1-006 |
| US-D4-006 | Versionner un dossier (V1 → V2) | Should | 13 | US-D1-002 |
| US-D4-007 | Révoquer une signature (avant archivage) | Could | 13 | US-D3-001 à 003 |
| US-D4-008 | Caviarder un document (PII redaction) | Should | 13 | Compliance RGPD |
| US-D4-009 | Notifier une exception au bon acteur | **Must** | 5 | Service notif |

**Sous-total D4** : 88 Story Points

#### Domaine D5 : Audit & Archivage (5 US)

| ID | User Story | Priorité | SP | Dépendances |
|----|-----------|----------|-----|-------------|
| US-D5-001 | Consulter l'audit trail d'un dossier | **Must** | 5 | Logging |
| US-D5-002 | Générer le dossier de preuve PDF | **Must** | 13 | US-D3-* |
| US-D5-003 | Archiver automatiquement vers GED | **Must** | 13 | Intégration GED |
| US-D5-004 | Rechercher dans les dossiers archivés | **Must** | 8 | US-D5-003 |
| US-D5-005 | Exporter un rapport audit (CSV/PDF) | Should | 8 | US-D5-001 |

**Sous-total D5** : 47 Story Points

#### Domaine D6 : Administration & Configuration (6 US)

| ID | User Story | Priorité | SP | Dépendances |
|----|-----------|----------|-----|-------------|
| US-D6-001 | Gérer les utilisateurs et rôles (RBAC) | **Must** | 13 | SSO |
| US-D6-002 | Configurer un circuit de validation | **Must** | 21 | — |
| US-D6-003 | Définir des règles métier (DSL low-code) | Should | 21 | US-D6-002 |
| US-D6-004 | Configurer les SLA par type de dossier | **Must** | 8 | US-D6-002 |
| US-D6-005 | Gérer les certificats et clés | **Must** | 13 | PKI |
| US-D6-006 | Configurer les notifications (templates) | Should | 8 | Service notif |

**Sous-total D6** : 84 Story Points

#### Récapitulatif Backlog

| Domaine | Nb US | Story Points | Must (P0) | Should (P1) | Could (P2) |
|---------|-------|--------------|-----------|-------------|------------|
| D1 — Gestion Dossier | 8 | 60 | 6 | 1 | 1 |
| D2 — Workflow | 12 | 99 | 9 | 3 | 0 |
| D3 — Signature | 7 | 97 | 6 | 0 | 1 |
| D4 — Exception | 9 | 88 | 5 | 3 | 1 |
| D5 — Audit | 5 | 47 | 4 | 1 | 0 |
| D6 — Admin | 6 | 84 | 5 | 1 | 0 |
| **TOTAL** | **47** | **475** | **35** | **9** | **3** |

**Note méthodologique** : 1 SP ≈ 0,5 j/h développement (équipe senior). 475 SP ≈ 240 j/h dev = 12 semaines équipe de 4 développeurs. Ajout 30 % testing/QA = 16 semaines.

### A.3 User Story type — Modèle exhaustif (US-D1-001)

```
┌─────────────────────────────────────────────────────────────────┐
│ US-D1-001 — Créer un nouveau dossier avec métadonnées          │
├─────────────────────────────────────────────────────────────────┤
│
│ ROLE          : Initiateur
│ PRIORITÉ      : Must (P0) — Bloquant MVP
│ STORY POINTS  : 5
│ SPRINT CIBLE  : Sprint 1
│
│ NARRATIVE
│ ─────────
│ EN TANT QU'   Initiateur (Manager opérationnel, Acheteur, RH)
│ JE VEUX       créer un nouveau dossier en saisissant les
│               métadonnées obligatoires et en chargeant les
│               documents associés
│ AFIN DE       lancer un workflow de validation et signature
│               sur un livrable nécessitant approbation hiérarchique
│
│ PRÉCONDITIONS
│ ─────────────
│ ✓ L'utilisateur est authentifié (SSO actif)
│ ✓ L'utilisateur a le rôle "Initiateur" sur au moins une entité
│ ✓ Au moins un type de dossier est configuré pour son entité
│
│ FLUX NOMINAL (Happy Path)
│ ─────────────────────────
│ 1. L'utilisateur clique sur "+ Nouveau dossier" depuis le dashboard
│ 2. Le système affiche le formulaire de création (modal/page)
│ 3. L'utilisateur sélectionne le Type de dossier (dropdown)
│ 4. Le système charge dynamiquement les champs métadonnées
│    associés au type sélectionné
│ 5. L'utilisateur saisit les métadonnées obligatoires :
│    - Titre (texte, 5-255 car.)
│    - Description (texte, 0-2000 car.)
│    - Entité (dropdown, restreint au scope user)
│    - Classification (dropdown selon type)
│    - Urgence (Normal/Urgent)
│ 6. L'utilisateur clique "Suivant : Documents"
│ 7. Le système valide les données et persiste le dossier en
│    statut "BROUILLON" avec un identifiant unique (UUID)
│ 8. Le système redirige vers l'écran de chargement documents (US-D1-002)
│
│ FLUX ALTERNATIFS
│ ────────────────
│ A1. Sauvegarder en brouillon depuis l'écran de saisie
│     → Pas de validation des champs obligatoires, persistance partielle
│
│ A2. Annuler la création
│     → Confirmation modal "Êtes-vous sûr ? Données perdues" si données saisies
│     → Retour dashboard sans persistance
│
│ FLUX D'ERREUR
│ ─────────────
│ E1. Champ obligatoire manquant
│     → Message inline rouge sous le champ : "Ce champ est obligatoire"
│     → Bouton "Suivant" désactivé jusqu'à correction
│
│ E2. Titre dépasse 255 caractères
│     → Compteur visible 254/255, blocage saisie au caractère 255
│     → Message : "Maximum 255 caractères"
│
│ E3. Erreur backend (timeout, 500)
│     → Toast erreur : "Erreur technique, veuillez réessayer"
│     → Bouton "Réessayer", données conservées en local storage
│     → Log incident dans Sentry
│
│ E4. Session expirée
│     → Redirection page login avec message "Session expirée"
│     → Sauvegarde brouillon local pour récupération post-login
│
│ ACCEPTANCE CRITERIA (testable)
│ ──────────────────────────────
│ AC1. ÉTANT DONNÉ un utilisateur authentifié avec rôle Initiateur
│      QUAND il remplit tous les champs obligatoires correctement
│      ET clique sur "Suivant : Documents"
│      ALORS un dossier est créé en statut BROUILLON
│      ET un UUID lui est attribué
│      ET il est redirigé vers l'écran upload documents
│      ET un événement "DOSSIER_CRÉÉ" est tracé en audit
│
│ AC2. ÉTANT DONNÉ un utilisateur authentifié
│      QUAND il omet le champ Titre
│      ET clique sur "Suivant"
│      ALORS un message d'erreur s'affiche sous le champ Titre
│      ET le formulaire n'est pas soumis
│      ET le focus est placé sur le champ en erreur
│
│ AC3. ÉTANT DONNÉ un utilisateur ayant accès à 3 entités
│      QUAND il ouvre le dropdown Entité
│      ALORS seules ses 3 entités autorisées apparaissent
│      ET l'entité par défaut est celle de son profil principal
│
│ AC4. ÉTANT DONNÉ un dossier en cours de saisie
│      QUAND l'utilisateur quitte la page sans sauvegarder
│      ALORS un dialogue de confirmation s'affiche
│      ET les données sont conservées en localStorage (24h)
│
│ AC5. ÉTANT DONNÉ une saisie de 1000 dossiers/heure (load test)
│      QUAND le système est sous charge
│      ALORS le temps de réponse de la création reste < 500ms (p95)
│      ET aucune erreur 5xx n'est observée
│
│ RÈGLES DE GESTION ASSOCIÉES (testables)
│ ────────────────────────────────────────
│ RG-001 : IF Type_Dossier = "Achat"
│          THEN Champs additionnels obligatoires :
│                Montant (decimal > 0), Devise (EUR/USD/GBP),
│                Fournisseur (autocomplete)
│
│ RG-002 : IF Urgence = "Urgent"
│          THEN SLA_Cible = SLA_Standard × 0.5
│          AND Notification_canal = "Email + SMS + Push"
│          AND Affichage badge "URGENT" rouge sur le dossier
│
│ RG-003 : Le titre doit être unique par initiateur sur 90 jours
│          IF (Initiateur, Titre) existe sur 90 derniers jours
│          THEN Warning non-bloquant : "Vous avez déjà créé un
│                dossier avec ce titre le [date]. Continuer ?"
│
│ RG-004 : L'entité du dossier détermine la juridiction et la
│          loi applicable. Une fois soumis, l'entité ne peut plus
│          être modifiée (seul un Admin peut, avec justification).
│
│ DONNÉES PERSISTÉES
│ ──────────────────
│ Table : dossiers
│ Colonnes :
│   - dossier_id (UUID, PK, indexé)
│   - external_ref (varchar 100, nullable, indexé)
│   - title (varchar 255, NOT NULL)
│   - description (text, nullable)
│   - type_dossier (FK, NOT NULL)
│   - entity_id (FK, NOT NULL)
│   - classification (varchar 50, NOT NULL)
│   - urgence (enum: NORMAL/URGENT)
│   - amount (decimal 18,2, nullable)
│   - currency (varchar 3, nullable)
│   - status (enum, NOT NULL, default BROUILLON)
│   - initiator_id (FK users, NOT NULL)
│   - created_at (timestamp UTC, NOT NULL)
│   - updated_at (timestamp UTC, NOT NULL)
│   - retention_until (timestamp, calculé selon RG-Rétention)
│
│ Index :
│   - idx_initiator_status (initiator_id, status)
│   - idx_entity_status (entity_id, status)
│   - idx_created_at (created_at DESC)
│
│ APIs APPELÉES
│ ─────────────
│ POST /api/v1/dossiers
│   Body : { title, description, type_dossier_id, entity_id,
│            classification, urgence, amount?, currency? }
│   Response 201 : { dossier_id, status: "BROUILLON", ... }
│   Response 400 : { errors: [{ field, message }] }
│   Response 401 : { error: "Unauthorized" }
│   Response 403 : { error: "Forbidden — entity not allowed" }
│
│ GET /api/v1/types-dossier?entity_id={id}
│   Response 200 : [{ id, name, fields_schema, ... }]
│
│ GET /api/v1/users/me/entities
│   Response 200 : [{ id, name, country, ... }]
│
│ ÉVÉNEMENTS ÉMIS
│ ───────────────
│ Topic : dossier.lifecycle
│   Event : DOSSIER_CRÉÉ
│   Payload : { dossier_id, initiator_id, entity_id,
│               type_dossier, timestamp_utc }
│   Subscribers : Audit, Notification, Analytics
│
│ MAQUETTES UI
│ ────────────
│ - Wireframe : FIG-D1-001-WF (Figma)
│ - Mockup HD : FIG-D1-001-HD (Figma)
│ - Prototype : FIG-D1-001-PROTO (Figma interactive)
│
│ PERFORMANCE & SLA
│ ─────────────────
│ - Temps réponse création (p95) : < 500 ms
│ - Temps réponse création (p99) : < 1 200 ms
│ - Disponibilité : 99,5 % (SLA contractuel)
│ - Capacité : 50 dossiers/seconde en pic
│
│ SÉCURITÉ
│ ────────
│ - Authentification obligatoire (JWT, expire 8h)
│ - Autorisation : RBAC (rôle Initiateur sur l'entité ciblée)
│ - Validation input : XSS, injection SQL, taille limite
│ - Audit : tracé "DOSSIER_CRÉÉ" + IP + User-Agent + Session-ID
│ - Rate limiting : 60 créations/heure par utilisateur
│
│ RGPD
│ ────
│ - Données personnelles : initiator_id (lien table users)
│ - Base légale : intérêt légitime (gestion documentaire entreprise)
│ - Durée conservation : selon classification (1-10 ans)
│ - Droit accès/rectification : via Admin (workflow dédié)
│
│ TESTS REQUIS
│ ────────────
│ - Tests unitaires backend : ≥ 90 % coverage controller + service
│ - Tests d'intégration : 12 scénarios couvrant AC1-AC5 + erreurs
│ - Tests E2E (Playwright) : 4 parcours (happy path + 3 erreurs)
│ - Tests de charge : 1000 créations en 60s, p95 < 500ms
│ - Tests sécurité : SAST + DAST sur endpoint POST /dossiers
│
│ DEFINITION OF DONE
│ ──────────────────
│ ☐ Code revu par un pair (PR approuvée)
│ ☐ Tests unitaires écrits et passants (coverage ≥ 90%)
│ ☐ Tests E2E passants en CI
│ ☐ Documentation API (OpenAPI/Swagger) à jour
│ ☐ Audit trail vérifié manuellement
│ ☐ Performance validée (load test passé)
│ ☐ Accessibilité WCAG 2.1 AA validée (audit Lighthouse)
│ ☐ Validation produit owner (démo sprint review)
│ ☐ Validation UX (review maquette implémentée)
│ ☐ Validation sécurité (scan SAST/DAST clean)
│
│ ESTIMATIONS DÉTAILLÉES
│ ──────────────────────
│ - Backend (API + DB + tests)        : 2 j/h
│ - Frontend (UI React + tests)       : 1,5 j/h
│ - Tests E2E + intégration            : 0,5 j/h
│ - Code review + corrections          : 0,5 j/h
│ - Documentation + démo               : 0,5 j/h
│ TOTAL                                 : 5 j/h ≈ 5 SP ✓
│
└─────────────────────────────────────────────────────────────────┘
```

### A.4 Règles de gestion — Référentiel exhaustif (35 règles testables)

#### RG Fonctionnement Workflow (10 règles)

| ID | Règle | Type | Testabilité |
|----|-------|------|-------------|
| RG-W-001 | IF Montant < 5K€ THEN Circuit_Type = "SIMPLE" (Manager → Sig.) | Routage | Unit test sur 12 valeurs limites |
| RG-W-002 | IF Montant >= 5K€ AND < 50K€ THEN Circuit_Type = "STANDARD" (Manager → Finance ∥ Legal → Sig.) | Routage | Unit test |
| RG-W-003 | IF Montant >= 50K€ THEN Circuit_Type = "COMPLEX" (Manager → Finance ∥ Legal ∥ Compliance → Director → CEO_Sig.) | Routage | Unit test |
| RG-W-004 | IF Type_Dossier = "RH_Embauche" THEN Circuit = [Manager → RH → Dir_RH → CEO_Sig.] indépendamment du montant | Routage | Unit test |
| RG-W-005 | IF Entity = "Filiale_X" AND Type ≠ "Local" THEN Ajouter Group_Finance dans le circuit | Routage multi-entité | Unit test |
| RG-W-006 | IF Tous_Validateurs_Niveau_K = APPROVED THEN Passage automatique Niveau_K+1 | Transition | Test intégration |
| RG-W-007 | IF Validateur = REJECTED THEN Statut = EN_MODIFICATION ET Notification Initiateur | Transition | Test intégration |
| RG-W-008 | IF Tous_Niveaux_Validés THEN Statut = EN_SIGNATURE | Transition | Test intégration |
| RG-W-009 | IF Tous_Signataires = SIGNED THEN Statut = SIGNÉ ET Lancement archivage | Transition | Test intégration |
| RG-W-010 | Mode parallèle : Tous validateurs même niveau notifiés simultanément, attente complète avant transition | Routage parallèle | Test intégration |

#### RG SLA et Escalade (8 règles)

| ID | Règle | Valeur | Configurable |
|----|-------|--------|--------------|
| RG-SLA-001 | SLA Manager (validation niveau 1) | 48h ouvrées | Oui (Admin) |
| RG-SLA-002 | SLA Director (validation niveau 2+) | 24h ouvrées | Oui |
| RG-SLA-003 | SLA Finance/Legal (parallèle) | 72h ouvrées | Oui |
| RG-SLA-004 | SLA Signature (CEO/PDG) | 24h ouvrées | Oui |
| RG-SLA-005 | IF Urgence = "URGENT" THEN SLA × 0,5 | Multiplicateur 0,5 | Oui |
| RG-SLA-006 | IF SLA dépassé THEN Notification 1 (rappel email) à T+0h | Trigger | Non |
| RG-SLA-007 | IF SLA + 24h dépassé THEN Notification 2 (email + SMS) + CC manager | Trigger | Non |
| RG-SLA-008 | IF SLA + 48h dépassé THEN ESCALADE_AUTO : Statut → EN_ESCALADE, transfert N+1 | Trigger | Non |

#### RG Signature & Conformité (7 règles)

| ID | Règle | Norme |
|----|-------|-------|
| RG-SIG-001 | Signature simple : OTP 6 chiffres SMS, validité 5min, 3 tentatives max | Interne |
| RG-SIG-002 | Signature avancée : Cert X.509 RSA 2048+ bits, horodatage TSA RFC 3161 | eIDAS Art. 26 |
| RG-SIG-003 | Signature qualifiée : Service tiers eIDAS-listé, HSM Common Criteria EAL4+ | eIDAS Art. 28 |
| RG-SIG-004 | Vérification cert non révoqué (CRL/OCSP) avant chaque signature | PKI standard |
| RG-SIG-005 | IF Cert expire < 30j THEN Notification renouvellement à signataire + admin | Interne |
| RG-SIG-006 | IF Cert expiré OU révoqué THEN Blocage signature, message explicite | Sécurité |
| RG-SIG-007 | Hash document signé : SHA-256 minimum (SHA-1 interdit) | NIST SP 800-131A |

#### RG Délégation (5 règles)

| ID | Règle | Contrainte |
|----|-------|------------|
| RG-DEL-001 | Délégué.Role >= Délégant.Role (pas de délégation descendante) | Bloquante |
| RG-DEL-002 | Durée délégation max : 30 jours consécutifs | Bloquante (admin override) |
| RG-DEL-003 | Une délégation à la fois par utilisateur source | Bloquante |
| RG-DEL-004 | IF User absent > 3j (LDAP/HR) AND aucune délégation THEN Alerte admin | Préventive |
| RG-DEL-005 | Toute action via délégation marquée "[via délégation de User_X]" en audit | Traçabilité |

#### RG Exceptions (5 règles)

| ID | Règle | Limite |
|----|-------|--------|
| RG-EXC-001 | Max 2 rejets par dossier, 3e rejet → escalade Admin manuelle | Anti-boucle |
| RG-EXC-002 | Max 2 retours arrière par dossier (anti ping-pong) | Anti-boucle |
| RG-EXC-003 | Max 3 versions par dossier, 4e → review obligatoire Admin | Qualité |
| RG-EXC-004 | Annulation possible uniquement avant signature complète | Cohérence |
| RG-EXC-005 | Caviardage PII uniquement par Admin avec justification écrite | RGPD |

### A.5 Modèle de données fonctionnel — Entités principales

```
┌─────────────────────────────────────────────────────────────────┐
│                  MODÈLE LOGIQUE — VUE FONCTIONNELLE              │
├─────────────────────────────────────────────────────────────────┤
│
│  ┌──────────────┐         ┌─────────────────┐
│  │   ENTITY     │1───────*│     USER        │
│  │ (Filiale)    │         │                 │
│  └──────────────┘         └─────────────────┘
│         │                          │
│         │ 1                        │ 1
│         │                          │
│         * *                        * *
│  ┌──────────────────┐      ┌──────────────────┐
│  │     DOSSIER      │1────*│   VALIDATION     │
│  │                  │      │                  │
│  └──────────────────┘      └──────────────────┘
│         │ 1
│         │
│         │ *
│  ┌──────────────────┐      ┌──────────────────┐
│  │    DOCUMENT      │      │    SIGNATURE     │
│  │                  │      │                  │
│  └──────────────────┘      └──────────────────┘
│         │                          │
│         │                          │
│         └──────┬───────────────────┘
│                │
│                * (via dossier)
│  ┌──────────────────────────┐
│  │     AUDIT_TRAIL          │
│  │  (immutable, append-only)│
│  └──────────────────────────┘
│
│  Cardinalités clés :
│  - 1 Entity → N Users (un user dans plusieurs entités)
│  - 1 User → N Dossiers (initiateur)
│  - 1 Dossier → N Documents (1 à 10)
│  - 1 Dossier → N Validations (1 par étape circuit)
│  - 1 Dossier → N Signatures (selon circuit)
│  - 1 Dossier → N Audit_Entries (tous événements)
│
└─────────────────────────────────────────────────────────────────┘
```

### A.6 Plan de release — MVP, V1, V2

| Release | Périmètre | US incluses | Date cible | SP |
|---------|-----------|-------------|-----------|-----|
| **MVP** | Workflow simple + Sig. simple + 1 entité | 28 US Must (D1-D2-D5 partiels) | 2026-09-30 | 220 |
| **V1** | Sig. avancée/qualifiée + Multi-entités + Exceptions | 7 US Must restantes + 6 Should | 2026-12-15 | 165 |
| **V2** | Mobile + IA + Optimisations | 3 Could + Phase 2 | 2027-Q2 | 90 |

---

## B. SPÉCIFICATIONS TECHNIQUES (TS)

### B.1 Architecture cible — Recommandation

**Choix architectural : Microservices avec API Gateway, Event-Driven**

| Critère | Monolithe modulaire | **Microservices (recommandé)** |
|---------|--------------------|-------------------------------|
| Time to market | Plus rapide (-2 mois) | Plus long initialement |
| Scalabilité | Verticale uniquement | Horizontale par service |
| Évolutivité | Couplage fort | Bounded contexts isolés |
| Coût infra année 1 | 60 K€ | 95 K€ (+35 K€) |
| Coût infra année 3 | 180 K€ (croissance forcée) | 210 K€ (croissance maîtrisée) |
| Robustesse | Single point of failure | Résilience par service |
| Volumétrie supportée | < 100 K dossiers/an | > 500 K dossiers/an |
| **Adapté à notre cible** | Non (250 K dossiers cible) | **Oui** |

### B.2 Vue C4 — Niveau Container

```
┌────────────────────────────────────────────────────────────────┐
│                  CONTEXTE — PARAPHEUR DIGITAL                  │
└────────────────────────────────────────────────────────────────┘

      ┌─────────────┐                 ┌──────────────────┐
      │  Utilisateur│                 │   ERP SAP        │
      │  (Web/Mobile│                 │  S/4HANA         │
      └──────┬──────┘                 └────────┬─────────┘
             │                                 │
             │ HTTPS                           │ REST/Webhooks
             ▼                                 │
      ┌──────────────────────────────────────────────────┐
      │            API GATEWAY (Kong/AWS APIGW)           │
      │     Authent. • Rate limiting • Routing • Logs     │
      └────────┬─────────────────────────────────────────┘
               │
               │ Internal mesh (mTLS)
               ▼
   ┌───────────────────────────────────────────────────────┐
   │                  MICROSERVICES (8)                    │
   │                                                       │
   │  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐ │
   │  │  Dossier-svc │  │ Workflow-svc │  │ Signature-  │ │
   │  │   (Java/SB)  │  │  (Java/SB)   │  │ svc (Java)  │ │
   │  └──────┬───────┘  └──────┬───────┘  └──────┬──────┘ │
   │         │                 │                 │        │
   │  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴──────┐ │
   │  │ Audit-svc    │  │ Notif-svc    │  │ Admin-svc   │ │
   │  │  (Go)        │  │  (Node.js)   │  │  (Java)     │ │
   │  └──────────────┘  └──────────────┘  └─────────────┘ │
   │                                                       │
   │  ┌──────────────┐  ┌──────────────┐                  │
   │  │ Search-svc   │  │ Integration- │                  │
   │  │ (ElasticS.)  │  │ svc (Java)   │                  │
   │  └──────────────┘  └──────────────┘                  │
   │                                                       │
   └───────────────────────────────────────────────────────┘
                          │
            ┌─────────────┼──────────────────┐
            ▼             ▼                  ▼
     ┌──────────┐   ┌──────────┐      ┌──────────────┐
     │PostgreSQL│   │  Redis   │      │ Kafka (event │
     │ (master) │   │  (cache) │      │  streaming)  │
     └──────────┘   └──────────┘      └──────────────┘
            │                                  │
            ▼                                  ▼
     ┌──────────────┐                ┌──────────────────┐
     │  S3 (docs)   │                │  GED (M-Files /  │
     │  + KMS       │                │  Nuxeo)          │
     └──────────────┘                └──────────────────┘
                                              │
                                              ▼
                                      ┌──────────────┐
                                      │  Universign  │
                                      │  (eIDAS sig) │
                                      └──────────────┘
```

### B.3 Stack technologique — Choix justifiés

| Couche | Techno retenue | Alternatives écartées | Justification |
|--------|----------------|----------------------|---------------|
| **Frontend** | React 18 + TypeScript + Vite | Vue, Angular | Écosystème mature, recrutement facile, perf SSR avec Next.js si besoin |
| **State management** | Zustand + React Query | Redux, MobX | Léger, excellent DX, gestion serverState découplée |
| **Backend principal** | Java 21 + Spring Boot 3.2 | Node.js, .NET | Compétences in-house, écosystème entreprise, perf JVM |
| **Backend léger** | Node.js 20 (Notif), Go 1.22 (Audit) | Java pour tout | Notif = I/O bound (Node OK), Audit = haute throughput (Go) |
| **API Gateway** | Kong Enterprise | AWS API Gateway, Apigee | Multi-cloud, OSS option, plugins riches, perf |
| **Base de données** | PostgreSQL 16 (HA) | MySQL, Oracle | RGPD-friendly, JSON natif, partitioning, coût Oracle évité (~120 K€/an) |
| **Cache** | Redis 7 (Cluster) | Memcached, Hazelcast | Pub/Sub natif (utile pour notif), persistance optionnelle |
| **Event streaming** | Apache Kafka 3.7 | RabbitMQ, AWS Kinesis | Throughput élevé, replay, écosystème connecteurs (SAP) |
| **Search** | Elasticsearch 8 | Solr, OpenSearch | Recherche full-text + facettes archive, écosystème mature |
| **Object storage** | S3 (AWS) ou MinIO (on-prem) | Azure Blob, GCS | Standard de fait, chiffrement KMS, lifecycle policies |
| **Container** | Docker + Kubernetes (EKS/AKS) | Nomad, ECS | Standard industrie, multi-cloud, écosystème |
| **CI/CD** | GitLab CI ou GitHub Actions | Jenkins | Cloud-native, IaC-friendly |
| **Monitoring** | Datadog ou Grafana+Prometheus | New Relic | APM + logs + traces unifiés |
| **Sig. qualifiée** | Universign (cloud) | DocuSign, Yousign, PKI interne | TCO 380 K€ vs 920 K€ interne, eIDAS native, time-to-market |

### B.4 API REST — Contrats clés (extraits)

#### Endpoint critique : POST /api/v1/dossiers

```yaml
# OpenAPI 3.1 specification (extrait)
openapi: 3.1.0
info:
  title: Parapheur API
  version: 1.0.0
  description: |
    API REST du parapheur digital — Conforme eIDAS, RGPD.
    Authentification : OAuth 2.0 + JWT Bearer.

paths:
  /api/v1/dossiers:
    post:
      summary: Créer un nouveau dossier
      operationId: createDossier
      tags: [Dossiers]
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateDossierRequest'
      responses:
        '201':
          description: Dossier créé avec succès
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DossierResponse'
          headers:
            Location:
              schema: { type: string }
              description: URI du dossier créé
            X-Request-Id:
              schema: { type: string, format: uuid }
        '400':
          description: Données invalides
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ValidationError'
        '401':
          description: Non authentifié
        '403':
          description: Non autorisé (entité hors scope)
        '429':
          description: Rate limit dépassé (60 req/h)
          headers:
            Retry-After:
              schema: { type: integer }

components:
  schemas:
    CreateDossierRequest:
      type: object
      required: [title, type_dossier_id, entity_id, classification, urgence]
      properties:
        title:
          type: string
          minLength: 5
          maxLength: 255
          example: "Achat serveurs DC Paris"
        description:
          type: string
          maxLength: 2000
        type_dossier_id:
          type: string
          format: uuid
        entity_id:
          type: string
          format: uuid
        classification:
          type: string
          enum: [PUBLIC, INTERNE, CONFIDENTIEL, SECRET]
        urgence:
          type: string
          enum: [NORMAL, URGENT]
          default: NORMAL
        amount:
          type: number
          format: decimal
          minimum: 0
        currency:
          type: string
          enum: [EUR, USD, GBP]
        external_ref:
          type: string
          maxLength: 100
          description: Référence externe (ex SAP PO_123)

    DossierResponse:
      type: object
      properties:
        dossier_id: { type: string, format: uuid }
        title: { type: string }
        status: { type: string, enum: [BROUILLON, EN_ATTENTE_VALIDATION, ...] }
        circuit: { $ref: '#/components/schemas/Circuit' }
        created_at: { type: string, format: date-time }
        # ... autres champs

    ValidationError:
      type: object
      properties:
        error_code: { type: string, example: "VALIDATION_FAILED" }
        message: { type: string }
        errors:
          type: array
          items:
            type: object
            properties:
              field: { type: string }
              code: { type: string }
              message: { type: string }
        request_id: { type: string, format: uuid }

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

#### Inventaire complet des endpoints (43 endpoints)

| Domaine | Méthode | Endpoint | Description |
|---------|---------|----------|-------------|
| Dossiers | POST | /api/v1/dossiers | Créer dossier |
| Dossiers | GET | /api/v1/dossiers/{id} | Lire dossier |
| Dossiers | PUT | /api/v1/dossiers/{id} | Modifier brouillon |
| Dossiers | DELETE | /api/v1/dossiers/{id} | Supprimer brouillon |
| Dossiers | POST | /api/v1/dossiers/{id}/submit | Soumettre |
| Dossiers | POST | /api/v1/dossiers/{id}/cancel | Annuler |
| Dossiers | GET | /api/v1/dossiers | Lister (filtres) |
| Documents | POST | /api/v1/dossiers/{id}/documents | Upload doc |
| Documents | GET | /api/v1/dossiers/{id}/documents/{docId} | Télécharger |
| Documents | DELETE | /api/v1/dossiers/{id}/documents/{docId} | Supprimer |
| Validations | POST | /api/v1/validations/{id}/approve | Approuver |
| Validations | POST | /api/v1/validations/{id}/reject | Rejeter |
| Validations | POST | /api/v1/validations/{id}/return | Retour arrière |
| Validations | GET | /api/v1/validations/me | Mes tâches |
| Signatures | POST | /api/v1/signatures/simple | Sig. simple |
| Signatures | POST | /api/v1/signatures/advanced | Sig. avancée |
| Signatures | POST | /api/v1/signatures/qualified | Sig. qualifiée |
| Signatures | GET | /api/v1/signatures/{id}/proof | Preuve sig. |
| Signatures | GET | /api/v1/signatures/{id}/verify | Vérifier intégrité |
| Audit | GET | /api/v1/dossiers/{id}/audit | Trail dossier |
| Audit | GET | /api/v1/audit/export | Export CSV/PDF |
| Admin | GET/POST/PUT | /api/v1/admin/users | CRUD users |
| Admin | GET/POST/PUT | /api/v1/admin/circuits | CRUD circuits |
| Admin | GET/POST/PUT | /api/v1/admin/rules | CRUD règles métier |
| ... | ... | ... | (43 endpoints au total) |

### B.5 Modèle de données physique — DDL critique

```sql
-- Table principale dossiers (partitionnée par année pour archivage)
CREATE TABLE dossiers (
    dossier_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    external_ref        VARCHAR(100),
    title               VARCHAR(255) NOT NULL CHECK (length(title) >= 5),
    description         TEXT CHECK (length(description) <= 2000),
    type_dossier_id     UUID NOT NULL REFERENCES types_dossier(id),
    entity_id           UUID NOT NULL REFERENCES entities(id),
    classification      VARCHAR(50) NOT NULL
                        CHECK (classification IN ('PUBLIC','INTERNE','CONFIDENTIEL','SECRET')),
    urgence             VARCHAR(10) NOT NULL DEFAULT 'NORMAL'
                        CHECK (urgence IN ('NORMAL','URGENT')),
    amount              DECIMAL(18,2),
    currency            VARCHAR(3) CHECK (currency IN ('EUR','USD','GBP')),
    status              VARCHAR(30) NOT NULL DEFAULT 'BROUILLON'
                        CHECK (status IN ('BROUILLON','EN_ATTENTE_VALIDATION',
                                         'EN_MODIFICATION','EN_ESCALADE',
                                         'EN_SIGNATURE','SIGNÉ','ARCHIVÉ','ANNULÉ')),
    initiator_id        UUID NOT NULL REFERENCES users(id),
    circuit_id          UUID REFERENCES circuits(id),
    current_step        INTEGER DEFAULT 0,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    submitted_at        TIMESTAMPTZ,
    signed_at           TIMESTAMPTZ,
    archived_at         TIMESTAMPTZ,
    retention_until     TIMESTAMPTZ NOT NULL,  -- Calculé selon classification
    metadata            JSONB,                  -- Champs custom selon type_dossier
    CONSTRAINT chk_dates CHECK (created_at <= updated_at)
) PARTITION BY RANGE (created_at);

CREATE TABLE dossiers_2026 PARTITION OF dossiers
    FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');

-- Index optimisés
CREATE INDEX idx_dossiers_initiator_status ON dossiers(initiator_id, status)
    WHERE status NOT IN ('ARCHIVÉ','ANNULÉ');
CREATE INDEX idx_dossiers_entity_status ON dossiers(entity_id, status);
CREATE INDEX idx_dossiers_external_ref ON dossiers(external_ref) WHERE external_ref IS NOT NULL;
CREATE INDEX idx_dossiers_metadata_gin ON dossiers USING GIN (metadata);

-- Table audit_trail (append-only, immutable)
CREATE TABLE audit_trail (
    audit_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    dossier_id          UUID NOT NULL,
    actor_id            UUID NOT NULL REFERENCES users(id),
    action_type         VARCHAR(50) NOT NULL,
    action_details      JSONB NOT NULL,
    ip_address          INET,
    user_agent          TEXT,
    session_id          VARCHAR(100),
    timestamp_utc       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    previous_hash       CHAR(64),                  -- SHA-256 entrée précédente
    current_hash        CHAR(64) NOT NULL,         -- Hash chaîné (intégrité)
    tsa_token           BYTEA                      -- Optionnel : timestamp tiers
);

-- Trigger empêche UPDATE/DELETE sur audit_trail
CREATE OR REPLACE FUNCTION prevent_audit_modification()
RETURNS TRIGGER AS $$
BEGIN
    RAISE EXCEPTION 'audit_trail is immutable — no UPDATE/DELETE allowed';
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_immutable_trigger
    BEFORE UPDATE OR DELETE ON audit_trail
    FOR EACH ROW EXECUTE FUNCTION prevent_audit_modification();
```

### B.6 Sécurité — Threat model STRIDE

| Catégorie | Menace identifiée | Probabilité | Impact | Mitigation |
|-----------|-------------------|-------------|--------|------------|
| **S**poofing | Usurpation d'identité signataire | 3 | 5 | SSO + MFA obligatoire signataires, vérif. cert PKI |
| **S**poofing | Phishing utilisateur (faux email parapheur) | 4 | 4 | DKIM/SPF/DMARC, formation users, lien signé |
| **T**ampering | Modification document après validation | 2 | 5 | Hash SHA-256 verrouillé, scellement post-sig |
| **T**ampering | Modification audit trail | 2 | 5 | Append-only DB + chaînage hashes + TSA |
| **R**epudiation | Signataire nie avoir signé | 3 | 5 | Sig. qualifiée eIDAS, audit IP/UA/timestamp |
| **I**nformation Disclosure | Fuite docs sensibles via API | 3 | 5 | RBAC strict, chiffrement E2E S3+KMS, DLP |
| **I**nformation Disclosure | Logs contenant PII | 4 | 3 | Masking centralisé, scrubbing pre-Datadog |
| **D**enial of Service | Attaque sur API publique | 3 | 4 | Rate limiting (Kong), WAF (Cloudflare), CDN |
| **E**levation of Privilege | Compte admin compromis | 2 | 5 | MFA mandatoire admins, principle of least privilege, audit accès admin renforcé |

### B.7 Performance — Objectifs et dimensionnement

| Métrique | Cible | Méthode mesure | Outils |
|----------|-------|----------------|--------|
| Temps réponse API (p50) | < 200 ms | APM | Datadog |
| Temps réponse API (p95) | < 500 ms | APM | Datadog |
| Temps réponse API (p99) | < 1 200 ms | APM | Datadog |
| Throughput création dossiers | 50/s en pic | Load test | k6 / Gatling |
| Throughput signatures | 20/s en pic | Load test | k6 |
| Disponibilité (SLA) | 99,5 % (43h indispo/an) | Synthetic monitoring | Datadog Synthetics |
| RTO (Recovery Time Obj.) | 4 heures | DR drill trimestriel | Runbook + tests |
| RPO (Recovery Point Obj.) | 15 minutes | Replication PostgreSQL | Streaming replication |
| Capacité stockage initiale | 5 TB | Volumétrie 250K dossiers × 20MB | S3 lifecycle |
| Croissance stockage | +1,5 TB/an | Estimation projection | Capacity planning |

#### Dimensionnement infra (an 1)

| Composant | Configuration | Coût mensuel |
|-----------|---------------|--------------|
| K8s cluster (EKS) | 6 nodes m5.xlarge, 3 AZ | 1 800 € |
| PostgreSQL HA | RDS r5.2xlarge + replica | 1 400 € |
| Redis Cluster | 3 nodes cache.r5.large | 450 € |
| Kafka MSK | 3 brokers kafka.m5.large | 600 € |
| Elasticsearch | 3 nodes m5.large | 750 € |
| S3 + KMS + Backup | 5 TB + reqs + KMS | 320 € |
| Universign (sig. qualifiée) | Pack 50 K signatures/an | 2 100 € |
| Datadog APM + Logs + Synthetics | 10 hosts + 50 GB logs | 950 € |
| Kong Enterprise | License | 1 200 € |
| WAF Cloudflare Enterprise | License | 1 800 € |
| **TOTAL mensuel** | | **11 370 €** |
| **TOTAL an 1** | | **136 K€** |

---

## C. SPÉCIFICATIONS UI/UX

### C.1 Design System — Fondations

| Élément | Valeur | Source |
|---------|--------|--------|
| Framework UI | Material UI v5 customisé | React |
| Couleur primaire | #1E3A8A (Bleu corporate) | Charte entreprise |
| Couleur secondaire | #F59E0B (Ambre — actions) | Charte entreprise |
| Couleur succès | #10B981 (Vert) | Standard |
| Couleur erreur | #EF4444 (Rouge) | Standard |
| Couleur warning | #F59E0B | Standard |
| Typographie | Inter (web), system-ui (fallback) | Open source |
| Tailles texte | 12, 14, 16, 18, 24, 32, 48 px | Échelle 8pt |
| Espacement | Multiples de 8 px (8, 16, 24, 32) | Grille 8pt |
| Border radius | 4 / 8 / 12 px | Doux |
| Shadow | 5 niveaux d'élévation | Material |
| Breakpoints responsive | 640 / 768 / 1024 / 1280 / 1536 px | Tailwind defaults |

### C.2 Inventaire écrans — 23 écrans principaux

| ID | Écran | Rôle ciblé | Priorité | Complexité UI |
|----|-------|-----------|----------|---------------|
| SCR-001 | Login (SSO) | Tous | Must | Faible |
| SCR-002 | Dashboard Initiateur | Initiateur | Must | Élevée |
| SCR-003 | Dashboard Validateur | Validateur | Must | Élevée |
| SCR-004 | Dashboard Signataire | Signataire | Must | Moyenne |
| SCR-005 | Dashboard Admin | Admin | Must | Élevée |
| SCR-006 | Création dossier — Étape 1 (métadonnées) | Initiateur | Must | Moyenne |
| SCR-007 | Création dossier — Étape 2 (documents) | Initiateur | Must | Élevée (DnD) |
| SCR-008 | Création dossier — Étape 3 (circuit) | Initiateur | Must | Élevée (visuel) |
| SCR-009 | Détail dossier (vue complète) | Tous | Must | Élevée |
| SCR-010 | Visualiseur PDF (annotations) | Validateur, Signat. | Must | Très élevée |
| SCR-011 | Modal validation (Approve/Reject) | Validateur | Must | Faible |
| SCR-012 | Modal signature simple (OTP) | Signataire | Must | Moyenne |
| SCR-013 | Modal signature avancée | Signataire | Must | Moyenne |
| SCR-014 | Redirection sig. qualifiée | Signataire | Must | Faible |
| SCR-015 | Confirmation signature | Signataire | Must | Faible |
| SCR-016 | Mes tâches (liste filtrable) | Tous | Must | Moyenne |
| SCR-017 | Recherche dossiers | Tous | Should | Moyenne |
| SCR-018 | Audit trail détaillé | Auditeur, Admin | Must | Moyenne |
| SCR-019 | Paramètres utilisateur | Tous | Should | Faible |
| SCR-020 | Configuration délégation | Tous | Must | Moyenne |
| SCR-021 | Admin — Gestion users/rôles | Admin | Must | Élevée |
| SCR-022 | Admin — Configurateur circuit | Admin | Must | Très élevée |
| SCR-023 | Admin — Tableau de bord KPI | Admin, Direction | Should | Élevée |

### C.3 User Journey type — Initiateur (de la création à l'archivage)

```
┌──────────────────────────────────────────────────────────────────┐
│             PARCOURS INITIATEUR — Achat 8 500 €                  │
├──────────────────────────────────────────────────────────────────┤
│
│ Persona : Marie, Manager IT, 38 ans, Filiale FR
│ Objectif : Faire valider un achat de serveurs (8 500 €)
│ Contexte : Reçoit devis fournisseur le matin
│
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 1 — DÉCOUVERTE BESOIN                                 │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Action  : Ouvre email fournisseur (devis serveurs 8 500 €) │
│ │ Pensée  : "Il faut que je fasse valider rapidement"        │
│ │ Émotion : Neutre                                           │
│ │ Pain    : Procédure papier complexe (avant)                │
│ └─────────────────────────────────────────────────────────────┘
│                            ▼
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 2 — ACCÈS PARAPHEUR                                   │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Action  : Clique sur "Parapheur" dans portail intranet     │
│ │ Système : SSO Okta → redirection automatique               │
│ │ Écran   : SCR-002 Dashboard Initiateur                     │
│ │ Pensée  : "Je vois mes 3 dossiers en cours, je crée"      │
│ │ Émotion : Confiance                                        │
│ │ Temps   : 5 secondes                                       │
│ └─────────────────────────────────────────────────────────────┘
│                            ▼
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 3 — CRÉATION DOSSIER                                  │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Action  : Clique "+ Nouveau dossier"                       │
│ │ Écran   : SCR-006 Création — Métadonnées                   │
│ │ Saisie  : Type=Achat, Titre=, Montant=8500, Devise=EUR    │
│ │ Système : Suggère circuit "STANDARD" (RG-W-002 : 5K-50K)   │
│ │ Pensée  : "L'auto-suggestion m'aide bien"                  │
│ │ Émotion : Satisfaction                                     │
│ │ Temps   : 3 minutes                                        │
│ └─────────────────────────────────────────────────────────────┘
│                            ▼
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 4 — UPLOAD DOCUMENTS                                  │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Action  : Drag & drop devis PDF + bon de commande          │
│ │ Écran   : SCR-007 Upload                                   │
│ │ Système : Scan AV (300ms), génère thumbnails (1s)          │
│ │ Pensée  : "C'est rapide, j'ai même la prévisualisation"    │
│ │ Émotion : Satisfaction                                     │
│ │ Temps   : 1 minute                                         │
│ └─────────────────────────────────────────────────────────────┘
│                            ▼
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 5 — VALIDATION CIRCUIT                                │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Action  : Visualise circuit suggéré, valide                │
│ │ Écran   : SCR-008 Circuit (BPMN visuel)                    │
│ │ Système : Affiche [Manager → Finance ∥ Legal → Director]   │
│ │ Pensée  : "Je vois clairement qui doit valider"            │
│ │ Émotion : Transparence                                     │
│ │ Temps   : 30 secondes                                      │
│ └─────────────────────────────────────────────────────────────┘
│                            ▼
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 6 — SOUMISSION                                        │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Action  : Clique "Soumettre"                               │
│ │ Système : Crée dossier, lance workflow, notifie Manager    │
│ │ Toast   : "Dossier soumis ✓ Suivi disponible"              │
│ │ Pensée  : "C'est parti, je peux suivre"                    │
│ │ Émotion : Accomplissement                                  │
│ │ Temps   : 2 secondes                                       │
│ └─────────────────────────────────────────────────────────────┘
│                            ▼
│ ┌─────────────────────────────────────────────────────────────┐
│ │ ÉTAPE 7 — ATTENTE & SUIVI (J+1 à J+5)                      │
│ ├─────────────────────────────────────────────────────────────┤
│ │ Notif   : Email J+1 — Manager a validé                     │
│ │ Notif   : Email J+3 — Finance + Legal validés en parallèle │
│ │ Notif   : Email J+4 — Director a validé                    │
│ │ Notif   : Email J+5 — Document signé par PDG               │
│ │ Email   : Document signé en PJ + lien archivage GED        │
│ │ Pensée  : "Très fluide, beaucoup mieux qu'avant"           │
│ │ Émotion : Satisfaction élevée                              │
│ │ Temps total : 5 jours (vs 18 jours en mode papier)         │
│ └─────────────────────────────────────────────────────────────┘
│
│ MÉTRIQUES PARCOURS
│ ──────────────────
│ - Durée création (E1-E6)     : 5 minutes
│ - Durée totale (E1-E7)       : 5 jours
│ - Nb clics                   : 12 (vs 5 cibles, à optimiser)
│ - Taux d'erreur saisie       : < 5 %
│ - Score satisfaction (SUS)   : > 80/100 (cible)
│
│ POINTS DE FRICTION IDENTIFIÉS
│ ─────────────────────────────
│ F1 : 12 clics au lieu de 5 cibles → simplifier étape 5 (auto-confirm circuit)
│ F2 : Pas d'aperçu fournisseur connu → autocomplete depuis SAP master data
│ F3 : Pas de templates achats récurrents → fonction "Cloner dossier" (US-D1-007)
│
└──────────────────────────────────────────────────────────────────┘
```

### C.4 Plan de tests utilisateurs

| Phase | Méthode | Participants | Livrable | Budget |
|-------|---------|--------------|----------|--------|
| Cadrage | Interviews semi-directives | 8 (2 par rôle × 4 rôles) | Personas finalisés + parcours cibles | 4 K€ |
| Wireframes | Tests d'arborescence + tri de cartes | 12 | IA optimisée (validation 75 % succès) | 3 K€ |
| Mockups | Tests modérés Figma | 18 (6 par rôle × 3 rôles) | 12 améliorations UX validées | 8 K€ |
| Prototype | Tests d'usabilité (5 tâches) | 30 (10 par entité × 3 entités) | Score SUS > 80, taux complétion > 90 % | 12 K€ |
| Pilot | A/B test post-MVP | 50 utilisateurs réels 4 sem | Taux adoption + retours qualitatifs | 5 K€ |
| **TOTAL** | | **118 sessions** | | **32 K€** |

### C.5 Accessibilité — Conformité WCAG 2.1 AA

| Critère WCAG | Niveau | Mesure de conformité |
|--------------|--------|---------------------|
| 1.1.1 Texte alternatif images | A | Tous les `<img>` ont `alt` non vide |
| 1.3.1 Information et relations | A | HTML sémantique, ARIA roles |
| 1.4.3 Contraste minimum | AA | Ratio ≥ 4,5:1 sur tout le texte |
| 1.4.5 Texte sous forme d'image | AA | Aucun (sauf logo) |
| 2.1.1 Navigation clavier | A | 100 % accessible au clavier |
| 2.4.3 Parcours du focus | A | Focus visible + ordre logique |
| 2.4.7 Visibilité du focus | AA | Outline 2px min, contraste élevé |
| 3.1.1 Langue de la page | A | `<html lang="fr">` |
| 3.3.1 Identification des erreurs | A | Messages explicites + ARIA-live |
| 3.3.3 Suggestion d'erreur | AA | Suggestion correction proposée |
| 4.1.2 Nom, rôle, valeur | A | ARIA labels sur tous les composants custom |

**Audit prévu** : Audit Lighthouse + tests manuels NVDA/JAWS + 2 utilisateurs malvoyants en panel test.

---

## D. RISK REGISTER PHASE 1

### D.1 Risques identifiés (12 risques)

| ID | Risque | P (1-5) | I (1-5) | Score | Owner | Mitigation |
|----|--------|---------|---------|-------|-------|------------|
| R1 | Indisponibilité experts métier | 4 | 5 | **20** | DAF/Sponsor | Engagement formel 2j/sem dès J0 |
| R2 | Scope creep (+30 %) | 4 | 4 | **16** | PM | Gel scope J+15, comité changement |
| R3 | Décisions architecturales tardives | 3 | 5 | **15** | DSI | COPIL J+5 dédié, escalade ComEx |
| R4 | Sous-estimation effort UX | 4 | 3 | **12** | UX Lead | Recruter UX senior dès J0 |
| R5 | Mauvaise qualité specs ERP | 3 | 4 | **12** | Architecte | Atelier dédié SAP J+3 |
| R6 | Conformité eIDAS mal cadrée | 2 | 5 | **10** | Legal/RSSI | Expert eIDAS externe (2 j/sem) |
| R7 | Volumétrie réelle > prévue | 2 | 5 | **10** | Architecte | Stress test sur hypothèses x2 |
| R8 | Choix Universign vs concurrent | 2 | 4 | **8** | Achats | RFP formelle 3 vendors avant J+30 |
| R9 | Désalignement sponsor | 2 | 5 | **10** | PM | COPIL hebdo + reporting structuré |
| R10 | Recrutement BA/Architect en retard | 3 | 3 | **9** | RH | Vivier interne identifié J0 |
| R11 | Évolution réglementaire (RGPD/eIDAS) | 1 | 4 | **4** | Legal | Veille mensuelle |
| R12 | Outils Figma/Confluence indisponibles | 1 | 2 | **2** | IT | Alternatives prêtes |

### D.2 Actions préventives prioritaires (top 5)

1. **J0** : Engagement formel sponsor + experts métier (lettre de mission)
2. **J+5** : COPIL décisions architecturales (D1-D2-D3)
3. **J+15** : Gel du scope fonctionnel + comité de changement instauré
4. **J+30** : RFP formelle service signature qualifiée (3 vendors)
5. **J+45** : Tests utilisabilité prototype Figma (30 users)

---

## E. PLAN DE PRODUCTION & JALONS

### E.1 Planning Phase 1 — 8 semaines

```
Semaine    │ S1  │ S2  │ S3  │ S4  │ S5  │ S6  │ S7  │ S8  │
───────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
Cadrage    │ ███ │ ██  │     │     │     │     │     │     │
SFD        │ ██  │ ███ │ ███ │ ███ │ ██  │ █   │     │     │
TS         │     │ █   │ ██  │ ███ │ ███ │ ███ │ ██  │ █   │
UI/UX      │ ██  │ ███ │ ███ │ ███ │ ███ │ ██  │ █   │     │
Tests UX   │     │     │     │ ██  │ ██  │ ███ │ ██  │     │
Reviews    │     │     │ █   │     │ █   │     │ █   │ ███ │
COPIL      │  ●  │     │     │  ●  │     │     │  ●  │  ●  │
───────────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

● = COPIL (J+5, J+25, J+45, J+55 final)
```

### E.2 Livrables et jalons

| Jalon | Date | Livrable | Validation |
|-------|------|----------|------------|
| J+5 | 2026-04-29 | Décisions architecturales D1/D2/D3 | COPIL |
| J+15 | 2026-05-09 | Gel scope + 47 US identifiées (titre+narrative) | COPIL |
| J+25 | 2026-05-19 | 47 US détaillées + 35 RG documentées | PO + Métier |
| J+30 | 2026-05-24 | RFP signature qualifiée envoyée | Achats |
| J+35 | 2026-05-29 | Mockups HD Figma (23 écrans) | UX + Métier |
| J+45 | 2026-06-08 | Tests usabilité 30 users complétés | UX |
| J+50 | 2026-06-13 | Architecture C4 + DDL + OpenAPI complets | DSI |
| J+55 | 2026-06-18 | Phase 1 finalisée — GO/NO-GO développement | **COPIL Direction** |
| J+57 | 2026-06-22 | **Démarrage Phase 2 — Développement** | — |

### E.3 Équipe et budget

| Rôle | ETP | Coût j/h | Jours | Total |
|------|-----|----------|-------|-------|
| Senior Manager (sponsor) | 0,2 | 1 200 € | 8 | 9,6 K€ |
| Project Manager | 1,0 | 850 € | 40 | 34 K€ |
| Architecte solution | 0,8 | 950 € | 32 | 30,4 K€ |
| Business Analyst senior | 1,5 | 750 € | 60 | 45 K€ |
| UX/UI Designer senior | 1,0 | 850 € | 40 | 34 K€ |
| Expert sécurité/eIDAS | 0,3 | 1 100 € | 12 | 13,2 K€ |
| Expert RGPD/Legal | 0,2 | 1 100 € | 8 | 8,8 K€ |
| Tests utilisabilité (externe) | — | forfait | — | 32 K€ |
| Outils (Figma, Confluence, Jira) | — | forfait | — | 8 K€ |
| Buffer (10 %) | — | — | — | 25 K€ |
| **TOTAL Phase 1 (8 semaines)** | | | **240 j** | **240 K€** |

### E.4 Critères de GO/NO-GO Phase 2

| Critère | Cible | Pondération |
|---------|-------|-------------|
| 47 US Must détaillées avec AC | 100 % | 25 % |
| Architecture validée DSI/RSSI | OUI | 20 % |
| Maquettes HD validées 30 users | Score SUS > 80 | 15 % |
| Décisions D1/D2/D3 prises | OUI | 15 % |
| Budget Phase 2 confirmé | OUI | 10 % |
| Équipe Phase 2 staffée | 100 % postes pourvus | 10 % |
| Risques top 5 mitigés | Score moyen < 12 | 5 % |

**Seuil GO** : Score pondéré ≥ 85 % sur tous les critères.
**Seuil NO-GO** : Tout critère < 70 % → escalade ComEx pour décision.

---

## ANNEXES

- **Annexe A** : Backlog complet 47 US (fichier Excel séparé)
- **Annexe B** : Spécifications API OpenAPI 3.1 (fichier YAML séparé)
- **Annexe C** : Modèle de données complet — DDL PostgreSQL (fichier SQL séparé)
- **Annexe D** : Maquettes Figma (lien Figma)
- **Annexe E** : Threat model complet — STRIDE + DREAD (fichier séparé)
- **Annexe F** : Plan de tests utilisabilité détaillé (fichier séparé)

---

**Document soumis à validation COPIL du 2026-04-29.**

**Signataires requis** :
- DAF (Sponsor financier)
- DSI (Sponsor technique)
- Direction Métier (Sponsor fonctionnel)
- RSSI (Validation sécurité)
- DPO (Validation RGPD)
