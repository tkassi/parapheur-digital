# 📊 DIAGRAMMES BPMN – PARAPHEUR DIGITAL

**Représentation des flux processus en notation BPMN 2.0**

---

## D1 : FLUX PRINCIPAL – Création à Archivage

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          PARAPHEUR DIGITAL - FLUX COMPLET                │
└─────────────────────────────────────────────────────────────────────────┘

Actors: Initiateur | Validateurs | Signataires | Système | Auditeurs

(START)
   │
   ├─────────────────────────────────────────────────────────────────────┐
   │ PHASE 1 : CRÉATION & DÉPÔT                                         │
   ├─────────────────────────────────────────────────────────────────────┤
   │
   ▼
[Initiateur remplit métadonnées]  ← Title, Classification, Entity, Urgence
   │
   ▼
[Upload documents]  ← Contrôle format, size, virus scan
   │
   ▼
{Contrôle conformité}  ← At least 1 doc, circuit validé, no loops
   │
   ├─ KO ──→ [Rejeter dépôt] ──→ [Notifier initiateur]
   │
   ├─ OK
   │
   ├─────────────────────────────────────────────────────────────────────┐
   │ PHASE 2 : VALIDATION À N NIVEAUX                                   │
   ├─────────────────────────────────────────────────────────────────────┤
   │
   ▼
[Créer dossier] → État: EN_ATTENTE_VALIDATION
   │
   ├─ Boucle: Pour k = 1 à N (niveaux)
   │
   ├──► [Identifier acteur niveau k] ← Rôle, délégation
   │     │
   │     ▼
   │   [Notifier validateur k]  ← Email, dashboard
   │     │
   │     ▼
   │   [Validateur consulte dossier] ← Dashboard, 48h-24h review
   │     │
   │     ├────────────────────────────────────────────┐
   │     │ Décision validateur                        │
   │     ├────────────────────────────────────────────┤
   │     │
   │     ├─ VALIDE ──────────────┐
   │     │                       ▼
   │     │              [Enregistrer action]
   │     │              ↓ k ← k+1
   │     │              (Continuer boucle)
   │     │
   │     ├─ REJETTE ─────────────┐
   │     │                       ▼
   │     │              [Enregistrer rejet]
   │     │                       ▼
   │     │              [Dossier → EN_MODIFICATION]
   │     │                       ▼
   │     │              [Notifier initiateur]
   │     │                       ▼
   │     │              [PAUSE: En attente correction]
   │     │                       ▼
   │     │              [Initiateur modifie + relance]
   │     │                       ▼
   │     │              (Retour k ← 1)
   │     │
   │     ├─ DÉLÈGUE ─────────────┐
   │     │                       ▼
   │     │              [Transfert vers délégué]
   │     │                       ▼
   │     │              [Notifier délégué + validateur]
   │     │              (Délégué remplace validateur)
   │     │
   │     └─ PAUSE/COMMENTER ───┐
   │                           ▼
   │                   [Enrichir audit trail]
   │                   (Puis VALIDE ou REJETTE)
   │
   ├─ Fin boucle: Tous niveaux = VALIDÉ
   │
   ├─────────────────────────────────────────────────────────────────────┐
   │ PHASE 3 : SIGNATURE ÉLECTRONIQUE                                   │
   ├─────────────────────────────────────────────────────────────────────┤
   │
   ▼
[État → EN_SIGNATURE]
   │
   ├─ Boucle: Pour S = 1 à N_signataires
   │
   ├──► [Notifier signataire S]
   │     │
   │     ▼
   │   [Signataire consulte document final]
   │     │
   │     ├────────────────────────────────────────────┐
   │     │ Type signature                             │
   │     ├────────────────────────────────────────────┤
   │     │
   │     ├─ SIMPLE ──────────────┐
   │     │ (Click + OTP)         ▼
   │     │              [Vérifier OTP]
   │     │                       ▼
   │     │              [Signature simple + timestamp]
   │     │
   │     ├─ AVANCÉE ─────────────┐
   │     │ (Cert PKI + TSA)      ▼
   │     │              [Auth forte (2FA)]
   │     │                       ▼
   │     │              [Générer hash + signer]
   │     │                       ▼
   │     │              [Appel TSA (horodatage)]
   │     │                       ▼
   │     │              [Encodage PKCS#7]
   │     │
   │     └─ QUALIFIÉE ──────────┐
   │       (Service tiers)      ▼
   │                   [Redirect service qualifié]
   │                           ▼
   │                   [Auth forte (biométrique)]
   │                           ▼
   │                   [Signature qualifiée (HSM)]
   │                           ▼
   │                   [Retour preuve + certificat]
   │
   │                           ▼
   │              [Enregistrer signature]
   │                           ▼
   │              [Confirmer à signataire]
   │              
   ├─ Fin boucle: Tous signataires = SIGNÉ
   │
   ├─────────────────────────────────────────────────────────────────────┐
   │ PHASE 4 : CLÔTURE & ARCHIVAGE                                      │
   ├─────────────────────────────────────────────────────────────────────┤
   │
   ▼
[État → SIGNÉ]
   │
   ▼
[Générer document final signé]  ← PDF/XML complet + métadonnées
   │
   ▼
[Créer sceau électronique global]  ← Hash + signature autorité
   │
   ▼
[Indexation & OCR]  ← Métadonnées, mots-clés, contenu searchable
   │
   ▼
[Archivage GED]  ← Zone chaude, réplication, backup
   │
   ▼
[Configurer accès]  ← Initiateur, signataires : lecture seule
   │
   ▼
[Notifications finales]  ← Email tous participants + statistiques
   │
   ▼
[État → ARCHIVÉ]
   │
   ▼
[Déplacement zone froide (après délai légal)]  ← 3 ans après
   │
   ▼
(END)


─────────────────────────────────────────────────────────────────────────
ERREURS PENDANT LE FLUX :

{Dépassement SLA} ──→ [Escalade auto] ──→ [Notifier N+1]
                              │
                              ▼
                     [Possibilité transfère délégué]

{Certificat expiré} ──→ [Bloquer signature] ──→ [Notifier admin]
                             │
                             ▼
                     [Renouveler certificat]

{Incident sécurité} ──→ [Bloquer dossier] ──→ [Audit + SOC]
```

---

## D2 : PROCESSUS VALIDATION (DÉTAIL – Niveau k)

```
┌────────────────────────────────────────────────────────────────┐
│         PROCESSUS VALIDATION À UN NIVEAU                       │
└────────────────────────────────────────────────────────────────┘

Pool: Système | Validateur | Manager

(Dossier reçu au niveau k)
              │
              ▼
      {Timer SLA = Now + 48h}  ← Configuration selon rôle
              │
              ▼
      [Identifier acteur pour rôle]
              │
              ├─ User configuré OK ? ──→ YES
              │                         │
              │                         ▼
              │                 {Check délégation}
              │                         │
              │                    ├─ Délégation active
              │                    │    ▼
              │                    │  [User → Délégué]
              │                    │
              │                    └─ Non
              │                         ▼
              │                    [Utiliser User]
              │
              └─ User absent > 3j ──→ YES
                                      │
                                      ▼
                                [Admin force délégation]
                                      │
                                      ▼
                                [Notifier User + Délégué]
              │
              ▼
      [Notifier validateur choisi]  ← Email HTML + push
              │
              ▼
      [Dashboard: Tâche affichée]  ← Badge rouge, SLA visible
              │
              ▼
      [Validateur se connecte]
              │
              ├─ Timeout 48h atteint ──→ YES
              │                         │
              │                         ▼
              │                 {Début escalade}
              │                         │
              │                    ├─ Escalade +48h buffer
              │                    │    ▼
              │                    │  [Notifier manager]
              │                    │  + [Alerte rouge]
              │                    │
              │                    ├─ Timeout +48h+48h ──→ YES
              │                    │    ▼
              │                    │  [Force signature délégué]
              │                    │  ou [Escalade niveau+1]
              │
              └─ Non
                   ▼
      [Consulter document]  ← Max 2 sessions/jour
         (Review détail)         (Analytics: 30 min avg)
              │
              ▼
      ┌──────────────────────────────────┐
      │ Décision Validateur              │
      └──────────────────────────────────┘
              │
    ┌─────────┼─────────┬──────────────┬───────────────┐
    │         │         │              │               │
    ▼         ▼         ▼              ▼               ▼
  VALIDE   REJETTE   DÉLÈGUE      PAUSE/COMMENTE   REQUEST_INFOS
   (60%)    (15%)     (5%)           (15%)            (5%)
    │         │         │              │               │
    ├─→ A    ├─→ B    ├─→ C          ├─→ D           ├─→ E
    
A: [VALIDATION]
   │
   ├─ {Step = dernière ?}
   │  ├─ YES ──→ [État → EN_SIGNATURE]
   │  │         ↓ (Aller PHASE 3)
   │  └─ NO  ──→ [k ← k+1, continuer niveau suivant]
   │
   └─ [Enregistrer audit trail]
      {action: APPROVE, timestamp, user, comment}
      
B: [REJET]
   │
   ├─ [Saisir motif + catégorie]
   │  (INCOMPLET / ERREUR / NON-CONFORME)
   │
   ├─ [Enregistrer rejet] (audit)
   │
   ├─ {Nb rejets déjà = 2 ?}
   │  ├─ YES ──→ [Escalade admin, review manual]
   │  └─ NO  ──→ [Continuer rejet normal]
   │
   └─ [État → EN_MODIFICATION]
      └─ [Notifier initiateur + manager]
         └─ [PAUSE: Attendre correction + relance]

C: [DÉLÉGATION]
   │
   ├─ [Saisir délégué désigné]
   │  (Doit avoir rôle >= rôle actuel)
   │
   ├─ {Check autorité délégué}
   │  ├─ OK   ──→ [Procéder]
   │  └─ FAIL ──→ [Erreur + rejeter]
   │
   ├─ [Délégation enregistrée]
   │  {validité: 30 jours max ou date fin}
   │
   └─ [Notifier délégué + parties prenantes]
      └─ [Attendre action délégué]

D: [PAUSE / ENRICHISSEMENT]
   │
   ├─ [Saisir commentaire texte libre]
   │  (Insight, question, demande clarif)
   │
   ├─ [Notifier parties concernées]
   │  (Initiateur, validateurs précédents)
   │
   └─ [Puis décider: VALIDE ou REJETTE]
      (Retour vers A ou B)

E: [REQUEST INFOS]
   │
   ├─ [Envoyer demande clarification]
   │  à initiateur ou validateur précédent
   │
   ├─ [État → EN_REVISION_NIVEAU_N]
   │
   └─ [Attendre retour, puis relancer]
      (Max 2 retours en arrière = sécurité)


(FIN NIVEAU k)
```

---

## D3 : PROCESSUS SIGNATURE (DÉTAIL)

```
┌──────────────────────────────────────────────────────────┐
│            FLUX SIGNATURE ÉLECTRONIQUE                   │
└──────────────────────────────────────────────────────────┘

Pool: Système | Signataire | TSA | Service tiers

(Tous niveaux validés)
        │
        ▼
[État → EN_SIGNATURE]
        │
        ▼
[Identifier signataires selon circuit]
        │
        ├─ Mode SÉQUENTIEL  ──→ [S1] → [S2] → [S3]
        │
        └─ Mode PARALLÈLE   ──→ [S1] ∥ [S2] ∥ [S3]
                                  (Tous doivent signer)
        │
        ▼
[Boucle: Pour chaque signataire S]
        │
        ├──► [Notifier signataire S]
        │    {Email + push notification}
        │    {Deadline: 24h}
        │    │
        │    ▼
        │  [Signataire accède interface]
        │    {Login SSO + 2FA}
        │    {Dashboard consultation}
        │    │
        │    ▼
        │  [Consulter document final SYNTHÉTIQUE]
        │    {Méta + docs + décisions validateurs}
        │    {Durée moyenne: 5-10 min}
        │    │
        │    ▼
        │  ┌──────────────────────────────────┐
        │  │ TYPE SIGNATURE                   │
        │  └──────────────────────────────────┘
        │    │
        │    ├─ [SIMPLE]
        │    │  │
        │    │  ├─ [Click "Je signe"]
        │    │  │
        │    │  ├─ [Envoyer OTP]
        │    │  │  {SMS ou email}
        │    │  │
        │    │  ├─ [Saisir OTP]
        │    │  │
        │    │  ├─ [Vérifier code]
        │    │  │  ├─ OK   ──→ [Procéder]
        │    │  │  └─ FAIL ──→ [Retry, max 3x]
        │    │  │
        │    │  └─ [Signature appliquée]
        │    │     {Timestamp serveur}
        │    │     {Format: simple, preuve contrat}
        │    │
        │    ├─ [AVANCÉE]
        │    │  │
        │    │  ├─ [Authentification forte]
        │    │  │  {LDAP + badge ou token}
        │    │  │
        │    │  ├─ [Vérifier certificat valide]
        │    │  │  ├─ Expiré    ──→ [Bloquer, notifier admin]
        │    │  │  ├─ Révoqué   ──→ [Bloquer, notifier admin]
        │    │  │  └─ Valide    ──→ [Procéder]
        │    │  │
        │    │  ├─ [Générer hash SHA-256 doc]
        │    │  │
        │    │  ├─ [Signer avec clé privée]
        │    │  │  {RSA 2048 bit}
        │    │  │  {Métadonnées: user, timestamp, IP, device}
        │    │  │
        │    │  ├─ [Appel TSA externe]
        │    │  │  {RFC 3161 compliant}
        │    │  │  {Horodatage autorité tierce}
        │    │  │
        │    │  ├─ [Récup TimeStamp Token]
        │    │  │  {Hash + signature TSA + certificat}
        │    │  │
        │    │  ├─ [Encodage PKCS#7 / CMS]
        │    │  │  {Signature + certificat + timestamp}
        │    │  │
        │    │  └─ [Signature avancée OK]
        │    │     {Légal France/UE}
        │    │
        │    └─ [QUALIFIÉE]
        │       │
        │       ├─ [Redirection service tiers]
        │       │  {Atos, Docusign, Universign}
        │       │
        │       ├─ [Authentification sur service]
        │       │  {2FA + SMS + biométrique}
        │       │
        │       ├─ [Signature qualifiée (HSM)]
        │       │  {Certificat réputé eIDAS}
        │       │  {Infrastructure sécurisée}
        │       │
        │       ├─ [Service retourne token proof]
        │       │  {Signature qualifiée + certificat}
        │       │
        │       └─ [Retour au parapheur]
        │          {Intégration preuve}
        │
        │    ▼
        │  [Enregistrer signature complète]
        │  {
        │    signature_id: unique,
        │    signatory: user,
        │    type: simple|avancée|qualifiée,
        │    timestamp: UTC,
        │    certificate_chain: [...]
        │    hash_doc: sha256_value,
        │    proof_url: URL_verification
        │  }
        │
        │    ▼
        │  [Envoyer confirmation à signataire]
        │  {Email + preuve téléchargeable}
        │
        │    ▼
        │  [Notifier parties prenantes]
        │  {Initiateur, validateurs, autres signataires}
        │
        ├─ (Mode SÉQUENTIEL: aller S+1)
        ├─ (Mode PARALLÈLE: attendre tous)
        │
[FIN BOUCLE]
        │
        ▼
{Tous signataires OK ?}
        │
        ├─ YES ──→ [Générer document final PDF/XML]
        │          {Fusion docs + métadonnées + preuves}
        │          │
        │          ▼
        │          [Créer sceau électronique global]
        │          {Hash complet + signature autorité}
        │          │
        │          ▼
        │          [État → SIGNÉ]
        │          │
        │          ▼
        │          [Aller PHASE 4 ARCHIVAGE]
        │
        └─ NO  ──→ [Attendre signature manquante]
                   {Escalade après SLA+48h}
```

---

## D4 : GESTION EXCEPTION – Escalade SLA

```
┌────────────────────────────────────────────────────────┐
│        ESCALADE AUTOMATIQUE (SLA DÉPASSÉ)              │
└────────────────────────────────────────────────────────┘

Pool: Système | Validateur | Manager | Admin

[Dossier EN_ATTENTE_VALIDATION au niveau k]
        │
        ▼
{Timer SLA déclenché}
{SLA = 48h business days (ex: Manager)}
        │
        ├─ [SLA + 0h] ──→ [ALERT 1: Email validateur]
        │                {Rappel courtois}
        │
        ├─ [SLA + 24h] ──→ [ALERT 2: Email + SMS]
        │                 {CC Manager validateur}
        │
        ├─ [SLA + 48h] ──→ [ESCALADE ACTIVE]
        │
        │        [État → EN_ESCALADE]
        │
        │        ▼
        │  [Notifier manager du validateur]
        │  {Message URGENT, Slack + SMS}
        │
        │        ▼
        │  [Activer options escalade]
        │  │
        │  ├─ Option A: Déléguer au manager
        │  │  [Manager peut signer à la place]
        │  │
        │  ├─ Option B: Transférer N+1 hiérarchique
        │  │  [Director peut approuver]
        │  │
        │  └─ Option C: Forcer signature
        │     [Après +48h supplémentaires]
        │
        │        ▼
        │  [Enregistrer audit]
        │  {ESCALADE_LEVEL_K_POR_SLA}
        │
        │        ▼
        │  [Dashboard affichage]
        │  {Dossier en rouge, criticité MAX}
        │
        ├─ [SLA + 96h (48h+48h)] ──→ [FORCE SIGNATURE]
        │                             (Si pas résolu)
        │                             {Signature délégué auto}
        │                             {Notification tous}
        │
        │        ▼
        │  [Dossier avance vers signature]
        │  ou [Escalade admin manuel]
        │
        └─ [SLA respecté] ──→ [Fin normal]
                            {Aucune escalade}
                            {Continuer flux régulier}
```

---

## D5 : GESTION EXCEPTION – Rejet et Modification

```
┌──────────────────────────────────────────────────────┐
│        PROCESSUS REJET & MODIFICATION                │
└──────────────────────────────────────────────────────┘

Pool: Validateur | Initiateur | Système

[Validateur decide REJETTER]
        │
        ▼
[Saisir motif rejet]
        │
        ├─ Catégorie : [Dropdown]
        │  ├─ INCOMPLET (40%)
        │  ├─ ERREUR_DONNÉES (35%)
        │  ├─ NON_CONFORME (25%)
        │  └─ AUTRE (texte libre)
        │
        ├─ Description : [Text area]
        │  {Min 20 char, max 500}
        │  {Ex: "Montant erroné, doit être 8500€ pas 9500€"}
        │
        ▼
[Enregistrer rejet]
{
  action: REJECT,
  level: k,
  motif_categorie: [Enum],
  motif_description: [Text],
  timestamp: [ISO 8601],
  audit_trail: [Enregistré]
}
        │
        ▼
{Nombre rejets déjà < 2 ?}
        │
        ├─ YES (1er ou 2e rejet)
        │  │
        │  ├─ [État → EN_MODIFICATION]
        │  │
        │  ├─ [Notifier initiateur]
        │  │  {Email détaillé avec motif}
        │  │  {Lien consultation document}
        │  │  {Deadline correction: 5 business days}
        │  │
        │  └─ [PAUSE: Attendre correction]
        │     {Dashboard: "EN ATTENTE VOTRE ACTION"}
        │     {Initiateur accède modification}
        │              │
        │              ▼
        │         [Télécharger documents]
        │              │
        │              ▼
        │         [Modifier métadonnées ou docs]
        │              │
        │              ▼
        │         [Cliquer "Relancer validation"]
        │              │
        │              ▼
        │         [Dossier retour niveau k]
        │         {Pas reset début = rapide}
        │         {Même validateur si dispo}
        │
        │         [Boucle: Retour PHASE 2]
        │
        └─ NO (3e rejet)
           │
           ├─ [État → ESCALADE_MANUEL]
           │
           ├─ [Notifier admin]
           │  {Alert rouge: "Dossier rejeté 3x"}
           │  {Demande review manual}
           │
           └─ [Admin intervient]
              {Peut: forcer approuver, annuler, etc.}


```

---

## D6 : FLUX DÉLÉGATION

```
┌──────────────────────────────────────────────────────┐
│        PROCESSUS DÉLÉGATION DE POUVOIR               │
└──────────────────────────────────────────────────────┘

Pool: Validateur | Délégué | Admin | Système

SCÉNARIO A: User configure délégation (proactif)
─────────────────────────────────────────────

[Validateur accède settings]
        │
        ▼
[Configurer délégations futures]
        │
        ├─ [Période: DATE_DÉBUT → DATE_FIN]
        │  {Ex: 2026-05-01 → 2026-05-30}
        │
        ├─ [Délégué: SELECT user]
        │  {Doit avoir rôle >= sien}
        │
        ├─ [Motif: Congé / Maladie / Sabbatique]
        │
        └─ [SAUVEGARDER]
           {Activation: immédiate ou future}
           {Notification délégué envoyée}


SCÉNARIO B: Admin détecte absence (passif)
──────────────────────────────────────────

[Absence User > 3 jours détectée]
        │
        ▼
{Système batch: Vérifier dernière login}
        │
        ├─ {Last_Login < NOW - 3j ?}
        │
        ├─ YES ──→ [ALERTE admin]
        │           {Dashboard: "User XX absent"}
        │           {Suggérer délégation}
        │
        └─ NO ──→ [Fin, aucune action]


ACTIVATION DÉLÉGATION
─────────────────────

[Dossier arrivé à niveau k]
[User normal = USER_A]
[Délégation active de USER_A → USER_B]
        │
        ▼
[Système reroute vers USER_B]
        │
        ├─ [Vérifier Rôle USER_B >= Rôle USER_A]
        │  ├─ OK   ──→ [Procéder]
        │  └─ FAIL ──→ [Erreur, notifier admin]
        │
        ├─ [Notifier USER_B]
        │  {Email: "Vous êtes délégué de USER_A"}
        │  {Durée: du DATE_DÉBUT au DATE_FIN}
        │  {Motif: [texte]}
        │
        ├─ [Dashboard USER_B]
        │  {Tâche assignée "VIA DÉLÉGATION de USER_A"}
        │  {Badge spécial: "Délégué"}
        │
        ├─ [USER_B traite dossier]
        │  {Décide: VALIDE / REJETTE / etc.}
        │
        └─ [Audit trail enregistre]
           {
             action: VALIDATION,
             by: USER_B,
             on_behalf_of: USER_A,
             delegated: true,
             delegation_valid_until: DATE_FIN
           }


FIN DÉLÉGATION
──────────────

{Date fin délégation atteinte}
        │
        ├─ [Désactiver délégation AUTO]
        │
        ├─ [Notifier USER_A]
        │  {Délégation expirée}
        │
        └─ [Prochains dossiers]
           {Reroute vers USER_A (si dispo)}
           {Sinon: escalade admin}


LIMITE SÉCURITÉ
───────────────

{Délégation > 30 jours}
        │
        ├─ [AVERTISSEMENT: "Durée > 30j anormale"]
        │
        └─ [Admin doit approuver prolongation]
           {Validation manuelle requise}
```

---

## D7 : CYCLE VIE DOSSIER (États)

```
┌─────────────────────────────────────────────┐
│   MACHINE À ÉTATS – VIE COMPLÈTE DOSSIER    │
└─────────────────────────────────────────────┘

                     ┌─ INITIAL
                     │
                 ┌───▼────┐
                 │ CRÉÉ   │
                 └───┬────┘
                     │
        ┌────────────┴────────────┐
        │                         │
    (Normal)              (Erreur conformité)
        │                         │
        ▼                         ▼
   ┌──────────────┐      [Rejeter dépôt]
   │EN_ATTENTE_   │         │
   │ VALIDATION   │         ▼
   └──────┬───────┘    (FIN ERREUR)
          │
          ├──────────────────────────┐
          │                          │
      (Valide)              (Rejette)
          │                          │
          ▼                          ▼
    ┌─────────────┐         ┌──────────────────┐
    │Niveau suivant        │EN_MODIFICATION   │
    │(k ← k+1)            │ (Attente correct) │
    │si k < N             └──────┬───────────┘
    │                            │
    │                       (Relance)
    │                            │
    │                     (Retour k ← 1)
    │                            │
    │                    ┌────────┘
    └────────────────────┤
                         │
                  (Dernier K validé)
                         │
                         ▼
              ┌──────────────────┐
              │  EN_SIGNATURE    │
              └────────┬─────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
    (Sig.OK)    (Timeout)    (Erreur)
        │           │            │
        ▼           ▼            ▼
   [Tous OK]   [Escalade]  [Révocation]
        │           │            │
        │           ▼            │
        │    [Force sig]         │
        │      ou                │
        │    [Annulation]    ┌───┘
        │           │        │
        └───────────┼────────┘
                    │
                    ▼
           ┌──────────────────┐
           │    SIGNÉ        │
           └────────┬────────┘
                    │
          (Immédiat après 24h)
                    │
                    ▼
         ┌──────────────────┐
         │   ARCHIVÉ       │
         │ (Immuable)      │
         │ (Lecture seule) │
         └────────┬────────┘
                  │
        (Après 3 ans légal)
                  │
                  ▼
         ┌──────────────────┐
         │ ARCHIVÉ_FROID   │
         │(Zone cold store) │
         └────────┬────────┘
                  │
        (Après 10 ans)
                  │
                  ▼
         [DESTRUCTION légale]


TRANSITION ANNULATION (à tout moment)
──────────────────────────────────────

[*] ──ANNULER──→ ┌──────────────┐
                 │   ANNULÉ    │
                 │ (Non supprimé)
                 │ (Historique ok)
                 └─────────────┘

{État: ANNULÉ}
{Motif obligatoire}
{Audit trail complet}
{Droits: lecture seule}
```

---

## D8 : POOL & SWIMLANES (Acteurs)

```
┌────────────────────────────────────────────────────────┐
│        INTERACTION MULTI-ACTEURS                       │
└────────────────────────────────────────────────────────┘

    Initiateur          Validateurs        Signataires       Système
    ─────────          ────────────        ───────────       ──────────
        │                  │                   │                │
    ┌───▼─────┐        ┌───▼─────┐        ┌───▼─────┐     ┌───▼─────┐
    │Créer     │   ←→   │Consulter│   ←→   │Signer   │ ←→  │Audit    │
    │dossier   │        │dossier  │        │doc      │     │trail    │
    └───┬─────┘        └───┬─────┘        └───┬─────┘     └───┬─────┘
        │                  │                   │                │
        │ (Relancer)       │ (Rejette)        │ (Preuves)      │
        │                  │                  │                │
        └──────────────────┴──────────────────┴────────────────┘
                           (Notifications)


INTERACTIONS:
─────────────

1. Initiateur → Système: Create dossier + upload docs
   ↓ (Webhook/Event)
2. Système → Validateur: Email + Dashboard notification
3. Validateur → Système: Décision (VALIDE/REJETTE/DÉLÈGUE)
   ↓ (Webhook/Event)
4. Système → Signataire: Email "Ready to sign"
5. Signataire → Système: Signature + certificat
   ↓ (Webhook/Event)
6. Système → GED: Archivage final
7. Système → Auditeur: Log audit trail (acces read-only)
8. Auditeur → Système: Consultation rapports (no modify)

ÉVÉNEMENTS ASYNCHRONES:
──────────────────────

dossier.créé         → Webhook ERP (SAP)
dossier.validé       → Notif next level
dossier.rejeté       → Retour initiateur
dossier.escaladé     → Alert manager
dossier.signé        → Webhook SAP + GED
dossier.archivé      → Freeze, backup
```

---

## Résumé Diagrammes BPMN

| Diagramme | Focus | Flux |
|-----------|-------|------|
| **D1** | Complet | Création → Archivage (all phases) |
| **D2** | Validation | Détail 1 niveau + décisions |
| **D3** | Signature | Types sig + TSA + certificats |
| **D4** | Exception | Escalade SLA auto |
| **D5** | Exception | Rejet + modification |
| **D6** | Exception | Délégation proactive/passive |
| **D7** | États | Cycle vie dossier (state machine) |
| **D8** | Acteurs | Interactions multi-acteurs (swimlanes) |

**Pour visualisation graphique**: Importer ces descriptions dans Lucidchart, Draw.io, ou BPMN editor.

