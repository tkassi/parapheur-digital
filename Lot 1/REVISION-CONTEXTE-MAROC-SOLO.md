# Programme Parapheur Digital — Révision contextuelle

**Document amendant les Phases 1, 2, 3, 4 et la synthèse COPIL**

| Métadonnée | Valeur |
|---|---|
| Version | R1.0 — Avril 2026 |
| Auteur | Direction de programme |
| Statut | À valider COPIL |
| Documents amendés | PHASE1, PHASE2, PHASE3, PHASE4, COPIL-Synthese |
| Confidentialité | Interne / Direction |

---

## 0. Résumé exécutif (1 page)

### 0.1 Pourquoi cette révision ?

Quatre paramètres structurants modifient l'architecture, la conformité, le coût et le mode d'exécution du programme :

1. **Géographie & secteur** — déploiement dans une banque marocaine. Le cadre réglementaire bascule de RGPD/eIDAS (UE) vers **Loi 09-08 (CNDP) + Loi 43-20 (signature électronique) + circulaires Bank Al-Maghrib + DNSSI/DGSSI**.
2. **Plateforme de signature électronique interne** — la banque dispose déjà d'une plateforme de signature. Universign + HSM Thales + intégration PSCo externe **sont supprimés**. Intégration via **API REST avec la plateforme interne**.
3. **Plateforme de gestion documentaire (DMS) interne** — la banque dispose déjà d'une GED/archivage. S3 + KMS + politique de cycle de vie **sont supprimés** pour l'archivage long terme. Intégration via **API REST avec la DMS**.
4. **Mode d'exécution solo + Claude** — un seul développeur (le commanditaire) assisté par Claude. **L'équipe de 18 mois (8-12 ETP) est supprimée**. Budget exprimé en **JH (jours-homme)** et **MAD (dirhams)**.

### 0.2 Impact financier global

| Indicateur | Avant (UE / équipe 12 ETP) | Après (Maroc / solo + Claude) | Variation |
|---|---|---|---|
| **CAPEX Build** | 1,688 M€ ≈ 18,2 M MAD | ~250–320 K MAD (cash) + 280 JH | **-98 % cash** |
| **OPEX Run /an** | 576 K€ ≈ 6,2 M MAD | ~85–110 K MAD/an | **-98 %** |
| **TCO 5 ans** | 4,57 M€ ≈ 49,4 M MAD | ~700–880 K MAD + 280 JH effort | **-98 %** |
| **Effort total** | ~2 200 JH équipe | **280 JH solo** (assistance Claude) | -87 % |
| **Calendrier exécution** | 44 semaines / 18 mois | **52–78 semaines part-time** (12–18 mois) | équivalent |
| **ROI break-even** | 3,2 ans | **0,7 an** (TCO effondré) | -78 % |

**Ce que ça veut dire** : ce qui était un programme à 4,5 M€ devient un projet personnel outillé à ~0,7 M MAD cash. Le risque bascule du financier vers **l'humain (bus factor = 1)** et vers **la qualité d'exécution solo**.

### 0.3 Trois messages COPIL

1. **Le projet devient économiquement trivial mais opérationnellement risqué** — la dépendance à un seul développeur impose des garde-fous : documentation, automatisation, réversibilité, sponsorship banque.
2. **La conformité reste exigeante** — Loi 09-08 + Loi 43-20 + circulaires BAM + DNSSI sont aussi contraignantes que RGPD/eIDAS, avec moins de jurisprudence et un régulateur (CNDP, DGSSI, BAM) actif.
3. **L'externalisation à des plateformes internes (e-sign, DMS) déporte le risque de conformité** — la valeur du parapheur dépend de l'agrément des plateformes amont (PSCo agréé DGSSI ? DMS conforme circulaire BAM 5/W/2017 archivage ?). À auditer avant kickoff.

---

## 1. Nouveau cadre réglementaire (révision Phase 3 — Conformité)

### 1.1 Ce qui change

| Domaine | Phase 3 initiale (UE) | Phase 3 révisée (Maroc banque) |
|---|---|---|
| **Protection des données** | RGPD (UE 2016/679) | **Loi 09-08** + **CNDP** (Commission Nationale de Contrôle de la Protection des Données à Caractère Personnel) |
| **Transferts hors pays** | Schrems II / SCC / TIA | **Article 43 Loi 09-08** : autorisation préalable CNDP pour transfert hors Maroc |
| **Signature électronique** | eIDAS (UE 910/2014) — SES/AdES/QES | **Loi 43-20** sur les services de confiance pour transactions électroniques (août 2020) — **SES, SEA, SEQ** |
| **Tiers de confiance** | PSCo agréé eIDAS (Universign, DocuSign, etc.) | **PSCo agréé DGSSI** (Direction Générale de la Sécurité des Systèmes d'Information) — Barid eSign, Naps, autres prestataires nationaux |
| **Sécurité SI bancaire** | DORA (UE 2022/2554) | **Circulaire BAM 03/W/2016** (gouvernance SI) + **Circulaire BAM 5/W/2017** (continuité d'activité) + **DNSSI** (Directive Nationale Sécurité SI 2014) |
| **Cybersécurité** | NIS2 | **Loi 05-20** (cybersécurité, juillet 2020) + obligations DGSSI pour OIV (Opérateurs d'Importance Vitale, dont banques) |
| **Cloud** | Politique nationale par État | **Directive BAM hébergement & cloud** (souveraineté, localisation données sensibles au Maroc) |
| **Preuve électronique** | Code civil UE | **Loi 53-05** (loi-cadre commerce électronique, abrogée par 43-20) + **Code de commerce 15-95** + **Dahir Obligations & Contrats (DOC)** art. 417-1 et 417-2 |
| **Banque cotée** | MAR / MiFID | **AMMC** (Autorité Marocaine du Marché des Capitaux) si banque cotée |
| **Lutte anti-blanchiment** | AMLD UE | **Loi 43-05** (LAB-FT) + **UTRF** (Unité de Traitement du Renseignement Financier) |
| **Archivage légal** | Code de commerce UE — 10 ans | **Code de commerce marocain art. 22** : 10 ans pour livres et correspondances commerciales |

### 1.2 Loi 09-08 — équivalent CNDP/RGPD : exigences clés

| Article | Exigence | Implémentation |
|---|---|---|
| **Art. 12** | Déclaration préalable CNDP de tout traitement (sauf exemptions) | Dépôt déclaration via portail CNDP avant mise en production |
| **Art. 4-7** | Information personnes concernées (collecte loyale, finalité explicite) | Mention d'information dans tous formulaires + politique confidentialité |
| **Art. 8-9** | Consentement de la personne concernée (sauf bases légitimes : exécution contrat, obligation légale) | Bases légales documentées par traitement (cf. registre) |
| **Art. 7** | Données sensibles (santé, religion, opinion, race, syndicale) — autorisation expresse CNDP | À éviter : aucune donnée sensible dans le parapheur |
| **Art. 23-26** | Droits : accès, rectification, opposition, suppression | Procédures + délai 30 jours + journal d'exercice des droits |
| **Art. 23 al. 2** | Délai de réponse 30 jours (vs 1 mois RGPD ≈ identique) | Workflow CRM/ticketing |
| **Art. 51-55** | Sanctions : 10 000 à 300 000 MAD (≈ 925–28 000 €) + emprisonnement 3 mois–1 an | Inclus dans analyse risque |
| **Art. 43** | Transfert hors Maroc : autorisation préalable CNDP **sauf** liste pays adéquats (UE adéquat depuis 2009) | **Si cloud hors Maroc** → demande autorisation CNDP préalable (8–12 sem. délai) |
| **Décret 2-09-165** | Modalités d'application | RAS |
| **Délibération 478-2013** | Sécurité des données | Mesures techniques équivalentes ISO 27001 + DNSSI |

**Différences notables vs RGPD** :
- **Pas de DPO obligatoire** (mais recommandé) ; la CNDP exige un correspondant
- **Pas de notification 72h** des violations (mais la circulaire BAM impose une notification BAM 24h pour incident SI majeur)
- **Sanctions plus faibles** que RGPD (300 K MAD vs 4 % CA) mais **prison possible**
- **Approche déclarative** vs accountability RGPD : déclaration préalable au lieu de registre + DPIA

### 1.3 Loi 43-20 sur la signature électronique — exigences clés

La loi 43-20 (août 2020, abrogeant la loi 53-05 de 2007) reconnaît trois niveaux de signature électronique alignés sur eIDAS :

| Niveau Loi 43-20 | Équivalent eIDAS | Force probante (DOC art. 417-1) | Cas d'usage parapheur |
|---|---|---|---|
| **Signature Électronique Simple (SES)** | SES | Recevable mais à apprécier par le juge | Délégations internes, accusés réception, validations < 50 K MAD |
| **Signature Électronique Avancée (SEA)** | AdES | Recevable, force probante | Décisions internes, ordres de mission, validations 50 K – 5 M MAD |
| **Signature Électronique Qualifiée (SEQ)** | QES | **Équivalente à signature manuscrite** (art. 417-1 DOC) — présomption de fiabilité | Engagements externes, contrats clients, > 5 M MAD, actes notariés |

**Conditions SEQ** :
- PSCo agréé par la **DGSSI** (la DGSSI tient le registre des PSCo agréés Maroc)
- Certificat qualifié sur dispositif sécurisé (carte à puce, HSM, app mobile certifiée FIDO2)
- Identification renforcée du signataire (face-to-face ou équivalent)
- Horodatage qualifié (TSA agréée)

**PSCo connus agréés DGSSI au Maroc (à confirmer registre 2026)** :
- **Barid eSign** (Poste Maroc / Barid Al-Maghrib) — signature qualifiée + horodatage
- **Maroc Telecom Trust Services**
- **Inwi Trust** (à vérifier)

→ La plateforme interne de la banque est-elle **agréée DGSSI** comme PSCo ? Sinon : la signature qu'elle produit est **au mieux SEA** (force probante mais pas équivalence manuscrite). **Décision COPIL D-R1**.

### 1.4 Bank Al-Maghrib — circulaires applicables

| Circulaire | Objet | Impact parapheur |
|---|---|---|
| **03/W/2016** (gouvernance SI) | Gouvernance, politique SSI, comité, externalisation | Le parapheur doit être inscrit dans la cartographie SI ; comité SSI valide la mise en production ; contrat SLA si externalisation |
| **5/W/2017** (PCA / continuité) | Plan de continuité d'activité, RTO/RPO, tests annuels | RTO ≤ 4h, RPO ≤ 1h, test PRA annuel obligatoire |
| **2/G/2012** (contrôle interne) | Cartographie risques, contrôles permanents | Parapheur doit être dans la cartographie risques opérationnels |
| **Directive cloud BAM** (à vérifier 2026) | Localisation données, audit, réversibilité | **Probable obligation hébergement Maroc** pour données clients sensibles |
| **DGSSI/DNSSI** | Directive Nationale Sécurité SI — 11 chapitres, ~120 mesures | Référentiel de sécurité obligatoire OIV. ISO 27001 reste recommandé en complément. |
| **Circulaire 1/G/2010** (LAB-FT) | KYC, déclarations soupçon | Identité signataires tracée + horodatée |

**À retenir** : la conformité bancaire marocaine impose un référentiel sécurité au moins équivalent à ISO 27001 + DNSSI + tests PRA annuels + audit annuel par cabinet inscrit BAM.

### 1.5 Hébergement & souveraineté des données

| Option | Cadre légal | Coût indicatif (solo) | Recommandation |
|---|---|---|---|
| **AWS Paris (eu-west-3)** | Transfert UE — autorisation CNDP requise (UE pays adéquat → procédure simplifiée) | ~5 000 MAD/mois | Possible si banque l'a déjà autorisé pour d'autres SI ; attente directive cloud BAM |
| **AWS Bahreïn (me-south-1)** | Transfert hors UE — autorisation CNDP nominative | ~5 000 MAD/mois | À éviter : pas adéquat |
| **OVH Casablanca / Maroc Datacenter / N+ONE** | Hébergement national | ~6 000–9 000 MAD/mois | **Recommandé** — souveraineté + simplicité conformité |
| **Cloud privé interne banque** | Hébergement banque | Coût refacturé interne | **Optimal** si la banque l'autorise — réduit coûts cash + élimine flux externe |

**Recommandation** : prioriser **hébergement interne banque** (option 4) si possible, sinon **datacenter Maroc certifié Tier III+** (option 3).

### 1.6 Récapitulatif décisions COPIL conformité (révisées)

| ID | Décision | Avant | Après |
|---|---|---|---|
| **D1** | Niveau signature pour engagements > 50 K€ | QES Universign | **SEQ Loi 43-20** via plateforme interne **si agréée DGSSI**, sinon SEA + déclaration de risque |
| **D2** | Hébergement | OVH SecNumCloud + AWS Paris | **Cloud interne banque** ou **datacenter Maroc Tier III+** |
| **D3** | HSM | Thales Luna 7 EAL4+ | **Supprimé** — délégué à plateforme e-sign interne |
| **D4** | Pentest + bug bounty | Pentest annuel + BB continu | **Pentest annuel par cabinet inscrit BAM** ; bug bounty optionnel |
| **D5** | DPO / correspondant CNDP | DPO interne | **Correspondant CNDP** rattaché à la conformité banque |
| **D6** | Déclaration CNDP | Registre RGPD art. 30 | **Déclaration CNDP préalable** + dossier autorisation transfert si cloud non-Maroc |
| **D7** | Notification incident | RGPD 72h | **BAM 24h** (incident SI majeur) + CNDP si données personnelles compromises |

---

## 2. Architecture révisée (révision Phase 4 — Architecture & Intégration)

### 2.1 Ce qui change

| Composant | Phase 4 initiale | Phase 4 révisée |
|---|---|---|
| **Signature électronique** | MS-5 Signature + HSM + Universign (PSCo) | **API REST → plateforme e-sign interne banque** |
| **Archivage long terme** | S3 + Glacier + KMS + politique cycle de vie | **API REST → DMS interne banque** |
| **HSM** | Thales Luna 7 dédié | **Supprimé** (intégré à la plateforme e-sign) |
| **PSCo externe** | Universign | **Supprimé** |
| **Stockage actif** | PostgreSQL + S3 chiffré (workflow) | PostgreSQL + S3 court terme **OU** stockage objet interne banque |
| **Bus événementiel** | Kafka | Kafka **ou** RabbitMQ (selon disponibilité interne) |
| **Authentification** | Okta SSO | **AD/LDAP banque** + **MFA OTP/SMS** interne |
| **Bus d'intégration** | Kafka Connect + ESB | API Gateway interne + REST + queues internes |
| **Microservices** | 8 microservices Java/Spring | **Réduit à 5–6** (suppression MS Signature dédié, MS Archivage dédié) |
| **Cloud** | AWS Paris + DR Frankfurt | Cloud interne banque ou Maroc DC |

### 2.2 Architecture cible révisée — vue C4 niveau 2

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UTILISATEURS (Web / Mobile)                       │
└──────────────────────┬──────────────────────────────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │     Frontend React (PWA)          │
        │     Auth: AD/LDAP + MFA           │
        └──────────────┬───────────────────┘
                       │
                       ▼
        ┌──────────────────────────────────┐
        │   API Gateway (Kong / NGINX)      │
        │   AuthZ, rate-limit, audit log    │
        └──────────────┬───────────────────┘
                       │
       ┌───────────────┼────────────────────────────┐
       ▼               ▼              ▼             ▼
  ┌─────────┐  ┌─────────────┐  ┌────────────┐  ┌────────────┐
  │ MS-1    │  │ MS-2        │  │ MS-3       │  │ MS-4       │
  │ Dossier │  │ Workflow    │  │ Notification│  │ Audit      │
  │         │  │ (Camunda /  │  │            │  │            │
  │         │  │  custom)    │  │            │  │            │
  └────┬────┘  └──────┬──────┘  └─────┬──────┘  └─────┬──────┘
       │              │                 │                │
       └──────────────┼─────────────────┘                │
                      │                                   │
       ┌──────────────┼───────────────────┐              │
       ▼              ▼                    ▼              ▼
  ┌────────┐    ┌──────────┐       ┌──────────┐    ┌─────────┐
  │ PG 16  │    │ Cache    │       │ MS-5     │    │ MS-6    │
  │ (état  │    │ Redis    │       │ Adapter  │    │ Adapter │
  │ workflow)   │          │       │ e-Sign   │    │ DMS     │
  └────────┘    └──────────┘       └────┬─────┘    └────┬────┘
                                         │                │
                                         ▼                ▼
                              ┌──────────────────┐  ┌──────────────────┐
                              │ PLATEFORME       │  │ PLATEFORME GED   │
                              │ E-SIGN INTERNE   │  │ INTERNE BANQUE   │
                              │ (API REST)       │  │ (API REST)       │
                              │ - HSM            │  │ - Archivage      │
                              │ - PSCo / certif  │  │   légal 10 ans   │
                              │ - Horodatage     │  │ - Cycle de vie   │
                              └──────────────────┘  └──────────────────┘
                                         │
                                         │ (intégrations sortantes)
                                         ▼
                              ┌──────────────────┐
                              │ AD / LDAP        │
                              │ Core banking     │
                              │ Email (SMTP)     │
                              │ SMS Gateway      │
                              └──────────────────┘
```

### 2.3 Microservices révisés (5 au lieu de 8)

| MS | Nom | Responsabilité | Effort solo (JH) |
|---|---|---|---|
| **MS-1** | Dossier | CRUD dossier, métadonnées, recherche | 25 |
| **MS-2** | Workflow | Moteur règles + orchestration états (BPMN simplifié ou tables d'état + Camunda Embedded) | 40 |
| **MS-3** | Notification | Email/SMS/push, templates, tracking | 12 |
| **MS-4** | Audit | Journal d'événements immuable, recherche, export | 15 |
| **MS-5** | Adapter e-Sign | Façade API plateforme interne (envoi à signer, callback, vérification) | 18 |
| **MS-6** | Adapter DMS | Façade API GED interne (dépôt archivage, récupération, métadonnées) | 12 |
| (Frontend) | Web React + PWA mobile | UI complète | 60 |
| (Infra) | IaC Terraform + CI/CD GitHub Actions | Pipeline + secrets + monitoring | 20 |
| **Total dev** | | | **202 JH** |

**Note** : Camunda BPM peut être remplacé par un moteur d'état simple (tables PostgreSQL + machine à états) en mode solo, économisant ~10 JH d'apprentissage et 0 € de licence. **Décision technique** : statemachine custom léger.

### 2.4 Spécifications API plateforme e-signature interne (à confirmer banque)

À demander à la DSI banque (questionnaire kickoff) :

| Question | Format réponse attendu |
|---|---|
| URL endpoint API | https://esign.banque.ma/api/v1/ |
| Protocole | REST/JSON ou SOAP/XML ? |
| Authentification | OAuth2 client_credentials / mTLS / API key ? |
| Niveaux de signature supportés | SES / SEA / SEQ Loi 43-20 ? |
| Agrément DGSSI | Oui/non + ID registre |
| Format signature | PAdES / XAdES / CAdES ? PAdES-LTA pour archivage long ? |
| Horodatage | TSA RFC 3161 intégrée ? |
| Modèle synchrone/asynchrone | Sync (réponse immédiate) ou async (webhook callback) ? |
| Rate-limit | Req/sec / req/jour |
| SLA disponibilité | 99,5 % ? 99,9 % ? |
| Audit / journal | API d'export journal des signatures ? |
| Vérification a posteriori | API "verify" sur document signé ? |
| Coût refacturation interne | MAD par signature ? Forfait mensuel ? |
| Certificats utilisateurs | Carte CNIE + lecteur ? Mobile app ? OTP SMS ? |
| Documentation Swagger | Lien |
| Environnement de test (sandbox) | URL + credentials |

**Cas générique d'API e-signature (modèle de référence)** :

```
POST /api/v1/sign-requests
Authorization: Bearer {token}
Content-Type: application/json
{
  "documentId": "uuid",
  "documentBase64": "...",
  "documentName": "facture-2026-001.pdf",
  "signatureLevel": "SEA",  // SES | SEA | SEQ
  "signers": [
    { "userId": "matricule123", "email": "...", "order": 1, "role": "approver" }
  ],
  "callbackUrl": "https://parapheur.banque.ma/api/v1/esign/callback",
  "metadata": { "dossierId": "DOS-2026-001234", "engagement": 250000 }
}
→ 202 Accepted
{ "signRequestId": "uuid", "status": "PENDING", "expiresAt": "..." }

POST /api/v1/esign/callback (par plateforme e-sign)
{
  "signRequestId": "uuid",
  "status": "SIGNED" | "REJECTED" | "EXPIRED",
  "signedDocumentBase64": "...",
  "signatureCertificate": "...",
  "timestamp": "..."
}
```

### 2.5 Spécifications API plateforme GED / DMS interne

```
POST /api/v1/documents
{
  "name": "DOS-2026-001234-signe.pdf",
  "category": "engagement-bancaire",
  "retentionPolicy": "10-years-commercial",
  "metadata": { "dossierId": "...", "type": "facture", "amount": 250000, "currency": "MAD" },
  "fileBase64": "..."
}
→ 201 Created
{ "documentId": "uuid", "url": "...", "archivedAt": "..." }

GET /api/v1/documents/{documentId}
→ 200 OK + document binaire

GET /api/v1/documents?dossierId=...
→ 200 OK + liste

POST /api/v1/documents/{documentId}/legal-hold
→ blocage purge
```

**À demander à la DSI** :
- Conformité circulaire BAM 5/W/2017 (PCA) et Loi 09-08 archivage ?
- Politique de rétention paramétrable ?
- Audit log des accès ?
- Quotas / coût ?
- Format métadonnées (Dublin Core ? schéma maison ?)
- Versioning ?

### 2.6 Décisions architecture COPIL révisées

| ID | Décision | Avant | Après |
|---|---|---|---|
| **D8** | Modèle d'intégration SAP / core banking | Event-driven Kafka Connect | **REST synchrone** (volumétrie 12 K/an = 50/jour, pas besoin de stream) |
| **D9** | Stratégie de release | Blue/Green Canary EKS | **Rolling deploy simple** (Kubernetes basic ou conteneurs Docker Swarm / VM) |
| **D10** | Migration historique | 5 K dossiers actifs uniquement | **Pas de migration** — démarrage greenfield + double saisie 3 mois |
| **D11** | DR | Active/Passive cross-region AWS | **Sauvegarde quotidienne** + restore < 24h (suffisant en mode solo) |
| **D12** | API publique externe | API privée (interne uniquement) | **API interne uniquement** — pas d'exposition externe |
| **D13** | Hébergement | AWS Paris + DR Frankfurt | **Cloud interne banque** (préféré) ou datacenter Maroc |
| **D14** | Stockage actif | S3 + KMS | **PostgreSQL JSONB + bucket S3-compatible interne** ou **MinIO on-prem** |
| **D15** | Bus événementiel | Kafka 3.7 | **PostgreSQL LISTEN/NOTIFY** ou **RabbitMQ léger** (Kafka surdimensionné en solo) |
| **D16** | Observabilité | Prometheus + Grafana + Loki + Tempo (stack complète) | **Prometheus + Grafana + Loki** (Tempo retiré, traces = nice to have) |
| **D17** | Authentification | Okta SSO + WebAuthn FIDO2 | **AD banque + MFA OTP** existant (pas de nouvel IDP) |

---

## 3. Impacts sur Phase 1 et Phase 2

### 3.1 Phase 1 — Spécifications détaillées : impact mineur

Les 47 user stories restent valides à 95 %. Modifications :

| US | Modification |
|---|---|
| US-15 (signer document) | Wording adapté : "envoi à plateforme e-sign interne" au lieu de "signature locale" |
| US-22 (archiver dossier) | Wording adapté : "appel API DMS" au lieu de "S3 archive" |
| US-31 (notification SMS) | Maintenir, via SMS Gateway interne banque |
| US-09 (login) | AD banque + MFA OTP (pas Okta) |
| US-12 (afficher montants) | **Devise par défaut MAD** + multi-devise (EUR, USD pour international) |
| US-18 (générer PDF) | PDF/A-1b (norme archivage) — confirmer compatible avec DMS |
| US-25 (export Excel) | Encoding UTF-8 + entêtes en français |
| US-40 (langue UI) | **Français + arabe (RTL)** — nouveau besoin Maroc |
| US-44 (calendrier) | Calendrier grégorien + jours fériés Maroc + week-end **vendredi-samedi** ou **samedi-dimanche** selon politique banque |

**Effort additionnel** : ~8 JH (i18n FR/AR + RTL + adaptations devise).

### 3.2 Phase 2 — Spécifications métier : impact moyen

Les 87 règles DMN restent structurellement valides. Modifications :

| Règle DMN | Modification |
|---|---|
| **DMN-01 à 05** (seuils délégation) | Conversion en MAD : seuils 50 K€ → 550 K MAD ; 200 K€ → 2,2 M MAD ; 1 M€ → 11 M MAD (à confirmer politique banque) |
| **DMN-12 à 18** (TVA) | TVA Maroc : 20 % standard, 14 %, 10 %, 7 % réduits — à reparamétrer |
| **DMN-21** (devise) | Multi-devise : MAD pivot + conversion taux BAM jour |
| **DMN-30 à 35** (signature requise) | Niveau signature mappé sur Loi 43-20 (SES/SEA/SEQ) au lieu d'eIDAS |
| **DMN-50 à 60** (workflow approbation) | Hiérarchie banque + comité de crédit + commission délégation |
| **DMN-70 à 75** (documents requis) | Documents marocains : CIN/CNIE, registre commerce, ICE, IF, CNSS |

**Effort additionnel** : ~12 JH (re-paramétrage règles + tests).

### 3.3 Tableau consolidé impact par phase

| Phase | Pages initiales | Modifications | Effort additionnel |
|---|---|---|---|
| Phase 1 — Spécifications détaillées | 105 | i18n FR/AR, devise MAD, calendrier Maroc | 8 JH |
| Phase 2 — Spécifications métier | 110 | Règles DMN MAD, TVA Maroc, hiérarchie banque, documents marocains | 12 JH |
| Phase 3 — Conformité & sécurité | 85 | **Refonte majeure** : RGPD→09-08, eIDAS→43-20, BAM, DGSSI/DNSSI | 18 JH (analyse + dépôt CNDP + audit interne) |
| Phase 4 — Architecture & intégration | 95 | E-sign et DMS via API internes, microservices réduits, cloud interne | 16 JH (révision archi + spec API) |
| **Total révision phases** | **395** | | **54 JH** |

---

## 4. Mode d'exécution solo + Claude

### 4.1 Méthodologie

**Principe** : un développeur unique pilote la totalité du programme avec Claude comme paire de programmation, analyste et générateur de code. Pas d'équipe humaine sous-jacente.

**Forces** :
- Productivité multipliée (Claude génère 70–80 % du code, le dev valide, teste, ajuste)
- Coût cash quasi nul
- Pas de coordination, pas de meetings, pas de Gantt complexe
- Itération rapide

**Faiblesses** :
- **Bus factor = 1** — si le dev est indisponible (maladie, départ), le projet stoppe
- Pas de revue de code par un pair humain (mitigé par Claude + outils statiques)
- Risque de dérive qualitative sans regard externe
- Charge mentale forte sur un seul individu
- Pas de séparation des préoccupations (dev, archi, sécurité, conformité = même personne)

**Garde-fous obligatoires** :
1. **Tout en code** (Infrastructure as Code, configuration as code, tests as code, doc as code)
2. **Couverture tests automatisés ≥ 80 %**
3. **Revue automatisée** : SonarQube ou équivalent + Snyk + Trivy
4. **Documentation continue** : ADR (Architecture Decision Records) à chaque décision
5. **Reproductibilité** : `make` + Docker Compose + Terraform — clone + 1 commande = environnement fonctionnel
6. **Sponsorship banque** : un référent banque (RSSI ou DSI) en backup conformité + accès comptes/secrets
7. **Audit externe annuel** par cabinet inscrit BAM (obligatoire de toute façon)

### 4.2 Pile technique simplifiée pour solo

| Composant | Choix techno | Rationale solo |
|---|---|---|
| **Frontend** | React 18 + TypeScript + Vite + TailwindCSS + shadcn/ui | Stack mainstream, écosystème mature, Claude code rapide |
| **Backend** | Node.js 20 + Fastify + TypeScript **OU** Python 3.12 + FastAPI | TypeScript préféré (mêmes types front/back, productivité solo). Plus léger que Java/Spring (gain ~30 % JH) |
| **DB** | PostgreSQL 16 | Suffit pour 12 K dossiers/an. JSONB pour flexibilité métadonnées |
| **Workflow** | State machine custom (XState côté front, table d'états côté back) | Pas de Camunda — surdimensionné solo |
| **Cache/Queue** | Redis 7 (cache + queue avec BullMQ) | Léger, multi-usage |
| **Fichiers actifs** | MinIO ou S3-compatible interne | Standard, simple |
| **Auth** | Passport.js + AD banque + OTP | Pas de licence Okta |
| **Tests** | Vitest + Playwright + k6 | Stack JS unifié |
| **CI/CD** | GitHub Actions (ou GitLab CI banque) | Gratuit jusqu'à un volume confortable |
| **IaC** | Terraform + Docker Compose (dev) + Helm (prod si k8s banque) | Standard |
| **Observabilité** | Prometheus + Grafana + Loki | Open source, suffisant |
| **Erreurs** | Sentry self-hosted | Gratuit |

**Différence vs Phase 4 (Java + Spring + Kafka)** : –30 à –40 % d'effort de développement, pas de licences. Stack TypeScript unifiée full-stack.

### 4.3 Workflow Claude — pratiques recommandées

| Phase tâche | Usage Claude | Validation humaine |
|---|---|---|
| **Spec → user story affinée** | Claude rédige les acceptance criteria | Lecture + critique |
| **US → schéma DB** | Claude propose migrations Prisma/Drizzle/Knex | Revue + tests |
| **US → endpoint API** | Claude génère controller + service + DTO + tests unitaires | Revue + tests intégration |
| **US → composant React** | Claude génère composant + Storybook + tests Vitest | Revue + tests E2E |
| **Bug** | Claude analyse stack + propose correctif | Tests régression |
| **Conformité** | Claude rédige ADR + checklist 09-08 / 43-20 | Validation correspondant CNDP banque |
| **Doc** | Claude génère README + OpenAPI + diagrammes Mermaid | Relecture |

**Productivité observée (benchmarks publics 2025–2026)** : un développeur senior + Claude délivre 2–3× plus de code par jour qu'un développeur seul, avec qualité comparable à condition de revue systématique.

### 4.4 Effort total estimé en JH solo

| Lot | Description | JH solo + Claude |
|---|---|---|
| **Setup** | IaC, CI/CD, environnements dev/staging/prod, monitoring | 15 |
| **Backend** | 5 microservices + auth + audit | 100 |
| **Frontend** | Web React + PWA mobile + i18n FR/AR | 60 |
| **Intégrations** | API e-sign, API DMS, AD/LDAP, SMTP, SMS, core banking (REST) | 30 |
| **Données** | Schémas, migrations, seed, tests | 12 |
| **Tests** | Unit + intégration + E2E + perf + sécurité | 25 |
| **Conformité** | DPIA équivalent CNDP, déclarations, ADR, doc sécurité, audit interne | 20 |
| **Recette** | UAT avec utilisateurs banque (8 sessions × 0,5 JH solo) | 4 |
| **Documentation** | OpenAPI, runbook, manuel utilisateur, formations | 10 |
| **Mise en production** | Déploiement, hypercare 4 semaines | 14 |
| **Total Build** | | **290 JH** |
| **Run /an** | TMA + évolutions + monitoring + audit annuel | **45 JH/an** |

**Calendrier** :
- À temps plein (220 JH/an utiles) : ~16 mois de Build
- À temps partiel (50 % capacité = 110 JH/an) : ~32 mois de Build → **non recommandé**
- **Recommandation** : full-time 14–18 mois, hypercare 4 semaines, puis Run à temps partiel

### 4.5 Risques spécifiques au mode solo

| ID | Risque | P×I | Mitigation |
|---|---|---|---|
| **RS-01** | Indisponibilité dev (maladie, départ) | 3×5 = 15 | Sponsor banque + backup interne + doc exhaustive + repository GitHub banque |
| **RS-02** | Dérive qualitative / dette technique | 4×3 = 12 | Tests automatisés ≥ 80 %, SonarQube, audit externe semestriel |
| **RS-03** | Erreur conformité non détectée | 2×5 = 10 | Audit cabinet inscrit BAM + correspondant CNDP banque |
| **RS-04** | Charge mentale → burnout | 3×4 = 12 | Cadence soutenable, périodes off, automatisation maximale |
| **RS-05** | Dépendance Claude (panne API, changement pricing) | 2×3 = 6 | Code reste utilisable sans Claude ; alternatives Cursor/Copilot/Codeium |
| **RS-06** | Sous-estimation effort par excès d'optimisme | 4×4 = 16 | Buffer 20 % + jalons tous les 2 semaines + replanification trimestrielle |
| **RS-07** | API e-sign ou DMS non documentée / non fiable | 3×4 = 12 | **Phase 0 — kickoff** : 5 JH d'investigation des API banque AVANT engagement |
| **RS-08** | CNDP refuse traitement / hébergement | 2×5 = 10 | Pré-consultation CNDP avant build (gratuite) |
| **RS-09** | Plateforme e-sign banque non agréée DGSSI → SEQ impossible | 3×4 = 12 | Audit agrément en kickoff ; plan B = SEA + déclaration risque ou PSCo externe Barid eSign |
| **RS-10** | Banque change de stratégie / sponsor part | 3×5 = 15 | Cadrage écrit + lettre de mission signée + jalons formels |

---

## 5. Budget révisé en MAD et JH

### 5.1 Hypothèses

- Taux de change : **1 EUR = 10,8 MAD** (avril 2026, indicatif)
- TJM développeur senior Maroc : **3 500 MAD** (référence si valorisation du temps requise par DAF)
- TJM consultant audit/conformité : **6 000 MAD**
- TJM auditeur cabinet inscrit BAM : **8 000 MAD/jour**
- Coût Claude (Pro/API) : **400 MAD/mois** (~37 €)
- Cloud (option datacenter Maroc Tier III+) : **6 000 MAD/mois**

### 5.2 CAPEX Build (cash, hors valorisation temps dev)

| Poste | Détail | MAD |
|---|---|---|
| **Pré-étude & kickoff** | Audit API e-sign + DMS banque, pré-consultation CNDP | 10 000 |
| **Licences logicielles** | Aucune (open source 100 %) | 0 |
| **Domaine + certificats SSL** | parapheur.banque.ma + certif Let's Encrypt | 1 500 |
| **Outillage dev** | GitHub (banque), JetBrains licence solo, Figma solo | 8 000 |
| **Cloud setup** | Provisionning environnements dev/staging/prod (3 mois) | 18 000 |
| **Claude Pro + API** | 14 mois × 400 | 5 600 |
| **Audit conformité externe** | Cabinet inscrit BAM, ~5 jours × 8 000 | 40 000 |
| **Audit pentest** | Cabinet pentest agréé, ~8 jours × 6 000 | 48 000 |
| **Frais CNDP** | Déclaration + autorisation transfert si applicable | 5 000 |
| **Formation utilisateurs** | Supports + 4 sessions onsite | 12 000 |
| **Buffer / aléas (15 %)** | | 22 000 |
| **TOTAL CAPEX cash** | | **170 100 MAD** |
| | ≈ 15 750 € | |

**Si valorisation temps dev (TJM 3 500)** :
- 290 JH × 3 500 = **1 015 000 MAD** ≈ 94 K€
- **CAPEX total avec temps valorisé** : 170 100 + 1 015 000 = **1 185 100 MAD** ≈ 110 K€

### 5.3 OPEX Run /an

| Poste | Détail | MAD/an |
|---|---|---|
| Cloud | 6 000 × 12 | 72 000 |
| Claude Pro/API | 400 × 12 | 4 800 |
| Domaine + certificats | renouvellement | 1 500 |
| Outillage | renouvellement licences | 8 000 |
| Audit annuel cabinet BAM | 5 jours × 8 000 | 40 000 |
| Pentest annuel | 5 jours × 6 000 | 30 000 |
| Renouvellement déclaration CNDP | si changement matériel | 0–5 000 |
| Sentry, monitoring tiers | self-host gratuit, ou Sentry hosted entry plan | 6 000 |
| **OPEX Run cash /an** | | **162 300 MAD** |
| | ≈ 15 030 € | |

**Si valorisation temps TMA solo** :
- 45 JH/an × 3 500 = **157 500 MAD/an** ≈ 14,6 K€
- **OPEX total /an avec temps valorisé** : 162 300 + 157 500 = **319 800 MAD/an** ≈ 29,6 K€

### 5.4 TCO 5 ans

| Modalité | Build cash | Run cash 5 ans | Total cash | + Temps valorisé | Total |
|---|---|---|---|---|---|
| **Cash uniquement** | 170 K MAD | 812 K MAD | **982 K MAD** ≈ 91 K€ | – | 982 K MAD |
| **Avec temps valorisé** | 1 185 K MAD | 1 599 K MAD | – | – | **2 784 K MAD** ≈ 258 K€ |

**Comparaison vs initial** :
- TCO 5 ans initial : **49,4 M MAD** (≈ 4,57 M€)
- TCO 5 ans révisé cash : **0,98 M MAD** → **-98 %**
- TCO 5 ans révisé avec temps : **2,78 M MAD** → **-94 %**

### 5.5 ROI révisé

Bénéfices annuels (inchangés vs Phase 2 — synthèse économique) :

| Bénéfice | Calcul | MAD/an |
|---|---|---|
| Gain temps signataires | 12 K dossiers × 0,8 h × 250 MAD/h moyen | 2 400 000 |
| Réduction délais (cash conversion cycle) | 285 M€ × 0,5 % × 10,8 | 15 390 000 |
| Réduction litiges & pertes | estimation prudente | 800 000 |
| Réduction papier / logistique | 12 K dossiers × 80 MAD | 960 000 |
| **Total bénéfices /an** | | **19 550 000 MAD** ≈ 1,8 M€ |

**Note** : les bénéfices restent élevés car ils dépendent du volume traité et du capital engagé, pas du coût IT.

| Indicateur | Valeur |
|---|---|
| **TCO an 1** (cash) | 332 K MAD (CAPEX + Run an 1) |
| **Bénéfices an 1** (palier 50 % adoption) | 9 775 K MAD |
| **Bénéfice net an 1** | +9 443 K MAD |
| **Break-even** | **< 3 mois** (vs 3,2 ans initial) |
| **NPV 5 ans (taux 8 %)** | +71 M MAD |
| **IRR 5 ans** | > 1 000 % |

→ **Le business case devient évident**. Le débat n'est plus financier, il est **opérationnel et conformité**.

---

## 6. Planning révisé

### 6.1 Phases réorganisées

| Phase | Activités | Durée | JH effort |
|---|---|---|---|
| **P0 — Kickoff** | Audit API banque (e-sign, DMS, AD, core banking), pré-consultation CNDP, lettre de mission, accès | 4 sem | 15 |
| **P5 — Cadrage opérationnel** | ADR initiaux, setup repo + IaC + CI/CD + monitoring, environnements dev/staging | 4 sem | 25 |
| **P6 — Build vague 1 (MVP)** | MS-1 Dossier + MS-3 Notification + frontend MVP + auth AD + adapters fictifs e-sign/DMS (mocks) | 12 sem | 80 |
| **P7 — Build vague 2** | MS-2 Workflow + MS-5 e-Sign réel + MS-6 DMS réel + i18n FR/AR + règles DMN MAD | 14 sem | 90 |
| **P8 — Build vague 3** | MS-4 Audit + intégrations finales + reporting + perf + sécurité + tests E2E | 8 sem | 50 |
| **P9 — Recette & conformité** | Audit externe BAM, pentest, dépôt CNDP, UAT, formation, hypercare | 6 sem | 30 |
| **P10 — Mise en production & hypercare** | Go-live progressif (10 % → 50 % → 100 %) + hypercare 4 semaines | 6 sem | 25 |
| **TOTAL Build** | | **54 sem** ≈ 12,5 mois | **315 JH** |
| **Run** | TMA + évolutions + monitoring + audits annuels | continu | 45 JH/an |

**Note effort** : 315 JH (vs 290 estimés au § 4.4 + 25 P0) — j'inclus le P0 kickoff dans le total Build pour rigueur.

### 6.2 Jalons clés

| Jalon | Date cible | Livrables |
|---|---|---|
| **J0** | T+0 | Lettre de mission signée, accès, équipe sponsor identifiée |
| **J1** | T+4 sem | Audit API banque livré, ADR socle, environnements provisionnés |
| **J2** | T+20 sem | MVP démontrable (dossier + workflow simple + 1 type signature) |
| **J3** | T+34 sem | Intégration complète e-sign + DMS, i18n FR/AR, règles métier MAD |
| **J4** | T+42 sem | Code complet, tests ≥ 80 %, audit externe BAM en cours |
| **J5** | T+48 sem | Audit OK, dépôt CNDP validé, UAT signée |
| **J6** | T+54 sem | Go-live + hypercare terminé |

### 6.3 Gantt simplifié (54 semaines)

```
Sem.  1───4    5──8    9────────20    21───────34    35─────42    43──48    49──54
P0  ████
P5         ████
P6              ████████████
P7                          ██████████████
P8                                          ████████
P9                                                    ██████
P10                                                              ██████
                                                                   J6▲
```

---

## 7. Conditions de succès révisées

| # | Condition | Détail |
|---|---|---|
| **1** | Sponsor banque actif | DSI ou DG signataire de la lettre de mission, point hebdo 30 min, escalation < 48h |
| **2** | Accès aux API internes garantis | Sandbox e-sign + DMS + AD opérationnels semaine 1, doc Swagger, contacts dédiés |
| **3** | Correspondant CNDP désigné | Validation conformité 09-08, dépôt déclaration sous 8 sem |
| **4** | RSSI banque associé | Validation architecture sécurité + DNSSI + circulaire BAM 03/W/2016 |
| **5** | Cadence soutenable | 4 jours / sem effectifs, vendredi off en repli ; jalons tous les 2 sem ; replanif trimestrielle |
| **6** | Backup humain | 1 référent SI banque accède au repo + comptes ; runbook exhaustif tenu à jour |
| **7** | Tests automatisés ≥ 80 % couverture | CI bloquante en dessous |
| **8** | Audit externe annuel | Cabinet inscrit BAM, calendrier fixé en P9 puis annuel |

---

## 8. Décisions COPIL consolidées (révisées)

Le programme initial listait 17 décisions. Avec le nouveau contexte, elles deviennent **12 décisions** (5 sont supprimées car liées à des choix qui n'existent plus : Universign, HSM Thales, Okta, Kafka, AWS DR Frankfurt).

| ID | Décision | Recommandation |
|---|---|---|
| **D-R1** | Hébergement | Cloud interne banque > datacenter Maroc Tier III+ > AWS Paris (avec autorisation CNDP) |
| **D-R2** | Niveau signature | SEQ Loi 43-20 si plateforme banque agréée DGSSI ; sinon SEA + politique compensatoire |
| **D-R3** | Stack technique | TypeScript full-stack (Node.js + React) — productivité solo +30 % vs Java |
| **D-R4** | Workflow engine | State machine custom léger (pas Camunda) |
| **D-R5** | Authentification | AD banque + MFA OTP (pas Okta, pas FIDO2 obligatoire) |
| **D-R6** | Bus messages | Redis BullMQ + PostgreSQL LISTEN/NOTIFY (pas Kafka) |
| **D-R7** | Migration historique | Pas de migration, démarrage greenfield + double saisie 3 mois |
| **D-R8** | Stratégie release | Rolling deploy simple (pas Blue/Green Canary) |
| **D-R9** | DR / PRA | Sauvegarde quotidienne + restore < 24h (pas Active/Passive multi-région) |
| **D-R10** | Devises | MAD pivot, multi-devise via taux BAM |
| **D-R11** | Langues UI | Français + arabe (RTL) ; pas d'autres langues phase 1 |
| **D-R12** | Sponsorship & gouvernance | Lettre de mission signée DG ou DSI + référent SI backup + correspondant CNDP + RSSI associé |

---

## 9. Plan d'action 30 jours (révisé)

| # | Action | Owner | Livrable | Délai |
|---|---|---|---|---|
| 1 | Lettre de mission signée DG/DSI | Sponsor banque | Document signé | J+5 |
| 2 | Désignation correspondant CNDP banque | Banque conformité | Mail désignation | J+5 |
| 3 | Désignation référent RSSI | Banque RSSI | Mail | J+5 |
| 4 | Audit API plateforme e-sign banque | Dev solo + DSI | Document spec + agrément DGSSI | J+10 |
| 5 | Audit API DMS banque + politique rétention | Dev solo + DSI | Document spec | J+10 |
| 6 | Demande accès AD + comptes services | Dev solo + DSI | Identifiants techniques | J+10 |
| 7 | Pré-consultation CNDP (mail ou rdv) | Dev solo + Conformité | Compte rendu CNDP | J+15 |
| 8 | Choix hébergement (interne ou DC Maroc) | Sponsor + RSSI | ADR + contrat | J+20 |
| 9 | Setup repo GitHub banque + CI/CD + IaC initial | Dev solo | Repo + pipeline vert | J+25 |
| 10 | Validation 12 décisions COPIL révisées | COPIL Direction | PV COPIL | J+30 |

---

## 10. Annexes

### A. Conversion EUR → MAD

Hypothèse : 1 EUR = 10,8 MAD (avril 2026)

| Donnée initiale (EUR) | Équivalent MAD |
|---|---|
| 1 688 K€ CAPEX initial | 18 230 K MAD |
| 576 K€ OPEX initial | 6 220 K MAD |
| 4 570 K€ TCO 5 ans initial | 49 360 K MAD |
| Bénéfices 1,45 M€/an initial | 15 660 K MAD/an |
| Capital engagé 285 M€/an | 3 078 M MAD/an |

### B. Sources réglementaires Maroc

- **Loi 09-08** — protection physique des personnes à l'égard du traitement des données à caractère personnel (18 février 2009)
- **Décret 2-09-165** — modalités d'application loi 09-08
- **CNDP** — www.cndp.ma — registre des traitements, formulaires, jurisprudence
- **Loi 43-20** — services de confiance pour les transactions électroniques (8 août 2020) — abroge loi 53-05
- **Loi 53-05** — échange électronique de données juridiques (2007, abrogée mais référencée pour antériorité)
- **DGSSI** — Direction Générale de la Sécurité des Systèmes d'Information — www.dgssi.gov.ma
- **DNSSI** — Directive Nationale de la Sécurité des Systèmes d'Information (2014, mise à jour périodique)
- **Loi 05-20** — cybersécurité (juillet 2020) + décret d'application
- **Bank Al-Maghrib** — Circulaire 03/W/2016 (gouvernance SI), 5/W/2017 (PCA), 1/G/2010 (LAB-FT), Directive cloud (à vérifier 2026)
- **Code de commerce 15-95** — preuve commerciale, archivage 10 ans
- **DOC** — Dahir des Obligations et Contrats — art. 417-1, 417-2 (preuve électronique)
- **Loi 43-05** — lutte contre le blanchiment de capitaux
- **AMMC** — www.ammc.ma — si banque cotée

### C. Glossaire

- **BAM** : Bank Al-Maghrib (banque centrale)
- **CNDP** : Commission Nationale de Contrôle de la Protection des Données à Caractère Personnel
- **DGSSI** : Direction Générale de la Sécurité des Systèmes d'Information
- **DNSSI** : Directive Nationale de la Sécurité des Systèmes d'Information
- **DOC** : Dahir des Obligations et Contrats (≈ Code civil)
- **CNIE** : Carte Nationale d'Identité Électronique
- **ICE** : Identifiant Commun de l'Entreprise
- **IF** : Identifiant Fiscal
- **CNSS** : Caisse Nationale de Sécurité Sociale
- **OIV** : Opérateur d'Importance Vitale
- **PSCo** : Prestataire de Services de Confiance
- **SES / SEA / SEQ** : Signature Électronique Simple / Avancée / Qualifiée (Loi 43-20)
- **TMA** : Tierce Maintenance Applicative
- **TJM** : Taux Journalier Moyen
- **JH** : Jour-Homme

---

## 11. Demande COPIL révisée

Le COPIL est sollicité pour valider :

1. **GO programme** dans le contexte révisé Maroc / banque / e-sign et DMS internes / mode solo
2. **Les 12 décisions D-R1 à D-R12** ci-dessus
3. **Le mandat solo + sponsorship banque** : lettre de mission + correspondant CNDP + référent RSSI + accès API + budget cash 170 K MAD CAPEX + 162 K MAD/an OPEX
4. **La levée des 4 prérequis bloquants P0** : (a) accès API e-sign opérationnel, (b) accès API DMS opérationnel, (c) hébergement validé, (d) pré-consultation CNDP réalisée

**Calendrier proposé** : signature lettre de mission J+5 ; kickoff P0 J+5 ; J0 build P5 à J+30 ; go-live à J+54 sem.

---

*Fin du document de révision R1.0.*
