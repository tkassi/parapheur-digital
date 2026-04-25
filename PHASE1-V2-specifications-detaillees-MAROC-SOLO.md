# Phase 1 v2 — Spécifications détaillées

**Programme Parapheur Digital — Banque commerciale universelle marocaine cotée à la Bourse de Casablanca, classée OIV**

| Métadonnée | Valeur |
|---|---|
| Version | R2.0 — Avril 2026 |
| Auteur | Direction de programme |
| Statut | Pour validation Direction métier + DSI + UX/UI |
| Documents amendés | Phase 1 v1 (47 user stories, eIDAS QES, Okta) |
| Confidentialité | Interne / Direction |
| Pages | ~80 |

---

## Note de cadrage

Cette Phase 1 v2 remplace la Phase 1 v1 pour le contexte d'une **banque marocaine cotée OIV avec parapheur en mode solo + Claude**.

**Ce qui change vs v1** :

| Domaine | v1 (UE) | v2 (Maroc / banque) |
|---|---|---|
| Authentification | Okta SSO + WebAuthn FIDO2 | **AD banque + MFA OTP** existant |
| Devise affichée | EUR par défaut | **MAD pivot**, format `1 234 567,89 MAD` |
| Langues UI | FR + EN | **FR + AR (RTL)** |
| Calendrier | Calendrier UE | **Calendrier Maroc** (jours fériés, week-end sam-dim) |
| Documents | UE génériques | **Maroc** : CIN/CNIE, ICE, IF, RC, CNSS |
| Niveau signature | QES eIDAS pour > 50 K€ | **SEA Loi 43-20** par défaut, SEQ optionnelle > 5 M MAD |
| Format date | dd/mm/yyyy | dd/mm/yyyy (FR) ; arabe : même format ou indo-arabes selon réglage user |
| Wireframes | Figma équipe UX | **Wireframes textuels** (description ASCII) — solo, validation users en P9 |

**Le périmètre des 47 user stories reste valide** ; les modifications portent sur l'ergonomie (i18n RTL), les libellés, les types de documents, et l'authentification.

---

## Sommaire

1. Cadrage produit
2. Personas
3. Parcours utilisateurs principaux
4. Catalogue des 47 user stories
5. Wireframes textuels (écrans clés)
6. Internationalisation FR + AR (RTL)
7. Accessibilité WCAG 2.1 AA + RTL
8. Critères d'acceptation transverses
9. Annexes

---

## 1. Cadrage produit

### 1.1 Vision

> Donner à chaque collaborateur de la banque un **parapheur digital simple, rapide et conforme**, accessible 24/7 depuis n'importe quel poste banque ou mobile, en français ou en arabe, pour soumettre, valider, signer et archiver les documents internes sans délai et avec traçabilité totale.

### 1.2 Public cible

| Profil | Volume utilisateurs | Périphériques cibles | Compétence numérique |
|---|---|---|---|
| Émetteurs (chargés clientèle, opérationnels) | ~800 | Desktop banque + mobile pro | Moyenne |
| Valideurs N5-N4 (responsables agence/dept) | ~280 | Desktop + mobile pro | Moyenne-élevée |
| Valideurs N3-N2 (directeurs centraux/régionaux) | ~40 | Desktop + mobile pro + PWA tablette | Élevée (souvent en déplacement) |
| DG / DGA / Comités | ~10 | Desktop + tablette + mobile | Moyenne (assistance fréquente) |
| Audit / Inspection / Conformité | ~50 | Desktop | Élevée |

**Total** : ~1 200 comptes actifs.

### 1.3 Contraintes principales

- **Conformité** : Loi 09-08, 43-20, 05-20, circulaires BAM, DNSSI, AMMC (cf. P3 v2)
- **Performance** : P95 < 500 ms (interface), pic 200 req/min
- **Disponibilité** : 99,5 % en heures ouvrées
- **Mobile** : PWA installable, fonctionnement offline en lecture (consultation dossiers en attente)
- **Multilingue** : FR (par défaut) + AR (RTL)
- **Accessibilité** : WCAG 2.1 niveau AA + tests RTL arabe
- **Hébergement** : Maroc (cloud banque ou DC Maroc)

---

## 2. Personas

### 2.1 Persona 1 — Khalid, chargé de clientèle PRO (émetteur)

| Attribut | Valeur |
|---|---|
| Âge | 32 ans |
| Direction | Banque commerciale, agence Casablanca centre |
| Rôle parapheur | Émetteur (~30 dossiers/mois) |
| Compétence numérique | Moyenne |
| Langues utilisées | Français principalement, arabe pour clientèle locale |
| Périphérique | Desktop banque + smartphone pro |
| Frustrations actuelles (papier) | Délais, perte courriers, attente signatures, déplacements bureaux signataires |
| Besoins | Soumettre rapide, voir statut en direct, recevoir alertes, relancer un valideur si bloqué |

### 2.2 Persona 2 — Aïcha, directrice de département Crédit (valideur N4)

| Attribut | Valeur |
|---|---|
| Âge | 45 ans |
| Direction | Direction Crédit, siège |
| Rôle parapheur | Valideur N4 (~150 décisions/mois) |
| Compétence numérique | Élevée |
| Langues utilisées | Français + anglais (réunions internationales) |
| Périphérique | Desktop + tablette (réunions) + smartphone |
| Frustrations actuelles | Signatures à la chaîne, files d'attente parapheur papier, retours pour info manquante |
| Besoins | File d'attente claire, info synthétisée, validation par lot quand possible, signature mobile pour urgences |

### 2.3 Persona 3 — Mohammed, DGA Banque commerciale (valideur N2 + Comité)

| Attribut | Valeur |
|---|---|
| Âge | 56 ans |
| Direction | Direction Générale |
| Rôle parapheur | Valideur N2 + membre Comité de Crédit (~80 décisions/mois) |
| Compétence numérique | Moyenne (assistante l'aide souvent) |
| Langues utilisées | Français + arabe (CA, communications officielles) |
| Périphérique | Desktop + tablette (déplacements, voyages) |
| Frustrations actuelles | Décisions urgentes en déplacement, signature physique impossible |
| Besoins | Application mobile, notifications push, vue de synthèse, signature simple en quelques clics |

### 2.4 Persona 4 — Fatima, responsable RH (valideur N5)

| Attribut | Valeur |
|---|---|
| Âge | 38 ans |
| Direction | Ressources Humaines |
| Rôle parapheur | Valideur N5 + émettrice (recrutements, mutations) |
| Compétence numérique | Moyenne |
| Langues utilisées | Français + arabe |
| Périphérique | Desktop |
| Besoins | Workflow recrutement multi-étapes, intégration AD pour identification candidat existant, conservation documents RH |

### 2.5 Persona 5 — Yassir, auditeur interne (consultation)

| Attribut | Valeur |
|---|---|
| Âge | 41 ans |
| Direction | Inspection Générale |
| Rôle parapheur | Consultation lecture + extraction journal (audit, contrôle a posteriori) |
| Compétence numérique | Élevée |
| Langues utilisées | Français + arabe + anglais |
| Périphérique | Desktop |
| Besoins | Recherche multi-critères, export CSV/PDF, reconstitution complète historique d'un dossier, accès journal d'audit |

### 2.6 Persona 6 — Rachid, RSSI (administrateur)

| Attribut | Valeur |
|---|---|
| Âge | 49 ans |
| Direction | DSI / Sécurité |
| Rôle parapheur | Administrateur sécurité (config, audit logs, gestion incidents) |
| Compétence numérique | Très élevée |
| Périphérique | Desktop |
| Besoins | Tableau de bord sécurité, alertes, rotation secrets, audit trail détaillé |

---

## 3. Parcours utilisateurs principaux

### 3.1 Parcours P1 — Soumission engagement crédit (Khalid → Aïcha → Mohammed)

```
J0 - 09:00  Khalid se connecte (AD + OTP)
J0 - 09:05  Crée nouveau dossier "Engagement crédit ABC SARL — 800 K MAD"
            Saisit type, montant, devise, contrepartie (lookup core banking)
            Charge documents requis : RC, ICE, IF, attestation fiscale, business plan
J0 - 09:15  Soumet le dossier au workflow
            Le système évalue les règles DMN-R-001 :
            → 800 K MAD = niveau N5 + N4
            → assigne valideur N5 (responsable agence) puis N4 (Aïcha)
J0 - 09:16  Notification email + SMS au responsable agence

J0 - 11:00  Responsable agence reçoit notif, ouvre le dossier sur mobile
            Examine les documents, valide → SES (signature simple)
            Auto-passage au valideur suivant (Aïcha)

J0 - 14:30  Aïcha reçoit notif sur tablette en réunion
            Ouvre dossier, examine pièces, demande info complémentaire
            (commentaire à Khalid)
J0 - 15:00  Khalid voit demande, ajoute pièce manquante
J0 - 15:05  Aïcha valide → SEA (signature avancée via plateforme e-sign)
            Plateforme e-sign banque envoie SMS à Aïcha pour confirmation OTP
            Signature appliquée
J0 - 15:30  Workflow passe à signature finale (selon règles, le dossier est complet)
            Document signé envoyé à GED
J0 - 15:40  Archivage GED confirmé
            Notification finale à Khalid + contrepartie info (interne)

Cycle total : 6h ouvrées (vs ~8 jours en mode papier)
```

### 3.2 Parcours P2 — Validation comité de crédit > 5 M MAD

```
Émetteur soumet dossier 12 M MAD
→ DMN-R-001 : valideurs_chain = [N3, N2, COMITE_CREDIT]
              signature_level = SEQ optionnelle (sinon SEA double)

J1   N3 valide (SEA)
J2   N2 valide (SEA)
J3   Dossier inscrit à l'ordre du jour Comité Crédit prochaine session
J5   Comité Crédit délibère → décision collégiale
     Chaque membre signe (SEA + multi-signataires) 
     OU signature unique du président + PV signé par tous
J5+1 Archivage
```

### 3.3 Parcours P3 — Validation RH recrutement cadre (Fatima → ...)

```
Fatima crée dossier "Recrutement directeur agence Rabat"
Documents : CIN/CNIE, diplômes, CV, références, casier judiciaire récent
DMN-R-004 : valideurs_chain = [Direction RH, DGA banque commerciale]
J0   Fatima soumet
J1   Direction RH valide (SEA)
J2   DGA Banque commerciale valide (SEA)
     Notification automatique à Direction Juridique (rédaction contrat)
J3   Archivage GED + dossier RH électronique
```

### 3.4 Parcours P4 — Délégation pour vacances

```
Aïcha part en congés du 15 au 31 juillet
Crée dossier "Délégation vacances → Karima (chef adjoint Crédit)"
Soumet
DMN-R-015 : auto-validation par DRH (workflow simplifié)
Activation automatique le 15/07 à 00:00
Désactivation automatique le 01/08 à 00:00
Notifications : Aïcha, Karima, équipe Crédit, RSSI (audit)
Tous workflows arrivés pendant la période sont assignés à Karima au lieu d'Aïcha
```

### 3.5 Parcours P5 — Audit a posteriori (Yassir)

```
Yassir reçoit demande de l'AMMC : extraction des engagements > 5 M MAD signés au T1 2026
J0   Yassir se connecte (AD + OTP) — rôle "auditeur"
     Lance recherche : période = 01/01-31/03/2026, type = engagement-credit, montant >= 5M
     → 187 résultats
J0   Pour chaque dossier, clique "Reconstitution historique"
     → Affichage chronologique : création, soumission, chaque validation, signature, archivage
     → Liens vers documents originaux + signés (téléchargement chiffré)
J1   Export CSV global (en-tête : reference, type, montant, devise, valideurs, dates, hash docs)
     + Export PDF synthétique (10 pages) pour transmission AMMC
     Action tracée en MS-4 (export = événement CRITICAL)
```

---

## 4. Catalogue des 47 user stories

**Format** :
- ID + titre
- Persona + As/I want/So that
- Critères d'acceptation (CA)
- Effort (S/M/L)

### Épopée A — Authentification et identité (4 US)

#### US-01 — Connexion via AD + MFA
**Persona** : Tous
**As a** collaborateur banque
**I want** me connecter avec mon compte AD habituel + OTP
**So that** je n'ai pas à mémoriser un nouveau mot de passe et la sécurité est garantie
**CA** :
- Saisie matricule + mot de passe AD
- Vérification AD/LDAP banque (LDAPS)
- Demande OTP (SMS / app authenticator selon config user)
- En cas d'échec MFA : 5 tentatives puis verrouillage 30 min + alerte SOC
- Journalisation MS-4 (succès, échec)
- Session JWT 8h, refresh token 30j
- Bouton « Déconnexion » termine la session côté backend (Redis revoke)
**Effort** : M

#### US-02 — Délégation expresse (vacances)
**Persona** : Aïcha
**As a** valideur
**I want** désigner un suppléant pendant mes vacances
**So that** mes dossiers ne restent pas bloqués
**CA** :
- Saisie : suppléant (lookup AD), date début, date fin, motif
- Validation par DRH (workflow auto)
- Activation automatique à date début 00:00
- Désactivation automatique à date fin 00:00
- Visualisation des délégations actives par tous concernés
- Journalisation MS-4
**Effort** : M

#### US-03 — Profil utilisateur
**As a** utilisateur
**I want** consulter et ajuster mon profil (langue, fuseau, notifications)
**CA** :
- Lecture seule : matricule, nom, direction, agence, niveau, groupes (issus AD, non éditables)
- Édition : langue UI (FR/AR), fuseau horaire, préférences notifications (email/SMS/push), affichage dashboard par défaut
- Sauvegarde immédiate
**Effort** : S

#### US-04 — Visualisation rôles et délégations actives
**As a** utilisateur
**I want** voir mes rôles, périmètres et délégations en cours
**CA** :
- Affichage rôles RBAC issus AD
- Liste délégations en cours (reçues + données)
- Filtrage par type
**Effort** : S

### Épopée B — Création et gestion des dossiers (8 US)

#### US-05 — Créer un nouveau dossier
**Persona** : Khalid
**CA** :
- Wizard étape 1 : choix type décision (engagement crédit, dépense, RH, etc.)
- Étape 2 : champs spécifiques au type (montant + devise, contrepartie, segment, période)
- Étape 3 : upload documents requis (DMN-DOC affiche la liste obligatoire dynamique)
- Étape 4 : récapitulatif + bouton « Enregistrer brouillon » ou « Soumettre »
- Validation côté front + back (Zod schéma partagé)
- Génération automatique de la référence DOS-AAAA-NNNNNN
**Effort** : L

#### US-06 — Modifier un dossier en brouillon
**CA** :
- Possible uniquement en statut `draft`
- Tous les champs modifiables sauf type décision (qui implique recréer)
- Historique des modifications (MS-4)
**Effort** : S

#### US-07 — Soumettre un dossier
**CA** :
- Vérification complétude documents (selon DMN-DOC)
- Évaluation règles DMN-R (routage)
- Affichage prévisualisation chaîne de validation : « Le dossier sera envoyé à : Y, puis Z, puis W »
- Confirmation utilisateur
- Création workflow_instance, premier task assigné
**Effort** : M

#### US-08 — Annuler un dossier soumis (avant validation)
**CA** :
- Possible si statut `pending_validation` et aucun valideur n'a encore approuvé
- Confirmation requise
- Notification valideurs en attente
- Statut → `cancelled`
**Effort** : S

#### US-09 — Charger des documents
**CA** :
- Drag & drop ou bouton parcourir
- Formats acceptés : PDF, DOC/DOCX, XLS/XLSX, PNG, JPG
- Taille max 50 Mo par fichier
- Antivirus (ClamAV ou EDR banque) — délai max 30 s
- Hash SHA-256 calculé et stocké
- Aperçu PDF intégré (PDF.js)
- Renommage possible
- Suppression possible avant soumission
**Effort** : M

#### US-10 — Lookup contrepartie via core banking
**CA** :
- Champ recherche : ICE, raison sociale, ou ID interne
- Appel API core banking (cache Redis 24h)
- Affichage résultats (max 10) avec ICE, RC, segment, statut
- Sélection auto-remplit champs contrepartie
- Si non trouvé : option saisie manuelle (avec validation format ICE 15 chiffres)
**Effort** : M

#### US-11 — Dupliquer un dossier précédent
**CA** :
- Bouton « Dupliquer » sur dossier archivé ou rejeté
- Copie tous les champs sauf documents (à recharger)
- Statut nouveau dossier = `draft`
- Référence vers dossier source dans métadonnées
**Effort** : S

#### US-12 — Affichage montants multi-devises
**CA** :
- Saisie : montant + devise (MAD, EUR, USD, autre via dropdown)
- Affichage : montant en devise saisie + équivalent MAD entre parenthèses
- Conversion automatique au taux BAM jour (cache 1h)
- Format MAD : `1 234 567,89 MAD` (séparateur d'espace insécable, virgule décimale en FR)
- Format AR : conversion possible chiffres indo-arabes selon préférence user
**Effort** : M

### Épopée C — Workflow et validation (10 US)

#### US-13 — File d'attente de mes validations
**Persona** : Aïcha, Mohammed
**CA** :
- Liste paginée (20/page) des dossiers à valider, triés par priorité (urgence + ancienneté)
- Colonnes : référence, émetteur, type, montant MAD, soumis le, échéance, statut
- Filtres : type, montant, urgence, émetteur direction
- Recherche full-text
- Indicateurs visuels : urgent (rouge), proche échéance (orange), normal (gris)
- Compteur global notifié dans le menu (badge)
**Effort** : M

#### US-14 — Consulter le détail d'un dossier
**CA** :
- En-tête : référence, type, statut, montant, dates clés
- Bloc émetteur : nom, direction, contact
- Bloc contrepartie : ICE, raison sociale, lien vers core banking
- Bloc documents : liste, aperçu PDF intégré, téléchargement
- Bloc historique chronologique : créations, validations, modifications, signatures, escalades
- Bloc commentaires : fil de discussion
- Bloc actions : selon statut + rôle (valider, rejeter, demander info, déléguer, etc.)
**Effort** : L

#### US-15 — Valider un dossier (signature simple SES)
**CA** :
- Bouton « Valider » disponible si statut = `pending_validation` et user = current_assignee
- Modal : commentaire optionnel, confirmation
- Appel signature SES (login + mot de passe re-confirmé)
- Workflow avance vers étape suivante
- Notification émetteur + valideur suivant
- Journalisation MS-4
**Effort** : M

#### US-16 — Valider un dossier (signature avancée SEA)
**CA** :
- Bouton « Valider et signer SEA »
- Récapitulatif : « Vous allez signer ce document avec une signature électronique avancée (SEA) ayant force probante »
- Confirmation OTP envoyé par la plateforme e-sign banque
- Saisie OTP
- Appel API plateforme e-sign banque (cf. P4 v2 § 5.2)
- Réception callback (SIGNED / REJECTED / ERROR)
- Si SIGNED : workflow avance, document signé stocké, archivage GED déclenché
- Si ERROR : retry automatique 3 fois (60s, 5min, 15min) puis mode dégradé
**Effort** : L

#### US-17 — Valider un dossier (signature qualifiée SEQ — optionnelle)
**CA** :
- Disponible si engagement > 5 M MAD ET plateforme banque agréée DGSSI
- Récapitulatif : « Signature qualifiée — équivalente à signature manuscrite »
- Procédure d'identification renforcée (CNIE + double facteur fort)
- Délai possible plus long (min 2 min, max 10 min)
**Effort** : M

#### US-18 — Rejeter un dossier
**CA** :
- Bouton « Rejeter »
- Motif obligatoire (min 20 caractères)
- Confirmation
- Workflow → statut `rejected` (final)
- Notification émetteur + tous valideurs précédents
**Effort** : S

#### US-19 — Demander une information complémentaire
**CA** :
- Bouton « Demander info »
- Saisie message (min 10 caractères)
- Workflow reste en `pending_validation`, mais l'émetteur reçoit un ticket à traiter
- Émetteur peut répondre + ajouter pièces
- Une fois fait, le valideur reçoit notification de mise à jour
**Effort** : M

#### US-20 — Validation par lot (« batch »)
**Persona** : Aïcha (volumes élevés)
**CA** :
- Sélection multiple dans la file d'attente (checkboxes)
- Filtres pour pré-sélection rapide (ex. tous les dossiers RH N5 < 100 K MAD)
- Bouton « Valider sélection » (max 50 dossiers à la fois)
- Confirmation : « Vous allez valider 35 dossiers totalisant 1,8 M MAD »
- Pour chaque dossier : appel signature individuel (SES en lot, SEA si chacun signé OTP unique)
- Progress bar
- Récapitulatif : succès / échecs avec motif
**Effort** : L

#### US-21 — Déléguer un dossier ad hoc
**CA** :
- Bouton « Déléguer ce dossier »
- Choix utilisateur cible (lookup AD, niveau égal ou supérieur uniquement)
- Motif obligatoire
- Notification cible + DRH + audit
- Le dossier passe au cible (current_assignee modifié)
**Effort** : M

#### US-22 — Escalader un dossier
**CA** :
- Bouton « Escalader » (disponible si délai dépassé)
- Choix : escalade hiérarchique (N+1) ou Comité Crédit ou Inspection
- Motif
- Notification cible + journalisation MS-4
**Effort** : S

### Épopée D — Notifications et alertes (4 US)

#### US-23 — Notifications email
**CA** :
- Templates 18 cas × 2 langues (FR + AR)
- Email contient : sujet, corps, lien direct dossier (deep link), bouton CTA, footer signature banque
- Tracking ouvertures + clics (pixel + UTM)
- Préférences user : activer/désactiver par type de notification
**Effort** : M

#### US-24 — Notifications SMS (urgences uniquement)
**CA** :
- SMS via gateway interne banque
- Pour S1 : escalade urgente, expiration imminente
- Texte court avec lien raccourci
- Préférences user : activer/désactiver
**Effort** : S

#### US-25 — Notifications push (PWA)
**CA** :
- Web Push API (HTTPS + Service Worker)
- Permission demandée à l'installation PWA
- Push pour : nouvelle action requise, mise à jour de mes dossiers
**Effort** : M

#### US-26 — Centre de notifications
**CA** :
- Icône cloche avec badge compteur
- Liste des 50 dernières notifications (toutes catégories)
- Marquer comme lu (individuel + tout)
- Filtres : non lu, urgent, type
**Effort** : M

### Épopée E — Recherche et reporting (5 US)

#### US-27 — Recherche full-text
**CA** :
- Barre de recherche globale (Cmd+K / Ctrl+K)
- Recherche dans : référence, métadonnées, contrepartie, commentaires
- Backend : PostgreSQL `tsvector` (français + arabe stemming si plugin disponible)
- Résultats : 20 max, surlignement
- Filtres rapides : période, statut, type, mes dossiers/tous
**Effort** : M

#### US-28 — Recherche avancée
**CA** :
- Formulaire dédié : période, statut, type, montant min/max, devise, émetteur direction, valideur, contrepartie ICE
- Sauvegarde recherches favorites
- Export résultats CSV/PDF
**Effort** : M

#### US-29 — Tableau de bord personnel
**CA** :
- Cartes : « À valider (12) », « En attente de mes pièces (3) », « Mes dossiers en cours (8) », « Délais à risque (2) »
- Graphique : volume mes dossiers/mois
- Lien vers chaque section
**Effort** : M

#### US-30 — Tableau de bord direction
**Persona** : Aïcha (direction Crédit)
**CA** :
- Filtres : ma direction, mes équipes
- KPIs : volume soumis/validé/rejeté, cycle moyen, top types, taux respect SLA
- Graphiques temporels (mensuel)
- Export PDF pour COPIL trimestriel
**Effort** : L

#### US-31 — Reporting réglementaire (audit BAM/AMMC)
**Persona** : Yassir
**CA** :
- Accès rôle « auditeur » uniquement
- Filtres avancés : période, type, montant, valideurs, signataires
- Reconstitution chronologique complète (extraction MS-4)
- Export structuré CSV + PDF
- Action tracée comme événement CRITICAL
**Effort** : L

### Épopée F — Internationalisation (3 US)

#### US-32 — Bascule FR / AR
**CA** :
- Bouton dans header : icône globe + langue actuelle
- Sélection FR ou AR
- Recharge interface en langue choisie
- Direction CSS bascule LTR ↔ RTL automatiquement
- Sauvegarde préférence dans profil
**Effort** : M

#### US-33 — Affichage RTL pour arabe
**CA** :
- Layout miroir : menu à droite, contenu à gauche
- Icônes directionnelles (flèches) inversées
- Tableaux : colonnes inversées
- Tests manuels avec utilisateurs arabophones (P9)
**Effort** : L (couvert dans US-32 mais effort spécifique tests)

#### US-34 — Format des nombres et dates
**CA** :
- Dates : `dd/mm/yyyy` en FR, idem en AR (format hégirien optionnel pour AR)
- Nombres : séparateur d'espace insécable + virgule décimale en FR ; chiffres indo-arabes optionnels en AR
- Devise : `1 234 567,89 MAD` (FR) ou ١٬٢٣٤٬٥٦٧٫٨٩ درهم (AR avec chiffres indo-arabes — optionnel)
**Effort** : M

### Épopée G — Mobile et offline (3 US)

#### US-35 — Application PWA
**CA** :
- Manifest, Service Worker, icônes
- Installable sur Android / iOS / desktop
- Splash screen
- Offline : lecture seule des dossiers récemment consultés (cache 7 jours)
- Sync automatique au retour réseau
**Effort** : L

#### US-36 — Vue optimisée mobile
**CA** :
- Layout adaptatif (Tailwind responsive)
- Boutons grands (≥ 44 px tactile)
- Navigation simplifiée
- Modes lecture rapide pour valideurs en déplacement
**Effort** : M

#### US-37 — Notifications push mobile
**CA** : (cf. US-25)
**Effort** : – (déjà couvert)

### Épopée H — Administration et audit (5 US)

#### US-38 — Tableau de bord RSSI
**Persona** : Rachid
**CA** :
- KPIs : auth réussies/échouées, sessions actives, tentatives anormales, alertes
- Top utilisateurs / direction
- Détection patterns suspects (5+ échecs auth, geo anormale)
**Effort** : L

#### US-39 — Gestion des rôles et permissions
**CA** :
- Visualisation matrice rôles × actions
- Édition rôle (par RSSI uniquement)
- Lien vers groupes AD (provisioning automatique)
**Effort** : M

#### US-40 — Audit logs (consultation)
**CA** :
- Recherche multi-critères (acteur, ressource, type événement, période)
- Affichage chronologique
- Export CSV/JSON
- Vérification intégrité hash-chain (bouton « Vérifier »)
**Effort** : M

#### US-41 — Configuration des règles DMN
**Persona** : Direction Crédit + Direction Risques
**CA** :
- Liste des 87 règles avec statut, version
- Édition (création nouvelle version)
- Validation par workflow interne (DC + DR)
- Activation à date programmée
**Effort** : L

#### US-42 — Gestion des templates de notification
**Persona** : Direction Communication
**CA** :
- Liste 18 × 2 = 36 templates
- Édition WYSIWYG simple (objet + corps)
- Variables disponibles {referenceDossier}, {emetteurNom}, etc.
- Aperçu rendu
- Test envoi à soi-même
**Effort** : M

### Épopée I — Conformité et droits (3 US)

#### US-43 — Demande d'exercice droit Loi 09-08 (accès, rectification, etc.)
**Persona** : Tout collaborateur
**CA** :
- Formulaire : type de demande (accès, rectification, opposition, suppression)
- Motif
- Soumission au DPO/correspondant CNDP banque (workflow ticketing)
- SLA 25 jours, alerte 5j avant échéance
- Notification statut user
**Effort** : M

#### US-44 — Politique de confidentialité et CGU
**CA** :
- Page statique multilingue
- Validation au premier login (case à cocher)
- Mise à jour : nouvelle validation requise
**Effort** : S

#### US-45 — Export RGPD-équivalent (mes données)
**CA** :
- Bouton « Télécharger mes données » dans profil
- Génération asynchrone (job BullMQ)
- Email avec lien sécurisé (expiration 7j)
- Format ZIP : profil JSON + liste dossiers JSON + audit personnel JSON
**Effort** : M

### Épopée J — Aide et support (2 US)

#### US-46 — Centre d'aide intégré
**CA** :
- Aide contextuelle (tooltip + lien vers doc)
- Recherche FAQ
- Tutoriels vidéo intégrés (5-6 vidéos courtes)
- Lien vers ticketing support
**Effort** : M

#### US-47 — Support — créer un ticket
**CA** :
- Formulaire intégré
- Catégories : bug, demande d'évolution, question fonctionnelle, problème de connexion
- Priorité (basse / normale / haute / urgent)
- Pièces jointes (max 10 Mo)
- Soumission via API ticketing banque (Jira, GLPI selon banque)
- Suivi statut
**Effort** : M

### 4.2 Synthèse effort

| Épopée | US | Effort total (relatif) |
|---|---|---|
| A — Auth | 4 | 8 pts |
| B — Dossiers | 8 | 22 pts |
| C — Workflow | 10 | 28 pts |
| D — Notifications | 4 | 8 pts |
| E — Recherche/reporting | 5 | 18 pts |
| F — i18n | 3 | 9 pts |
| G — Mobile | 3 | 7 pts |
| H — Admin/audit | 5 | 18 pts |
| I — Conformité | 3 | 7 pts |
| J — Support | 2 | 6 pts |
| **Total** | **47** | **131 pts** |

(1 point ≈ 0,5 JH solo + Claude → ~65 JH frontend pur + back logique. Cohérent avec estimation 60 JH frontend + intégration logique back partielle dans P4.)

---

## 5. Wireframes textuels (écrans clés)

### 5.1 Écran 1 — Login

```
┌──────────────────────────────────────────────────────────────────────┐
│ [Logo banque]                                          🌐 FR ▼       │
│                                                                       │
│                                                                       │
│                ┌────────────────────────────────┐                    │
│                │                                │                    │
│                │   Parapheur Digital            │                    │
│                │                                │                    │
│                │   Matricule banque             │                    │
│                │   ┌──────────────────────┐    │                    │
│                │   │                      │    │                    │
│                │   └──────────────────────┘    │                    │
│                │                                │                    │
│                │   Mot de passe                 │                    │
│                │   ┌──────────────────────┐    │                    │
│                │   │ ●●●●●●●●●●●         │    │                    │
│                │   └──────────────────────┘    │                    │
│                │                                │                    │
│                │   ┌──────────────────────┐    │                    │
│                │   │  Se connecter        │    │                    │
│                │   └──────────────────────┘    │                    │
│                │                                │                    │
│                │   [Aide]  [Mot de passe oublié]│                    │
│                └────────────────────────────────┘                    │
│                                                                       │
│ © Banque  •  Confidentialité  •  Conditions d'utilisation             │
└──────────────────────────────────────────────────────────────────────┘
```

### 5.2 Écran 2 — Dashboard (FR)

```
┌──────────────────────────────────────────────────────────────────────┐
│ ☰ Parapheur                              🔍 [Recherche...]  🔔 12  Khalid B. │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ Bonjour Khalid 👋                                                     │
│ Vous avez 3 actions en attente                                        │
│                                                                       │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐     │
│ │ À valider   │ │ Brouillons  │ │ En cours    │ │ Délais      │     │
│ │     0       │ │     2       │ │     8       │ │ à risque    │     │
│ │             │ │             │ │             │ │     1       │     │
│ │  [→ liste]  │ │  [→ liste]  │ │  [→ liste]  │ │  [→ liste]  │     │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘     │
│                                                                       │
│ Mes dossiers récents                                                  │
│ ┌────────────────────────────────────────────────────────────────┐   │
│ │ Réf            Type             Montant      Statut    Soumis  │   │
│ ├────────────────────────────────────────────────────────────────┤   │
│ │ DOS-2026-...   Eng. crédit     800 K MAD   En cours  J-2     │   │
│ │ DOS-2026-...   Dépense          50 K MAD   Validé    J-3     │   │
│ │ DOS-2026-...   Eng. crédit    1,2 M MAD   Brouillon  J-1     │   │
│ │ ...                                                              │   │
│ └────────────────────────────────────────────────────────────────┘   │
│                                  [+ Nouveau dossier]                  │
└──────────────────────────────────────────────────────────────────────┘
```

### 5.3 Écran 3 — Création de dossier (Wizard étape 2)

```
┌──────────────────────────────────────────────────────────────────────┐
│ Nouveau dossier — Étape 2/4 : Détails                                 │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ Type de décision : Engagement crédit                                  │
│                                                                       │
│ Montant *               Devise *                                      │
│ ┌──────────────────┐   ┌─────────┐                                   │
│ │ 800 000          │   │ MAD ▼   │                                   │
│ └──────────────────┘   └─────────┘                                   │
│  = 74 074,07 EUR (taux BAM jour : 1 EUR = 10,80 MAD)                  │
│                                                                       │
│ Contrepartie *                                                        │
│ ┌──────────────────────────────────────────────────────────────┐     │
│ │ Rechercher par ICE, raison sociale ou ID...                  │     │
│ └──────────────────────────────────────────────────────────────┘     │
│  [➜ ABC SARL — ICE 001234567890123 — segment PME]                    │
│                                                                       │
│ Objet *                                                               │
│ ┌──────────────────────────────────────────────────────────────┐     │
│ │ Découvert autorisé ABC SARL                                  │     │
│ └──────────────────────────────────────────────────────────────┘     │
│                                                                       │
│ Dérogation politique crédit                                           │
│ ☐ Oui (déclenche validation Comité Crédit)                            │
│                                                                       │
│                       [< Retour]   [Suivant →]                        │
└──────────────────────────────────────────────────────────────────────┘
```

### 5.4 Écran 4 — Dashboard valideur (file d'attente)

```
┌──────────────────────────────────────────────────────────────────────┐
│ ☰ Parapheur               🔍       🔔 5    Aïcha M.  (Direction Crédit)│
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ À valider — 12 dossiers                                               │
│                                                                       │
│ Filtres : [Type ▼] [Montant ▼] [Urgence ▼] [Direction émettrice ▼]   │
│                                                                       │
│ ☐ Tout sélectionner                              [Valider sélection]  │
│                                                                       │
│ ┌──────────────────────────────────────────────────────────────────┐ │
│ │ ☐ 🔴 DOS-2026-001234  Eng. crédit ABC SARL    800 K MAD          │ │
│ │      Khalid B. (Casa Centre) — soumis hier — échéance dans 2h    │ │
│ │      [Ouvrir →]                                                   │ │
│ ├──────────────────────────────────────────────────────────────────┤ │
│ │ ☐ 🟡 DOS-2026-001245  Dépense achat IT       250 K MAD           │ │
│ │      Karim S. (DSI) — soumis ce matin — échéance demain          │ │
│ │      [Ouvrir →]                                                   │ │
│ ├──────────────────────────────────────────────────────────────────┤ │
│ │ ☐ 🟢 DOS-2026-001256  Eng. crédit XYZ SA    3,5 M MAD            │ │
│ │      Latifa K. (Rabat) — soumis aujourd'hui — échéance 3 jours   │ │
│ │      [Ouvrir →]                                                   │ │
│ └──────────────────────────────────────────────────────────────────┘ │
│                                                  Page 1/2  [< 1 2 >]  │
└──────────────────────────────────────────────────────────────────────┘
```

### 5.5 Écran 5 — Détail dossier + action de validation SEA

```
┌──────────────────────────────────────────────────────────────────────┐
│ ← DOS-2026-001234                          Statut: En attente validation│
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│ ╔═══════════════════════════════╤═══════════════════════════════╗    │
│ ║ Type : Engagement crédit       │ Montant : 800 000,00 MAD    ║    │
│ ║ Émetteur : Khalid B.           │ Soumis : 24/04/2026 09:15    ║    │
│ ║ Contrepartie : ABC SARL        │ Échéance : 25/04/2026 09:15  ║    │
│ ║ Objet : Découvert autorisé...  │                                ║    │
│ ╚═══════════════════════════════╧═══════════════════════════════╝    │
│                                                                       │
│ Documents (5)                                                         │
│ ▶ business-plan.pdf       (1,2 Mo)  [Aperçu] [Télécharger]            │
│ ▶ rc-abc-sarl.pdf         (450 Ko)  [Aperçu] [Télécharger]            │
│ ▶ ice-attestation.pdf     (180 Ko)  [Aperçu] [Télécharger]            │
│ ▶ if-attestation.pdf      (200 Ko)  [Aperçu] [Télécharger]            │
│ ▶ bilans-3-ans.pdf        (3,8 Mo)  [Aperçu] [Télécharger]            │
│                                                                       │
│ Historique                                                            │
│ • 24/04 09:15 — Khalid B. a créé le dossier                           │
│ • 24/04 09:15 — Khalid B. a soumis le dossier                         │
│ • 24/04 11:00 — Mohamed L. (Resp. Agence Casa Centre) a validé (SES) │
│ • 24/04 14:30 — Aïcha M. a demandé "Ajouter attestation CNSS"         │
│ • 24/04 15:00 — Khalid B. a ajouté attestation-cnss.pdf               │
│                                                                       │
│ Commentaires                                                          │
│ ┌──────────────────────────────────────────────────────────────┐     │
│ │ Aïcha M. (24/04 14:30) :                                      │     │
│ │ Pouvez-vous fournir l'attestation CNSS récente ?              │     │
│ │                                                                │     │
│ │ Khalid B. (24/04 15:00) :                                     │     │
│ │ Voici l'attestation. Merci.                                   │     │
│ └──────────────────────────────────────────────────────────────┘     │
│                                                                       │
│ Actions                                                               │
│ [Valider et signer (SEA)]   [Demander info]   [Rejeter]   [Déléguer] │
└──────────────────────────────────────────────────────────────────────┘
```

### 5.6 Écran 6 — Modal signature SEA + OTP

```
                ┌─────────────────────────────────────────┐
                │ Validation et signature SEA              │
                ├─────────────────────────────────────────┤
                │                                          │
                │ Vous allez signer électroniquement le    │
                │ dossier DOS-2026-001234 d'un montant     │
                │ de 800 000 MAD avec une signature        │
                │ électronique avancée (SEA) ayant force   │
                │ probante.                                 │
                │                                          │
                │ Commentaire (optionnel) :                 │
                │ ┌────────────────────────────────────┐  │
                │ │                                    │  │
                │ │                                    │  │
                │ └────────────────────────────────────┘  │
                │                                          │
                │ Un code OTP a été envoyé au              │
                │ +212 6 ●● ●● ●● 12                       │
                │                                          │
                │ ┌────────────────────────────────────┐  │
                │ │ ●  ●  ●  ●  ●  ●                  │  │
                │ └────────────────────────────────────┘  │
                │                                          │
                │ Renvoyer le code (00:42)                 │
                │                                          │
                │           [Annuler]  [Confirmer]         │
                └─────────────────────────────────────────┘
```

### 5.7 Écran 7 — Vue mobile (PWA, file d'attente)

```
┌─────────────────────┐
│ ☰ 🔔  Aïcha     │
├─────────────────────┤
│                     │
│ À valider (12)      │
│                     │
│ ┌─────────────────┐ │
│ │ 🔴 DOS-2026-... │ │
│ │ ABC SARL        │ │
│ │ 800 K MAD       │ │
│ │ 2h restantes    │ │
│ │       [Ouvrir]  │ │
│ ├─────────────────┤ │
│ │ 🟡 DOS-2026-... │ │
│ │ Achat IT        │ │
│ │ 250 K MAD       │ │
│ │ 1 jour          │ │
│ │       [Ouvrir]  │ │
│ ├─────────────────┤ │
│ │ ...             │ │
│ └─────────────────┘ │
│                     │
│  [+ Nouveau]        │
└─────────────────────┘
```

### 5.8 Écran 8 — Dashboard RTL arabe (illustratif)

```
┌──────────────────────────────────────────────────────────────────────┐
│ خالد ب.   ٥ 🔔        🔍       Parapheur ☰                          │  ← layout miroir
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│                                                          مرحبا خالد   │
│                                  لديك ٣ إجراءات معلقة                 │
│                                                                       │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐     │
│ │ مهلة في خطر │ │ قيد التنفيذ │ │   مسودات    │ │  للتحقق     │     │
│ │      ١      │ │      ٨      │ │      ٢      │ │      ٠      │     │
│ │ [→ القائمة]│ │ [→ القائمة]│ │ [→ القائمة]│ │ [→ القائمة]│     │
│ └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘     │
│                                                                       │
│  ...                                                                  │
└──────────────────────────────────────────────────────────────────────┘
```

(Illustratif. Les libellés exacts à valider avec utilisateurs arabophones banque en P9.)

---

## 6. Internationalisation FR + AR (RTL)

### 6.1 Architecture i18n

```typescript
// i18n config
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

i18n.use(initReactI18next).init({
  fallbackLng: 'fr',
  defaultNS: 'common',
  ns: ['common', 'dossier', 'workflow', 'notifications', 'errors'],
  resources: {
    fr: { /* ... */ },
    ar: { /* ... */ }
  },
  interpolation: { escapeValue: false }  // React échappe déjà
});

// Composant racine
function App() {
  const { i18n } = useTranslation();
  const dir = i18n.language === 'ar' ? 'rtl' : 'ltr';
  return (
    <html dir={dir} lang={i18n.language}>
      ...
    </html>
  );
}
```

### 6.2 Tailwind RTL

```typescript
// tailwind.config.ts
export default {
  plugins: [require('tailwindcss-rtl')],
  // Utilise les classes ms-* (margin-start) au lieu de ml-*, pe-* (padding-end) au lieu de pr-*
};
```

```tsx
<div className="flex items-center gap-3 ms-4">  {/* margin-left en LTR, margin-right en RTL */}
  <Icon name="chevron-end" />  {/* utilise une icône directionnelle adaptée */}
  <span>Suivant</span>
</div>
```

### 6.3 Formats locaux

| Type | FR | AR |
|---|---|---|
| Date | 24/04/2026 | 24/04/2026 (ou ٢٤/٠٤/٢٠٢٦ selon préférence) |
| Heure | 14:30 | 14:30 (ou ١٤:٣٠) |
| Montant | 1 234 567,89 MAD | 1 234 567,89 درهم (ou ١٬٢٣٤٬٥٦٧٫٨٩ درهم) |
| Pourcentage | 12,5 % | 12,5 ٪ |
| Liste enum | « Validé », « En cours » | « موَثّق »، « قيد التنفيذ » |

### 6.4 Volume traductions

Estimation : ~600 clés à traduire FR → AR.

**Effort** : 4 JH (avec Claude pour aide à la traduction technique + relecture par utilisateur arabophone banque en P9).

---

## 7. Accessibilité WCAG 2.1 AA + RTL

### 7.1 Critères WCAG retenus

| Critère | Niveau | Implémentation |
|---|---|---|
| 1.1.1 Contenu non textuel | A | `alt` + `aria-label` sur tous éléments visuels |
| 1.3.1 Information et relations | A | HTML sémantique (h1-h6, nav, main, etc.) |
| 1.4.3 Contraste (minimum) | AA | Ratio ≥ 4.5:1 (texte normal), ≥ 3:1 (grand) |
| 1.4.4 Redimensionnement texte | AA | Zoom 200 % sans perte fonctionnelle |
| 1.4.10 Reflow | AA | Pas de scroll horizontal à 320 px largeur |
| 2.1.1 Clavier | A | Toutes actions accessibles au clavier |
| 2.4.4 Objet du lien | A | Libellés explicites |
| 2.4.7 Visibilité focus | AA | Outline visible sur focus |
| 3.1.2 Langue d'une partie | AA | `lang` sur portions multilingues |
| 3.3.1 Identification erreur | A | Messages d'erreur explicites |
| 3.3.3 Suggestion erreur | AA | Suggestions de correction |
| 4.1.2 Nom, rôle, valeur | A | ARIA quand HTML insuffisant |

### 7.2 Tests RTL spécifiques

- Navigation au clavier en RTL (Tab progresse de droite à gauche)
- Lecteurs d'écran arabes (NVDA + VoiceOver iOS — direction reconnue)
- Direction des icônes (flèches, retour, étape suivante)
- Tests utilisateurs arabophones banque (P9 — 3-5 utilisateurs)

### 7.3 Outils

- **Lighthouse** (Chrome DevTools)
- **axe DevTools** (extension navigateur)
- **WAVE** (en complément)
- **NVDA** + **VoiceOver** (lecteurs d'écran)
- **Manual testing** keyboard only

---

## 8. Critères d'acceptation transverses

### 8.1 Performance

| Indicateur | Cible |
|---|---|
| First Contentful Paint (FCP) | < 1,5 s |
| Largest Contentful Paint (LCP) | < 2,5 s |
| Time to Interactive (TTI) | < 3 s |
| API P95 latence | < 500 ms |
| Bundle JS initial | < 250 Ko gzip |

### 8.2 Sécurité (rappel P3)

- HTTPS obligatoire (TLS 1.3)
- CSP strict (nonces)
- Cookies SameSite=Strict + HttpOnly + Secure
- CSRF tokens sur toutes mutations
- Validation Zod côté front ET back
- Pas de secrets en clair dans le code/logs

### 8.3 Compatibilité navigateurs

| Navigateur | Version min |
|---|---|
| Chrome / Edge / Brave | 100+ |
| Firefox | 100+ |
| Safari | 15+ |
| Mobile Safari iOS | 15+ |
| Chrome Android | 100+ |

(Compatibilité IE 11 hors périmètre — banque a typiquement fait la transition.)

### 8.4 Critères qualité code

- ESLint + Prettier passent en CI
- TypeScript strict mode (`strict: true`)
- Tests Vitest coverage ≥ 80 %
- Pas de `any` non justifié
- Pas de console.log en prod
- Pas de TODO non tracé en backlog

---

## 9. Annexes

### Annexe A — Décisions COPIL produit

| ID | Décision | Recommandation |
|---|---|---|
| **D-R5** | Authentification | AD banque + MFA OTP |
| **D-R10** | Devise | MAD pivot |
| **D-R11** | Langues UI | FR + AR (RTL) |
| **Prod-1** | Mobile | PWA installable + offline lecture |
| **Prod-2** | Compatibilité | Browsers modernes (>2022), pas IE |
| **Prod-3** | Accessibilité | WCAG 2.1 AA + tests RTL |

### Annexe B — Roadmap fonctionnelle

| Vague | Période | Périmètre |
|---|---|---|
| **MVP — vague 1** (P6, sem 9-20) | Q3 2026 | US-01, 03, 05, 07, 09, 13, 14, 15, 18, 23, 26, 27, 29, 32-34, 44 (≈ 16 US) |
| **Vague 2** (P7, sem 21-34) | Q4 2026 | US-02, 04, 06, 08, 10, 11, 12, 16, 17, 19, 20, 21, 22, 24, 25, 28, 30, 35, 36, 39, 40, 41, 42, 43, 45 (≈ 25 US) |
| **Vague 3** (P8, sem 35-42) | Q1 2027 | US-31, 38, 46, 47 + finitions + perf + sécurité E2E (4 US + dette) |
| **Run** | continu | Évolutions, ajustements suite UAT, nouvelles US |

### Annexe C — Glossaire UX

| Terme | Définition |
|---|---|
| **CTA** | Call To Action — bouton principal incitant à l'action |
| **Deep link** | Lien direct vers une ressource précise dans l'app |
| **PWA** | Progressive Web App |
| **RTL** | Right-To-Left (sens d'écriture arabe, hébreu) |
| **Splash screen** | Écran d'accueil au lancement d'une PWA |
| **Toast** | Notification temporaire en bas/coin d'écran |
| **Tooltip** | Bulle d'aide contextuelle |
| **Wizard** | Assistant multi-étapes |

### Annexe D — Tableau récapitulatif des 47 US

| ID | Titre | Épopée | Effort |
|---|---|---|---|
| US-01 | Connexion AD + MFA | A | M |
| US-02 | Délégation expresse | A | M |
| US-03 | Profil utilisateur | A | S |
| US-04 | Voir rôles & délégations | A | S |
| US-05 | Créer dossier | B | L |
| US-06 | Modifier brouillon | B | S |
| US-07 | Soumettre dossier | B | M |
| US-08 | Annuler dossier | B | S |
| US-09 | Charger documents | B | M |
| US-10 | Lookup contrepartie | B | M |
| US-11 | Dupliquer dossier | B | S |
| US-12 | Affichage multi-devises | B | M |
| US-13 | File d'attente validation | C | M |
| US-14 | Détail dossier | C | L |
| US-15 | Valider SES | C | M |
| US-16 | Valider SEA | C | L |
| US-17 | Valider SEQ optionnel | C | M |
| US-18 | Rejeter | C | S |
| US-19 | Demander info | C | M |
| US-20 | Validation par lot | C | L |
| US-21 | Déléguer ad hoc | C | M |
| US-22 | Escalader | C | S |
| US-23 | Notifications email | D | M |
| US-24 | Notifications SMS | D | S |
| US-25 | Notifications push PWA | D | M |
| US-26 | Centre de notifications | D | M |
| US-27 | Recherche full-text | E | M |
| US-28 | Recherche avancée | E | M |
| US-29 | Tableau de bord personnel | E | M |
| US-30 | Tableau de bord direction | E | L |
| US-31 | Reporting BAM/AMMC | E | L |
| US-32 | Bascule FR/AR | F | M |
| US-33 | Affichage RTL | F | L |
| US-34 | Format nombres/dates | F | M |
| US-35 | PWA | G | L |
| US-36 | Vue mobile | G | M |
| US-37 | Push mobile | G | (≡US-25) |
| US-38 | Dashboard RSSI | H | L |
| US-39 | Gestion rôles | H | M |
| US-40 | Audit logs | H | M |
| US-41 | Configuration DMN | H | L |
| US-42 | Templates notifications | H | M |
| US-43 | Demande droit Loi 09-08 | I | M |
| US-44 | CGU + politique conf | I | S |
| US-45 | Export mes données | I | M |
| US-46 | Centre d'aide | J | M |
| US-47 | Créer ticket support | J | M |

---

*Fin de la Phase 1 v2 — Spécifications détaillées.*

**Programme complet livré** : Phases 1, 2, 3, 4 v2 + REVISION-CONTEXTE-MAROC-SOLO.md + COPIL deck révisé.
