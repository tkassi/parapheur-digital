# Phase 3 v2 — Conformité & sécurité

**Programme Parapheur Digital — Banque commerciale universelle marocaine cotée à la Bourse de Casablanca, classée OIV**

| Métadonnée | Valeur |
|---|---|
| Version | R2.0 — Avril 2026 |
| Auteur | Direction de programme |
| Statut | Pour validation COPIL Direction (DG, DSI, RSSI, DPO/Correspondant CNDP, Direction Juridique, Direction Conformité) |
| Documents amendés | Phase 3 v1 (RGPD/eIDAS/ISO 27001 — UE) |
| Confidentialité | Interne / Direction — diffusion restreinte |
| Pages | ~85 |

---

## Note de cadrage

Cette Phase 3 v2 remplace intégralement la Phase 3 v1 pour le contexte d'une **banque commerciale universelle marocaine, cotée à la Bourse de Casablanca, classée Opérateur d'Importance Vitale (OIV)**, opérant un parapheur digital pour ses **engagements et validations internes** (signature de documents externes hors périmètre).

**Ce qui change vs v1** :

| Domaine | v1 (UE) | v2 (Maroc / banque cotée OIV) |
|---|---|---|
| Données personnelles | RGPD (UE 2016/679) | **Loi 09-08** + CNDP |
| Signature électronique | eIDAS (UE 910/2014) | **Loi 43-20** + DGSSI |
| Sécurité SI bancaire | DORA + NIS2 | **DNSSI** + **Loi 05-20** + circulaires BAM 03/W/2016 + 5/W/2017 |
| Cybersécurité OIV | NIS2 entité essentielle | **Loi 05-20** + DGSSI (ex-DGSSI rattaché à l'Administration de la Défense Nationale) |
| Gouvernance société cotée | MAR + MiFID | **AMMC** + Loi 17-95 (sociétés anonymes) + Code Marocain de Bonnes Pratiques de Gouvernance |
| Référentiel sécurité | ISO 27001:2022 | **DNSSI** comme socle obligatoire + ISO 27001:2022 en complément (recommandé, non obligatoire) |
| Tiers de confiance | PSCo eIDAS (Universign) | **PSCo agréé DGSSI** — supprimé : signature via plateforme interne banque |
| HSM dédié | Thales Luna 7 EAL4+ | Supprimé : délégué à plateforme e-sign interne banque |
| Notifications incidents | RGPD 72h + DORA | **BAM 24h** (circulaire 03/W/2016) + **DGSSI 24h** (Loi 05-20) + CNDP (sans délai légal strict) |

**Périmètre fonctionnel** : signature électronique de documents **internes** (notes de décision, validations comité crédit, ordres de mission, délégations, validations RH, validations de dépense, engagements opérationnels). Hors périmètre : contrats clients, actes notariés, documents publics — pas de SEQ critique pour clients externes.

---

## Sommaire

1. Cadre réglementaire applicable
2. Statut OIV et obligations Loi 05-20 / DGSSI
3. Bank Al-Maghrib — circulaires applicables
4. AMMC et gouvernance société cotée
5. Architecture sécurité — modèle STRIDE et contrôles
6. Données personnelles — Loi 09-08 / CNDP
7. Signature électronique — Loi 43-20
8. Stratégie de tests
9. Gestion des incidents et notifications
10. Plan de continuité d'activité (PCA / PRA)
11. Risques cotés P×I
12. Plan d'action conformité et budget
13. Annexes

---

## 1. Cadre réglementaire applicable

### 1.1 Hiérarchie des textes opposables

| Niveau | Texte | Périmètre | Sanction max |
|---|---|---|---|
| Constitutionnel | Constitution 2011, art. 24 | Vie privée, secret correspondances | – |
| Législatif | **Loi 09-08** (18 fév. 2009) | Protection données personnelles | 300 K MAD + prison 3 mois–1 an |
| Législatif | **Loi 43-20** (8 août 2020) | Services de confiance pour transactions électroniques | 500 K MAD + nullité acte |
| Législatif | **Loi 05-20** (juillet 2020) | Cybersécurité | 1 M MAD + prison 6 mois–2 ans |
| Législatif | **Loi 17-95** (modifiée 20-19) | Sociétés anonymes (banque cotée) | Sanctions civiles + responsabilité dirigeants |
| Législatif | **Loi 43-05** | Lutte anti-blanchiment / FT | 500 K MAD à 100 M MAD + prison |
| Législatif | **Loi 31-08** | Protection consommateur | 50 K MAD |
| Réglementaire | **Code de commerce 15-95**, art. 22 | Conservation documents commerciaux 10 ans | Sanction fiscale + civile |
| Réglementaire | **DOC** (Dahir des Obligations et Contrats), art. 417-1 et 417-2 | Force probante écrit électronique | Inopposabilité |
| Réglementaire | **Décret 2-09-165** | Application Loi 09-08 | – |
| Réglementaire | **Décret 2-21-406** | Application Loi 05-20 | – |
| Sectoriel banque | **Circulaires Bank Al-Maghrib** | Gouvernance, PCA, LAB-FT, contrôle interne | Sanctions BAM (avertissement, amende, retrait agrément) |
| Sectoriel marchés | **Circulaires AMMC** | Société cotée — info financière, contrôle interne | Amendes AMMC + sanctions cotation |
| Référentiel sécurité | **DNSSI** (DGSSI, 2014) | Sécurité SI — OIV obligatoire | Mise en demeure DGSSI + sanctions Loi 05-20 |
| Délibérations CNDP | **Délibération 478-2013** + autres | Mesures techniques sécurité données | Sanctions Loi 09-08 |

### 1.2 Loi 09-08 sur la protection des données personnelles — articles clés

| Article | Disposition | Implémentation parapheur |
|---|---|---|
| **Art. 1-3** | Définitions (donnée personnelle, traitement, responsable, sous-traitant, consentement) | Glossaire métier + matrice rôles |
| **Art. 4** | Principes : licéité, loyauté, finalité, exactitude, minimisation, conservation limitée | Documenté par traitement |
| **Art. 5** | Consentement comme base, sauf exceptions (contrat, obligation légale, intérêt vital, mission d'intérêt public, intérêt légitime) | Bases légales : exécution contrat travail (collaborateurs) + obligation légale (archivage 10 ans) |
| **Art. 7** | Données sensibles (santé, religion, origine ethnique, opinion, syndicale, sexuelle) — autorisation expresse CNDP | **Aucune donnée sensible dans le parapheur** — confirmé par DPIA |
| **Art. 8** | Information préalable : identité responsable, finalité, destinataires, droits, durée | Mention d'information dans interface utilisateur + politique confidentialité |
| **Art. 9-10** | Droit d'accès — gratuité + délai 30 jours | Procédure + outil ticketing + journal des demandes |
| **Art. 11** | Droit de rectification | Procédure |
| **Art. 12** | **Déclaration préalable CNDP** — formulaire spécifique | Dépôt avant mise en production — délai instruction CNDP 2 mois |
| **Art. 13-15** | Demandes d'autorisation (données sensibles, transferts) | Demande autorisation transfert si hébergement hors Maroc |
| **Art. 23** | Droit d'opposition | Procédure |
| **Art. 23 bis** (ajouté par Loi 31-08) | Droit à l'oubli (suppression) | Procédure + politique rétention |
| **Art. 24** | Sécurité du traitement (mesures techniques + organisationnelles) | Cf. § 5 architecture sécurité |
| **Art. 30** | Sous-traitance — contrat écrit, obligations sécurité | Contrat type avec plateforme e-sign + GED internes (refacturation interne) |
| **Art. 43** | **Transferts hors Maroc** — autorisation préalable CNDP, sauf liste pays adéquats (UE depuis 2009) | Si hébergement AWS Paris → procédure simplifiée ; si hors UE/Maroc → autorisation expresse |
| **Art. 51-55** | Sanctions : 10 K à 300 K MAD + emprisonnement 3 mois–1 an | Inclus dans cartographie risques |
| **Art. 56-66** | Compétence CNDP : contrôles, mises en demeure, sanctions, recours | Procédure de coopération CNDP |

**Différences notables vs RGPD** :
- Approche **déclarative** (déclaration préalable art. 12) au lieu de RGPD accountability (registre interne)
- **Pas de DPO obligatoire** — mais désignation d'un correspondant CNDP recommandée et exigée par BAM en pratique
- **Pas de notification 72h** des violations (mais BAM impose 24h pour incident SI majeur, donc plus strict en pratique)
- **Sanctions pénales** possibles (3 mois à 1 an de prison) — RGPD purement administratif
- **Sanctions financières plus faibles** (300 K MAD vs 4 % CA) mais effet réputationnel similaire pour banque cotée
- **CNIE** (Carte Nationale d'Identité Électronique) — identifiant unique national, traitement nécessite vigilance particulière

### 1.3 Loi 43-20 sur les services de confiance pour les transactions électroniques

**Adoptée le 8 août 2020, abrogeant la Loi 53-05 de 2007.** Loi 43-20 reproduit l'architecture eIDAS (UE 910/2014) avec adaptations marocaines.

#### Niveaux de signature électronique

| Niveau | Article 43-20 | Équivalent eIDAS | Force probante (DOC art. 417-1) | Identification signataire |
|---|---|---|---|---|
| **SES** Signature Électronique Simple | art. 4 | SES | Recevable, à apprécier par le juge | Faible (login + mot de passe) |
| **SEA** Signature Électronique Avancée | art. 4 al. 2 | AdES | Recevable, force probante | Moyenne (MFA + identité vérifiée) |
| **SEQ** Signature Électronique Qualifiée | art. 4 al. 3 | QES | **Équivalente à signature manuscrite** (DOC art. 417-1) — présomption de fiabilité | Forte (CNIE + face-to-face ou équivalent + dispositif sécurisé) |

#### Conditions cumulatives SEA (art. 5)

1. Liée uniquement au signataire
2. Permet d'identifier le signataire
3. Créée à l'aide de moyens que le signataire conserve sous son contrôle exclusif
4. Liée aux données associées de telle sorte que toute modification ultérieure est détectable

#### Conditions cumulatives SEQ (art. 6)

1. Toutes conditions SEA
2. Repose sur un **certificat qualifié** émis par un **PSCo agréé DGSSI**
3. Créée par un **dispositif sécurisé de création de signature** (carte à puce, HSM certifié, app mobile certifiée)
4. Identification renforcée du signataire (face-to-face ou procédure équivalente)

#### PSCo agréés DGSSI au Maroc (registre public — à vérifier 2026)

- **Barid eSign** (Barid Al-Maghrib / Poste Maroc) — signature qualifiée + horodatage qualifié
- **Maroc Telecom Trust Services**
- **Inwi Trust** (à confirmer)
- (registre tenu à jour par la DGSSI — www.dgssi.gov.ma)

#### Implication pour le parapheur (signature interne uniquement)

| Cas d'usage | Niveau requis | Justification |
|---|---|---|
| Délégations internes, accusés de réception, validations < 100 K MAD | **SES** | Force probante interne suffisante, faible risque |
| Décisions internes, ordres de mission, validations 100 K – 5 M MAD | **SEA** | Engagement interne moyen, contrôle interne requiert traçabilité forte |
| Validations comité de crédit > 5 M MAD, engagements ALM, dérogations risque, dotations capitaux propres | **SEA** + double signature (deux signataires) ou **SEQ optionnelle** | Exigence contrôle interne renforcée (banque cotée AMMC) — la SEQ apporte preuve qualifiée vis-à-vis audit interne et inspection BAM |

**Note importante** : la signature étant **interne uniquement**, l'enjeu n'est **pas** l'opposabilité aux tiers (clients externes). L'enjeu est :
1. La **traçabilité interne** vis-à-vis du contrôle interne et de l'audit
2. La **preuve vis-à-vis de l'inspection BAM** (audit annuel obligatoire pour OIV bancaire)
3. La **preuve vis-à-vis de l'AMMC** (banque cotée, contrôle interne renforcé)

→ **Recommandation** : SEA suffit dans 95 % des cas. SEQ optionnelle pour engagements > 5 M MAD avec impact patrimonial banque (audit AMMC — Code Marocain de Bonnes Pratiques de Gouvernance).

→ **Décision COPIL D-R2 (révisée)** : SEA pour 100 % des cas internes ; SEQ uniquement pour engagements internes > 5 M MAD si plateforme banque agréée DGSSI ; sinon double SEA + ticket d'audit.

### 1.4 Code de commerce 15-95 — preuve commerciale

| Article | Disposition | Application parapheur |
|---|---|---|
| **Art. 22** | Conservation pendant 10 ans des livres et correspondances commerciales | Archivage 10 ans dans GED interne |
| **Art. 23-24** | Force probante des livres de commerce | Journal d'audit immuable du parapheur considéré comme registre |

### 1.5 Dahir des Obligations et Contrats (DOC) — preuve électronique

| Article | Disposition |
|---|---|
| **Art. 417-1** | L'écrit sous forme électronique est admis en preuve au même titre que l'écrit sur support papier, sous réserve que la personne dont il émane puisse être dûment identifiée et qu'il soit établi et conservé dans des conditions de nature à en garantir l'intégrité |
| **Art. 417-2** | La signature nécessaire à la perfection d'un acte juridique identifie la personne qui l'appose. Lorsqu'elle est électronique, elle consiste en l'usage d'un procédé fiable d'identification garantissant son lien avec l'acte auquel elle s'attache. La fiabilité de ce procédé est présumée jusqu'à preuve contraire lorsque la signature électronique est créée, l'identité du signataire assurée et l'intégrité de l'acte garantie dans les conditions fixées par la législation en vigueur (=Loi 43-20) |

**Conséquence pratique** : un document signé en SEQ via PSCo agréé DGSSI bénéficie d'une **présomption légale de fiabilité**. Pour SEA, la fiabilité doit être démontrée par le responsable du traitement (logs, certificats, intégrité hash, horodatage).

---

## 2. Statut OIV et obligations Loi 05-20 / DGSSI

### 2.1 Classification OIV et conséquences

La **Loi 05-20** sur la cybersécurité (juillet 2020), avec son décret d'application **2-21-406**, classe certaines entités comme **Opérateurs d'Importance Vitale (OIV)** dont les **banques de la place de Casablanca de taille systémique**. Le statut OIV impose des obligations renforcées :

| Obligation | Échéance | Détail |
|---|---|---|
| **Désignation d'un Responsable de la Sécurité des Systèmes d'Information (RSSI)** | Permanent | Officiellement déclaré à la DGSSI, cumul fonctions interdit avec DSI |
| **Cartographie des SI sensibles** | Annuelle | Inventaire complet — le parapheur y figure (signature documents internes engagement banque) |
| **Plan de Sécurité de l'Opérateur (PSO)** | Annuel | Plan formalisé soumis DGSSI |
| **Audit DGSSI** | Annuel ou bisannuel | Audit par la DGSSI ou prestataire agréé |
| **Notification incidents 24h** | Continu | Tout incident SI majeur notifié DGSSI sous 24h |
| **Tests d'intrusion** | Annuel | Pentest annuel obligatoire par cabinet agréé DGSSI |
| **Conformité DNSSI** | Permanent | Application du référentiel DNSSI (~120 mesures sur 11 chapitres) |
| **Hébergement** | Permanent | Données sensibles **hébergées au Maroc** (sauf dérogation expresse DGSSI) |
| **Personnel** | Permanent | Habilitation des intervenants critiques + clauses contractuelles |
| **Continuité** | Permanent | PCA testé annuellement |

### 2.2 DNSSI — Directive Nationale de la Sécurité des Systèmes d'Information

La **DNSSI** (Directive 2014, mises à jour DGSSI périodiques) est le **référentiel sécurité socle** opposable aux OIV. Structure en 11 chapitres et ~120 mesures.

| Chapitre DNSSI | Thématique | Mesures applicables au parapheur |
|---|---|---|
| **Ch. 1** | Politique et organisation de la sécurité | PSSI banque, comité sécurité, responsabilités |
| **Ch. 2** | Gestion des risques | Analyse risques EBIOS-RM ou ISO 27005, revue annuelle |
| **Ch. 3** | Sécurité des ressources humaines | Habilitations, sensibilisation, départ |
| **Ch. 4** | Gestion des biens | Cartographie, classification, propriétaire |
| **Ch. 5** | Contrôle d'accès | IAM, MFA, séparation des privilèges, journalisation |
| **Ch. 6** | Cryptographie | Chiffrement at-rest et in-transit, gestion des clés, algorithmes recommandés DGSSI |
| **Ch. 7** | Sécurité physique et environnementale | Datacenter Tier III+, zones sécurisées |
| **Ch. 8** | Sécurité de l'exploitation | Patch management, malware, backup, segmentation réseau |
| **Ch. 9** | Sécurité des communications | Chiffrement réseau, DMZ, IDS/IPS, filtrage |
| **Ch. 10** | Acquisition, développement, maintenance | Sécurité dans le cycle de développement (SDLC), tests de sécurité, secrets |
| **Ch. 11** | Gestion des incidents et continuité | Détection, réponse, notification 24h DGSSI, retour d'expérience |

**Mapping DNSSI ↔ ISO 27001:2022** : ~80 % des contrôles sont équivalents. Annexe B fournit le mapping détaillé. La DNSSI prime en cas de conflit.

### 2.3 Loi 05-20 cybersécurité — obligations du parapheur

| Article | Disposition | Implémentation |
|---|---|---|
| **Art. 6** | Désignation RSSI | Existant banque |
| **Art. 7** | Audit annuel | Cabinet agréé DGSSI |
| **Art. 8** | Notification incident 24h | Procédure formalisée |
| **Art. 12-15** | Mesures techniques (chiffrement, MFA, journalisation) | Cf. § 5 |
| **Art. 21-25** | Hébergement données sensibles | Au Maroc sauf dérogation DGSSI |
| **Art. 30-35** | Sanctions : 100 K à 1 M MAD + prison 6 mois–2 ans | Cartographie risques |

---

## 3. Bank Al-Maghrib — circulaires applicables

### 3.1 Circulaire BAM 03/W/2016 — Gouvernance des SI

Cadre de **gouvernance, gestion des risques SI et contrôle des SI** des établissements de crédit. Structure en 4 piliers.

| Pilier | Exigences | Implémentation parapheur |
|---|---|---|
| **Gouvernance** | Schéma directeur SI, comité SI, rôles formalisés | Le parapheur figure au schéma directeur, comité sécurité valide mise en production |
| **Gestion des risques SI** | Cartographie, analyse, plans de traitement | Cartographie risques opérationnels du parapheur, registre des risques |
| **Contrôle interne SI** | Permanent + périodique, séparation des fonctions, traçabilité | Audit interne semestriel + audit externe annuel cabinet inscrit BAM |
| **Externalisation** | Contrat type, SLA, droit d'audit, plan de réversibilité | Contrats internes (refacturation) avec plateformes e-sign + GED — clauses SLA, audit, réversibilité |

#### Notification incident — circulaire 03/W/2016 art. 23

> Tout incident SI majeur affectant la sécurité, la disponibilité ou l'intégrité des SI critiques fait l'objet d'une notification à Bank Al-Maghrib **dans un délai maximal de 24 heures** suivant sa détection.

### 3.2 Circulaire BAM 5/W/2017 — Plan de Continuité d'Activité (PCA)

| Exigence | Cible parapheur |
|---|---|
| RTO (Recovery Time Objective) | ≤ 4h pour systèmes critiques |
| RPO (Recovery Point Objective) | ≤ 1h |
| Plan documenté | PCA + PRA |
| **Test annuel** | Obligatoire — PV de test à BAM sur demande |
| Site de secours | Distinct géographiquement (cross-site Maroc) |

**Note OIV** : la circulaire 5/W/2017 + Loi 05-20 imposent un **second site de secours physiquement distant** (>50 km recommandé). Pour solo + cloud Maroc, ceci se traduit par : sauvegarde quotidienne sur région secondaire (ex. Casablanca + Rabat) avec restore < 24h.

### 3.3 Circulaire BAM 1/G/2010 — Lutte anti-blanchiment / FT

| Exigence | Implémentation parapheur |
|---|---|
| KYC signataires | Identification CNIE + matricule banque |
| Traçabilité signatures | Journal immuable + horodatage qualifié |
| Conservation 10 ans | Archivage GED conforme art. 22 Code commerce |
| Signalement opérations suspectes | Workflow ticket UTRF si cas applicable |

### 3.4 Circulaire BAM 2/G/2012 — Contrôle interne

| Exigence | Implémentation parapheur |
|---|---|
| Cartographie des risques opérationnels | Le parapheur dans la cartographie |
| Trois lignes de défense (1L métier, 2L conformité/risques, 3L audit interne) | Modèle implémenté banque |
| Contrôles permanents | Contrôles automatisés dans le parapheur (séparation des fonctions, double validation > seuils) |
| Reporting trimestriel | Inclusion KPIs parapheur |

### 3.5 Directive BAM hébergement et cloud (à vérifier mise à jour 2026)

Bank Al-Maghrib a publié des recommandations sur l'externalisation cloud (note technique 2020, circulaire en projet 2025) imposant en pratique :

- **Souveraineté** : données sensibles hébergées au Maroc sauf dérogation justifiée
- **Audit** : droit d'audit du prestataire par BAM
- **Réversibilité** : plan documenté, tests annuels
- **Continuité** : engagement contractuel SLA et PCA prestataire
- **Confidentialité** : chiffrement en transit + at-rest, contrôle clés par la banque

---

## 4. AMMC et gouvernance société cotée

### 4.1 Périmètre AMMC

L'**Autorité Marocaine du Marché des Capitaux (AMMC)** régule les sociétés cotées à la Bourse des Valeurs de Casablanca. La banque, en tant qu'**émetteur coté**, est soumise à :

| Texte | Disposition | Impact parapheur |
|---|---|---|
| **Loi 17-95** (sociétés anonymes), modifiée par 20-19 | Gouvernance, conseil d'administration, contrôle interne | Le parapheur participe au dispositif de contrôle interne par traçabilité des engagements |
| **Loi 19-14** (Bourse des valeurs) | Obligations émetteur | Confidentialité info privilégiée — le parapheur ne traite pas d'info privilégiée mais le journal d'audit doit être protégé |
| **Circulaire AMMC n° 03-19** | Code Marocain de Bonnes Pratiques de Gouvernance d'Entreprise | Contrôle interne, gestion des risques, audit |
| **Circulaire AMMC n° 02-19** | Information financière périodique | Continuité du système d'information critique pour clôture comptable |

### 4.2 Code Marocain de Bonnes Pratiques de Gouvernance — exigences applicables

| Pilier | Exigence | Implémentation parapheur |
|---|---|---|
| **Conseil d'administration** | Comités spécialisés (audit, risques, rémunérations) | Le parapheur permet de tracer les délibérations engagement opérationnel |
| **Contrôle interne** | Dispositif formalisé, trois lignes de défense, indépendance audit interne | Journal d'audit du parapheur exploitable par audit interne |
| **Information** | Fiabilité, sincérité, exhaustivité | Logs immuables + horodatage qualifié |
| **Risques** | Identification, mesure, maîtrise, reporting | Cartographie risques opérationnels du parapheur |

### 4.3 Conséquences sur le parapheur (banque cotée)

1. **Renforcement traçabilité** — le journal d'audit immuable est un élément du dispositif de contrôle interne. Conservation **10 ans minimum** (Code commerce 15-95 art. 22) et **accessibilité audit AMMC** (sur demande, sous 5 jours ouvrés).
2. **Séparation des fonctions** — RBAC strict avec règle des 4 yeux pour engagements significatifs.
3. **Continuité** — l'indisponibilité du parapheur peut bloquer la clôture comptable trimestrielle (information financière périodique AMMC) → criticité élevée.
4. **Inspection AMMC** — capacité à fournir extraction journal d'audit sous 5 jours.

---

## 5. Architecture sécurité — modèle STRIDE et contrôles

### 5.1 Cartographie des composants et zones de sécurité

```
┌─ Zone Internet/Client (n/a — accès interne uniquement) ─┐
│
├─ Zone Frontend (DMZ interne banque)
│   ├─ React + TypeScript (PWA)
│   └─ API Gateway (Kong / NGINX)
│
├─ Zone Application (réseau interne sécurisé)
│   ├─ MS-1 Dossier
│   ├─ MS-2 Workflow
│   ├─ MS-3 Notification
│   ├─ MS-4 Audit
│   ├─ MS-5 Adapter e-Sign
│   └─ MS-6 Adapter GED
│
├─ Zone Données (réseau interne haute sécurité)
│   ├─ PostgreSQL 16 (cluster primary/replica)
│   ├─ Redis 7 (cache + queue BullMQ)
│   └─ MinIO ou stockage objet interne (fichiers actifs en transit)
│
└─ Zone Externe (intégrations API banque)
    ├─ Plateforme e-signature interne (HTTPS/mTLS)
    ├─ Plateforme GED interne (HTTPS/mTLS)
    ├─ AD/LDAP banque (LDAPS)
    ├─ Core banking (REST/HTTPS)
    └─ SMTP/SMS Gateway internes
```

### 5.2 Modèle STRIDE par composant

STRIDE = **S**poofing, **T**ampering, **R**epudiation, **I**nformation disclosure, **D**enial of service, **E**levation of privilege.

#### MS-1 Dossier (CRUD, métadonnées, recherche)

| Menace | Vecteur | Mitigation |
|---|---|---|
| Spoofing | Identité usurpée AD | MFA OTP obligatoire + journalisation accès + détection comportementale |
| Tampering | Modification métadonnées | Contrôle d'intégrité hash SHA-256 + journal immuable des modifications |
| Repudiation | Action niée par utilisateur | Journal MS-4 avec timestamp + signature requête + horodatage NTP synchronisé |
| Information disclosure | Lecture non autorisée | RBAC + ABAC (filtrage par périmètre — direction, agence, comité) + chiffrement at-rest |
| Denial of service | Saturation API | Rate limiting Kong (100 req/min/utilisateur) + circuit breaker |
| Elevation of privilege | Bypass autorisation | Validation JWT à chaque requête + contrôle ABAC en backend (pas seulement front) |

#### MS-2 Workflow (state machine + règles métier)

| Menace | Vecteur | Mitigation |
|---|---|---|
| Spoofing | Action déclenchée par autre utilisateur | Token JWT signé + contrôle owner du workflow |
| Tampering | Modification état workflow | Transitions auditées + état immuable post-validation |
| Repudiation | Validation contestée | Journal des transitions + signature à chaque étape critique |
| Information disclosure | Fuite règles métier | Chiffrement secrets DMN + accès lecture restreint Direction Risques |
| Denial of service | Boucle infinie workflow | Détection cycles + timeout configurable par étape |
| Elevation of privilege | Approbation auto au-delà délégation | Contrôle DMN + double validation côté backend (pas seulement DMN) |

#### MS-5 Adapter e-Signature

| Menace | Vecteur | Mitigation |
|---|---|---|
| Spoofing | Faux callback de la plateforme | mTLS + signature HMAC du callback + IP whitelist |
| Tampering | Document modifié entre demande et signature | Hash SHA-256 du document calculé avant envoi + vérifié au callback |
| Repudiation | Signature contestée | Certificat conservé + horodatage qualifié + journal MS-4 |
| Information disclosure | Document confidentiel intercepté | mTLS + TLS 1.3 + chiffrement at-rest + accès au document limité au signataire jusqu'à signature |
| Denial of service | Plateforme e-sign indisponible | Circuit breaker + retry exponentiel + queue d'attente BullMQ + procédure dégradée (signature physique avec scan a posteriori) |
| Elevation of privilege | Signature au nom d'un autre | Identification renforcée côté plateforme e-sign (CNIE, MFA fort) |

#### MS-6 Adapter GED

| Menace | Vecteur | Mitigation |
|---|---|---|
| Spoofing | Faux dépôt | mTLS + auth OAuth2 client_credentials |
| Tampering | Document modifié post-archivage | Hash conservé en MS-4 + vérification périodique batch |
| Repudiation | Suppression contestée | Politique legal hold + journal MS-4 |
| Information disclosure | Lecture non autorisée | RBAC GED + filtrage par dossier |
| Denial of service | GED saturée | Queue BullMQ + retry + alerting |
| Elevation of privilege | Suppression non autorisée | Politique de rétention 10 ans + legal hold actif par défaut |

### 5.3 Contrôles techniques détaillés

#### Authentification et gestion d'identité

| Contrôle | Mise en œuvre |
|---|---|
| Authentification primaire | AD banque (Kerberos / SAML 2.0 ou LDAP) |
| MFA | OTP banque (SMS, app TOTP, ou clé matérielle si déjà déployée) — obligatoire pour 100 % des utilisateurs |
| Session | JWT signé RS256, durée 8h, renouvellement avec refresh token, révocation centralisée Redis |
| Tentatives infructueuses | Verrouillage 5 tentatives, déverrouillage manuel admin |
| Politique mot de passe | Déléguée AD banque (≥ 12 car., complexité, rotation 90j) |
| Comptes service | Vault HashiCorp ou secret manager interne, rotation trimestrielle |

#### Contrôle d'accès — RBAC × ABAC

**Rôles** (RBAC) :

| Rôle | Périmètre | Permissions |
|---|---|---|
| Émetteur | Tout collaborateur | Créer dossier, soumettre |
| Valideur N1-N5 | Selon hiérarchie | Valider selon délégation DMN |
| Comité de crédit | Membres comité | Valider engagements > 5 M MAD |
| Inspection | Auditeurs | Lecture seule globale, extraction journal |
| Conformité | Direction conformité | Lecture seule + alertes LAB-FT |
| Risques | Direction risques | Lecture seule + alertes dérogation |
| RSSI | Sécurité | Configuration, audit logs, gestion clés |
| Admin technique | DSI | Configuration technique, secrets |
| Audit interne (3L défense) | Audit interne | Lecture seule globale, extraction |
| Auditeur externe BAM/AMMC | Sur demande, période limitée | Lecture seule, extraction |

**Attributs** (ABAC) :

| Attribut | Source | Usage |
|---|---|---|
| Direction | AD | Filtrage périmètre |
| Agence | AD | Filtrage géographique |
| Niveau hiérarchique | AD/RH | Délégation montants |
| Délégation expresse | Workflow | Délégation temporaire (vacances) |
| Périmètre LAB | Conformité | Workflow renforcé |
| Engagement comité | Décision conseil | Plafond ad hoc |

**Règle d'évaluation** : `permission = RBAC(rôle, action) AND ABAC(attributs, ressource)` — évaluation côté backend (PEP/PDP pattern), jamais en confiance frontend.

#### Cryptographie

| Donnée | Algorithme | Clé / Mode |
|---|---|---|
| Chiffrement at-rest PostgreSQL | AES-256-GCM | Clés gérées par cloud banque KMS ou HashiCorp Vault |
| Chiffrement at-rest fichiers (MinIO ou S3-compatible) | AES-256-GCM | Idem |
| Chiffrement in-transit interne | TLS 1.3 | Certificats internes banque (PKI banque) |
| Chiffrement in-transit externe | TLS 1.3 + mTLS | Vers plateforme e-sign + GED |
| Hashes documents | SHA-256 | Stockés dans MS-4 |
| Tokens JWT | RS256 | Clé privée 4096 bits, rotation annuelle |
| Sessions Redis | Chiffrement transparent | Redis 7 chiffrement at-rest et in-transit |
| Sauvegardes | AES-256 + GPG | Clés gérées séparément |
| Mots de passe | Délégué AD | – |
| Algorithmes recommandés DGSSI | RSA ≥ 2048, AES ≥ 128, SHA ≥ 256, ECDSA P-256 | Conformité chap. 6 DNSSI |

#### Journalisation et audit (MS-4)

| Type d'événement | Niveau | Conservation | Destinataire |
|---|---|---|---|
| Authentification (succès/échec) | INFO/WARN | 1 an chaud + 5 ans froid | Sécurité + audit |
| Création/modification dossier | INFO | 10 ans (Code commerce) | Audit + journal métier |
| Validation/signature | **CRITICAL** | 10 ans + legal hold possible | Audit + AMMC + BAM |
| Délégation accordée/révoquée | INFO | 5 ans | Audit + RH |
| Erreur applicative | ERROR | 90 jours chaud + 1 an archive | DSI + sécurité |
| Tentative d'accès non autorisée | **WARN/CRITICAL** | 1 an chaud + 5 ans froid | RSSI + SOC |
| Changement de configuration | **CRITICAL** | 5 ans | Audit + RSSI |
| Export de données | **CRITICAL** | 5 ans | DPO/correspondant CNDP + audit |

**Caractéristiques journal** :
- **Immuabilité** : append-only, hash chaîné par lot horaire (similaire blockchain), stockage WORM si possible (S3 Object Lock ou équivalent banque)
- **Horodatage NTP synchronisé** (DGSSI recommande source temps interne banque + NTP public + serveur SNTP DGSSI)
- **Format** structuré JSON, schéma versionné
- **Indexation** Elasticsearch ou PostgreSQL full-text pour recherche rapide
- **Anonymisation** post-période légale (suppression données personnelles, conservation événements anonymisés)

#### Sécurité applicative

| Mesure | Mise en œuvre |
|---|---|
| **OWASP Top 10** | Couverture par revue + tests (SAST, DAST, dépendances) |
| **Validation entrées** | Zod (TypeScript) côté front et back, règles strictes |
| **Échappement sorties** | React JSX (auto-échappement) + sanitization (DOMPurify si besoin HTML enrichi) |
| **Protection CSRF** | Tokens CSRF + SameSite=Strict cookies |
| **Protection XSS** | Content Security Policy strict + nonces |
| **Protection clickjacking** | X-Frame-Options DENY + CSP frame-ancestors |
| **Protection injection SQL** | Prisma ORM (paramétrisation automatique) — pas de requêtes brutes |
| **Protection NoSQL injection** | n/a (pas de NoSQL) |
| **Protection XXE** | Parsers XML désactivés DTD |
| **Protection deserialization** | Zod validation stricte |
| **Secrets** | Jamais dans code, jamais dans logs, masking automatique pino |
| **Dépendances** | Snyk + Dependabot, alertes critiques traitées sous 7j |
| **Conteneurs** | Trivy scan obligatoire, base images Alpine/distroless, non-root |

### 5.4 Cartographie 47 contrôles ISO 27001:2022 ↔ DNSSI

(Mapping complet en annexe B — 47 contrôles ISO Annexe A organisés par les 11 chapitres DNSSI. ~80 % d'équivalence.)

**Les 10 contrôles ISO les plus critiques pour le parapheur** :

| # | Contrôle ISO 27001:2022 | Chapitre DNSSI | Mise en œuvre |
|---|---|---|---|
| 1 | A.5.1 Politique de sécurité | Ch. 1 | PSSI banque |
| 2 | A.5.15 Contrôle d'accès | Ch. 5 | RBAC × ABAC |
| 3 | A.5.16 Identification | Ch. 5 | AD + MFA |
| 4 | A.5.17 Information d'authentification | Ch. 5 | Politique MdP AD + MFA |
| 5 | A.5.23 Sécurité services cloud | Ch. 9 | Hébergement Maroc, mTLS, audit |
| 6 | A.5.34 Confidentialité données personnelles | Ch. 5+10 | Chiffrement, RBAC, déclaration CNDP |
| 7 | A.8.5 Authentification sécurisée | Ch. 5 | MFA |
| 8 | A.8.7 Protection contre logiciels malveillants | Ch. 8 | Trivy + EDR banque |
| 9 | A.8.24 Cryptographie | Ch. 6 | AES-256, TLS 1.3, RS256 |
| 10 | A.8.28 Codage sécurisé | Ch. 10 | SAST, code review, secure SDLC |

---

## 6. Données personnelles — Loi 09-08 / CNDP

### 6.1 Inventaire des données personnelles traitées

| Catégorie | Donnée | Source | Finalité | Base légale | Conservation |
|---|---|---|---|---|---|
| **Identification** | Matricule banque | AD | Auth + traçabilité | Exécution contrat travail | Durée emploi + 5 ans |
| | Nom, prénom | AD | Identification | Idem | Idem |
| | CNIE (numéro) | RH | Signature, KYC interne | Obligation légale | Durée emploi + 10 ans |
| | Email professionnel | AD | Notification | Exécution contrat | Idem |
| | Téléphone professionnel | AD | Notification SMS | Exécution contrat | Idem |
| | Adresse IP | Logs | Audit, sécurité | Intérêt légitime | 1 an chaud + 5 ans archive |
| **Hiérarchie** | Direction, agence | AD | Routage workflow | Exécution contrat | Durée emploi |
| | Niveau, fonction | AD/RH | Délégation | Exécution contrat | Idem |
| | Manager direct | AD/RH | Escalade | Exécution contrat | Idem |
| | Délégations expresses | Saisie | Délégation temporaire | Exécution contrat | 5 ans après expiration |
| **Activité** | Dossiers émis/validés | Application | Suivi activité | Intérêt légitime + obligation contrôle interne | 10 ans (Code commerce) |
| | Horodatage actions | Application | Audit | Idem | Idem |
| | Géolocalisation IP | Logs | Sécurité | Intérêt légitime | 1 an |
| **Signature** | Certificat de signature | Plateforme e-sign | Preuve | Obligation légale (Code commerce) | 10 ans |
| | Horodatage qualifié | Plateforme e-sign | Preuve | Idem | Idem |

**Total** : ~25 PII (vs 47 en v1 RGPD — la signature étant interne uniquement, pas de données clients externes).

**Aucune donnée sensible** au sens de l'art. 7 Loi 09-08 (santé, religion, opinions, race, sexualité, syndicale).

### 6.2 Bases légales utilisées (Loi 09-08 art. 5)

| Base légale | Application | Justification |
|---|---|---|
| **Exécution du contrat de travail** | Identification, hiérarchie, activité professionnelle | Le parapheur est un outil de travail des collaborateurs |
| **Obligation légale** | Conservation 10 ans, signatures qualifiées, traçabilité audit | Code commerce 15-95, BAM circulaires, AMMC, Loi 05-20 |
| **Intérêt légitime** | Logs sécurité, IP, géolocalisation IP | Mise en balance : protection SI vs vie privée → favorable au traitement, mesures de minimisation appliquées |

**Pas de consentement requis** (toutes bases sont contractuelles ou légales).

### 6.3 Procédure de déclaration CNDP

**Étape 1 — Pré-consultation** (gratuite, recommandée)

- Mail à la CNDP avec note de présentation du traitement
- Délai de réponse : 2-4 semaines
- Permet d'identifier les ajustements préalables

**Étape 2 — Déclaration formelle** (formulaire CNDP)

| Section | Contenu |
|---|---|
| Identification responsable | Banque, RC, ICE, IF, adresse, contact correspondant CNDP |
| Sous-traitants | Plateformes e-sign + GED internes (refacturation interne — à analyser : sous-traitance ou même responsable de traitement ?) |
| Finalité | Signature électronique de documents internes engagement banque |
| Catégories de données | Cf. § 6.1 |
| Catégories de personnes | Collaborateurs banque (contractuels et stagiaires inclus) |
| Destinataires | Audit interne, AMMC sur demande, BAM sur demande, autorité judiciaire si réquisition |
| Transferts hors Maroc | Aucun (hébergement Maroc) — sinon autorisation art. 43 |
| Durée de conservation | 10 ans (Code commerce) puis anonymisation/suppression |
| Mesures de sécurité | Synthèse § 5 |
| Droits des personnes | Procédure d'exercice (cf. § 6.4) |

**Délai d'instruction CNDP** : 2 mois (silence = refus, mais en pratique réponse positive ou demande compléments).

**Frais** : ~500 à 5 000 MAD selon complexité.

### 6.4 Procédure d'exercice des droits (art. 9-23 bis)

| Droit | Délai légal | Procédure parapheur |
|---|---|---|
| Accès (art. 9) | 30 jours | Formulaire + ticketing → DPO/correspondant CNDP banque → extraction MS-4 |
| Rectification (art. 11) | 30 jours | Idem + workflow correction |
| Opposition (art. 23) | 30 jours | Évaluation cas par cas (motif légitime) |
| Suppression / oubli (art. 23 bis) | 30 jours | **Refusable si conservation légale** (10 ans Code commerce) — anonymisation possible des données non essentielles |
| Limitation | (non explicite Loi 09-08, RGPD oui) | Cas par cas |
| Portabilité | (non explicite Loi 09-08, RGPD oui) | Cas par cas |

**Outil** : ticketing (Jira / GLPI banque) avec catégorie « Demande Loi 09-08 », SLA 25 jours (marge sur 30 jours légaux).

### 6.5 Sous-traitance — plateformes internes banque

**Question juridique** : la plateforme e-sign et la GED internes sont-elles **sous-traitants** au sens de l'art. 30 Loi 09-08, ou **mêmes responsables de traitement** (puisque même entité juridique = banque) ?

**Analyse** :
- Si plateforme = même entité juridique → pas de sous-traitance, mais conventions internes formalisant rôles, responsabilités, sécurité
- Si plateforme = filiale ou entité distincte (cas fréquent : filiale technologique groupe) → sous-traitance, contrat type 09-08 art. 30 obligatoire

**Recommandation** : dans tous les cas, formaliser une **convention de service interne (SLA + sécurité + audit)** documentant :
- Rôles et responsabilités (qui fait quoi)
- Mesures de sécurité (équivalentes à celles de la banque)
- Droit d'audit annuel
- Gestion des incidents (notification mutuelle)
- Réversibilité (continuité en cas d'arrêt service)

### 6.6 Transferts hors Maroc

**Hypothèse retenue** : hébergement au Maroc (cloud banque ou DC Maroc Tier III+) → **aucun transfert** → procédure simplifiée.

**Si AWS Paris (UE)** : autorisation simplifiée art. 43 (UE = pays adéquat depuis 2009). Délai 1 mois CNDP.

**Si AWS Bahreïn / autre** : autorisation expresse + clauses contractuelles types CNDP. Délai 2-3 mois. **Déconseillé** pour OIV.

---

## 7. Signature électronique — Loi 43-20

### 7.1 Stratégie de signature

| Cas d'usage | Volume estimé | Niveau retenu | Justification |
|---|---|---|---|
| Notes de service, validations RH ≤ 100 K MAD | 7 000/an (60 %) | **SES** | Faible enjeu, rapide |
| Décisions internes 100 K – 5 M MAD, ordres de mission | 4 000/an (33 %) | **SEA** | Engagement moyen, traçabilité forte requise |
| Validations comité crédit > 5 M MAD, dérogations risques | 800/an (7 %) | **SEA + double signature** ou **SEQ optionnelle** | Engagement majeur, contrôle interne renforcé AMMC |
| **Total** | 12 000/an | – | – |

### 7.2 Architecture intégration plateforme e-signature interne

**Modèle d'API de référence** (à confirmer avec DSI banque en P0) :

```
POST /api/v1/sign-requests
Authorization: Bearer {token_oauth2_client_credentials}
mTLS: certificat client banque
Content-Type: application/json

{
  "documentId": "uuid-doc-v4",
  "documentName": "validation-engagement-2026-001234.pdf",
  "documentHash": "sha256:...",
  "documentBase64": "...",          // ou documentUrl si streaming
  "signatureLevel": "SEA",          // SES | SEA | SEQ
  "signers": [
    {
      "userId": "MAT12345",
      "email": "...",
      "phone": "+212...",
      "order": 1,
      "role": "valideur-n3",
      "delegationContext": "comite-credit-2026-04-25"
    }
  ],
  "callbackUrl": "https://parapheur.banque.ma/api/v1/esign/callback",
  "expiresAt": "2026-04-30T18:00:00+00:00",
  "metadata": {
    "dossierId": "DOS-2026-001234",
    "engagementMontant": 7500000,
    "engagementDevise": "MAD",
    "typeDecision": "validation-comite-credit"
  }
}
→ 202 Accepted
{
  "signRequestId": "uuid",
  "status": "PENDING",
  "signatureUrl": "https://esign.banque.ma/sign/...",
  "expiresAt": "..."
}
```

**Callback de la plateforme e-sign** (vers MS-5) :

```
POST /api/v1/esign/callback (vers parapheur)
mTLS + signature HMAC du payload (header X-Signature)

{
  "signRequestId": "uuid",
  "status": "SIGNED" | "REJECTED" | "EXPIRED" | "ERROR",
  "signatureLevel": "SEA",
  "signedDocumentUrl": "...",       // ou base64
  "signedDocumentHash": "sha256:...",
  "signers": [
    {
      "userId": "MAT12345",
      "signedAt": "2026-04-25T15:32:11+00:00",
      "certificateThumbprint": "...",
      "ipAddress": "10.x.x.x"
    }
  ],
  "timestampToken": "RFC3161-base64",
  "auditTrail": "..."
}
```

### 7.3 Format de signature

| Format | Utilisation | Avantages |
|---|---|---|
| **PAdES** (PDF Advanced Electronic Signature, ETSI EN 319 142) | Documents PDF signés | Norme reconnue, lecture sans outil tiers |
| **PAdES-LTA** (Long Term Archival) | Conservation > 5 ans | Re-horodatage, anti-obsolescence cryptographique |
| **XAdES** | Documents XML | Si métadonnées structurées |
| **CAdES** | Documents binaires | Rare cas d'usage |

**Recommandation** : **PAdES-LTA** pour archivage 10 ans (Code commerce). Re-horodatage tous les 5 ans pour résilience cryptographique (recommandation DGSSI).

### 7.4 Audit P0 — agrément DGSSI plateforme banque

**Question critique à clarifier en P0 (4 semaines)** : la plateforme e-signature interne de la banque est-elle :

- (a) **Agréée DGSSI comme PSCo qualifié** → SEQ disponible, équivalence signature manuscrite
- (b) **Non agréée mais conforme techniquement** → niveau plafonné à **SEA** (suffisant pour usage interne)
- (c) **Non opérationnelle / fonctionnalités manquantes** → escalade DSI banque + plan B

**Plan B (cas c)** :

| Option | Coût/an | Effort | Recommandation |
|---|---|---|---|
| **Barid eSign** (Poste Maroc) — PSCo agréé DGSSI | ~50-100 MAD/signature × 800 signatures/an SEQ = 40-80 K MAD/an | Intégration API REST 5 JH | **Plan B retenu** si plateforme banque indisponible |
| **Maroc Telecom Trust Services** | À négocier | Intégration 5 JH | Alternative |
| **Solution open source** (Cryptix, signatory) + horodatage TSA | ~10 K MAD setup | Effort dev 20 JH | Non agréée DGSSI → SES ou SEA seulement |

**Décision conditionnelle** : si plateforme banque non opérationnelle au démarrage P5, intégrer Barid eSign en parallèle (plan B activé).

---

## 8. Stratégie de tests

### 8.1 Pyramide ISTQB

```
                 ┌──────────────────┐
                 │  E2E Playwright  │  ~ 50 scénarios — parcours bout en bout
                 │       5 %        │
                 ├──────────────────┤
                 │  Intégration     │  ~ 200 tests — API + DB + adapters
                 │      15 %        │
                 ├──────────────────┤
                 │   Composants     │  ~ 300 tests — composants React isolés
                 │      30 %        │
                 ├──────────────────┤
                 │   Unit Vitest    │  ~ 800 tests — logique pure, services, règles DMN
                 │      50 %        │
                 └──────────────────┘
```

**Couverture cible** : ≥ 80 % global, ≥ 90 % sur services critiques (workflow, signature, audit).

### 8.2 Plan de test fonctionnel

| Phase | Périmètre | Outils | Effort solo |
|---|---|---|---|
| Tests unitaires | Services, fonctions pures, règles DMN | Vitest + JSDOM | Continu (TDD partiel) |
| Tests composants | Composants React isolés | Vitest + Testing Library + Storybook | Continu |
| Tests intégration | API + DB + Redis + adapters mocks | Vitest + Supertest + Testcontainers | Continu |
| Tests E2E | Parcours utilisateur complets | Playwright | 8 JH dédiés |
| Tests perfs | Charge, stress, endurance | k6 + Grafana | 4 JH dédiés |
| Tests sécurité (SAST) | Code | Semgrep, ESLint security, SonarQube | Continu pipeline |
| Tests sécurité (DAST) | Application déployée | OWASP ZAP, Burp Suite | 3 JH + pentest annuel |
| Tests dépendances | Vulnérabilités libs | Snyk, npm audit | Continu pipeline |
| Tests conteneurs | Images Docker | Trivy | Continu pipeline |
| Tests accessibilité | WCAG 2.1 AA + RTL arabe | axe-core, Lighthouse | 2 JH |

### 8.3 Plan de test sécurité — pentest annuel

**Périmètre** : application web + APIs + adapters + intégration AD/e-sign/GED.

**Méthodologie** : OWASP Testing Guide v4.2 + DGSSI guide test intrusion + PTES.

**Cabinet** : agréé DGSSI (registre public). Liste indicative :
- DataProtect (Casablanca)
- Dataprotection.ma
- Cyber-X-Pert
- Atos Cybersecurity Maroc
- Devoteam Maroc
(Liste non exhaustive — consulter registre DGSSI et liste cabinets inscrits BAM)

**Effort** : 8 jours pentest × 6 000 MAD = 48 000 MAD/an.

**Livrable** : rapport classifié, plan d'action correctif sous 30 jours, retest sous 60 jours.

### 8.4 Plan de test perf

| Test | Charge cible | Critère succès |
|---|---|---|
| Pic horaire | 200 dossiers/heure (× 4 du nominal 50/h) | P95 latence API < 500 ms |
| Endurance | 50 dossiers/h × 24h | Pas de fuite mémoire (< 5 % drift) |
| Stress | 1 000 dossiers/h | Dégradation gracieuse, circuit breaker actif |
| Reprise | Restauration backup | RTO ≤ 4h, RPO ≤ 1h |

### 8.5 Tests d'accessibilité (WCAG 2.1 AA + RTL arabe)

| Critère | Vérification |
|---|---|
| Contraste | Ratio ≥ 4.5:1 (normal), ≥ 3:1 (grand) |
| Navigation clavier | 100 % parcours accessible sans souris |
| Lecteurs d'écran | NVDA + JAWS + VoiceOver — labels ARIA |
| Direction RTL | Layout miroir pour arabe, icônes adaptées (flèches), tests manuels avec utilisateur arabophone |
| Internationalisation | Formats date `dd/mm/yyyy` (FR) ↔ `dd/mm/yyyy` (AR) — chiffres indo-arabes optionnels |

### 8.6 Pipeline CI/CD avec gates de qualité

```
Push → CI (GitHub Actions ou GitLab CI banque)
  ├─ Lint (ESLint, Prettier, Stylelint)
  ├─ TypeCheck (tsc --noEmit)
  ├─ Test unit + composant + intégration (Vitest)
  ├─ Coverage check (≥ 80 %)
  ├─ SAST (Semgrep, SonarQube)
  ├─ Dépendances (Snyk, npm audit)
  ├─ Build (Vite + esbuild)
  ├─ Build conteneurs (Docker)
  ├─ Scan conteneurs (Trivy)
  ├─ Test E2E sur env staging (Playwright)
  ├─ DAST sur env staging (OWASP ZAP baseline)
  ├─ Approval (manuel pour prod)
  └─ Deploy (Helm rolling)
```

**Gate bloquant** :
- Coverage < 80 % → échec
- Test unitaire/intégration en échec → échec
- Vulnérabilité critique (SonarQube/Snyk/Trivy) → échec
- Build échec → échec

**Gate non bloquant** (warning) :
- E2E flaky → alerte mais build passe (analyse manuelle)
- Code smell SonarQube → alerte mais build passe

---

## 9. Gestion des incidents et notifications

### 9.1 Classification des incidents

| Sévérité | Critère | Délai notification |
|---|---|---|
| **S1 — Critique** | Indisponibilité service > 1h en heures ouvrées, fuite données personnelles avérée, compromission identifiée | DGSSI 24h, BAM 24h, CNDP sans délai si données affectées |
| **S2 — Majeur** | Indisponibilité 30 min – 1h, dégradation perf significative, vulnérabilité critique exploitable | DGSSI 24h (selon évaluation), BAM 24h |
| **S3 — Modéré** | Dégradation < 30 min, vulnérabilité haute non exploitable | Reporting mensuel à BAM/DGSSI |
| **S4 — Mineur** | Bug applicatif sans impact sécurité ou disponibilité | Pas de notification externe |

### 9.2 Procédure de réponse aux incidents

```
Détection (monitoring Prometheus/Sentry/SIEM banque)
       │
       ▼
Triage (RSSI banque + dev solo)
       │
       ▼
Classification S1-S4
       │
       ▼
Confinement (isolation, désactivation, etc.)
       │
       ▼
Notification (selon sévérité — § 9.3)
       │
       ▼
Investigation (forensic, timeline, root cause)
       │
       ▼
Remédiation (patch, rollback, configuration)
       │
       ▼
Communication interne (utilisateurs banque, sponsors)
       │
       ▼
Post-mortem (RCA, leçons apprises, plan correctif)
       │
       ▼
Archivage (rapport, preuves, journal)
```

### 9.3 Notifications obligatoires

#### Bank Al-Maghrib (circulaire 03/W/2016 art. 23)

| Délai | 24h |
|---|---|
| Format | Note formelle au directeur de la supervision bancaire |
| Contenu | Description, impact, mesures prises, plan d'action |
| Suivi | Rapport définitif sous 30 jours |

#### DGSSI (Loi 05-20 art. 8)

| Délai | 24h |
|---|---|
| Format | Plateforme DGSSI + courrier formel |
| Contenu | Description technique, vecteur, IoC, mesures, demande d'assistance si nécessaire |
| Suivi | Rapport définitif sous 30 jours, intégration retex DGSSI |

#### CNDP (Loi 09-08 — pas de délai légal explicite mais bonnes pratiques)

| Délai | Sans délai indu (≤ 72h pour alignement RGPD interne) |
|---|---|
| Format | Courrier CNDP |
| Contenu | Nature violation, données affectées, personnes affectées, mesures prises |
| Personnes affectées | Information individuelle si risque élevé pour droits et libertés |

#### AMMC (banque cotée)

| Délai | Sans délai si information privilégiée affectée (Règlement Général AMMC) |
|---|---|
| Format | Communication financière |
| Contenu | Selon impact patrimonial / image |

### 9.4 SOC banque

**Hypothèse** : la banque dispose d'un SOC interne ou externalisé. Le parapheur :
- Émet des logs structurés JSON vers le SIEM banque (Splunk, Elastic, QRadar selon banque)
- Bénéficie de la surveillance H24 du SOC
- Reçoit alertes en cas de pattern anormal

**Si SOC banque indisponible** : monitoring autonome 8h/jour ouvré + alerting Sentry + on-call dev solo (compromis acceptable pour MVP, non OIV-conforme à terme → faire pression DSI pour intégration SOC banque dès P5).

---

## 10. Plan de continuité d'activité (PCA / PRA)

### 10.1 Exigences réglementaires

| Source | Exigence |
|---|---|
| BAM circulaire 5/W/2017 | RTO ≤ 4h, RPO ≤ 1h, test annuel obligatoire |
| Loi 05-20 OIV | Site secours physiquement distant |
| AMMC / Code gouvernance | Continuité critique pour clôture périodique |

### 10.2 Architecture PRA solo + cloud Maroc

**Configuration cible** (équilibre simplicité solo vs OIV) :

```
PRODUCTION (Site primaire)              SECOURS (Site secondaire)
DC Maroc Casablanca (ou cloud banque)   DC Maroc Rabat (ou cloud banque DR)
─────────────────────────              ─────────────────────────
PostgreSQL primary                      PostgreSQL standby (streaming replication)
Redis cluster                           Redis cluster réplica
MinIO/stockage                          MinIO/stockage (réplication)
App pods Kubernetes (ou VMs)            App pods en cold standby
                                        ↓
                                        Sauvegardes quotidiennes chiffrées
                                        (S3 banque ou stockage objet)
                                        Rétention : 30j + mensuel 12 mois + annuel 7 ans
```

**RTO / RPO mesurés** :

| Composant | RTO | RPO | Mécanisme |
|---|---|---|---|
| Application | 1-2h | n/a (stateless) | Helm rollback + nouveau cluster |
| PostgreSQL | 2-4h | < 1h | Streaming replication + WAL archive S3 |
| Redis | 30 min | n/a (cache) | Reconstruction depuis PG |
| Stockage objets | 1h | 1h | Réplication asynchrone |
| **Global** | **≤ 4h** | **≤ 1h** | Conforme BAM 5/W/2017 |

### 10.3 Plan de tests PRA annuel

**Calendrier** : test annuel Q4 (avant clôture annuelle banque).

**Scénarios** :

| # | Scénario | Mesure |
|---|---|---|
| 1 | Crash PostgreSQL primary | Failover réplica < 5 min |
| 2 | Indisponibilité datacenter primaire | Bascule complète DC secondaire < 4h |
| 3 | Corruption logique données (suite incident) | Restauration backup point-in-time < 2h |
| 4 | Indisponibilité plateforme e-sign banque | Mode dégradé : signature physique avec scan a posteriori |
| 5 | Indisponibilité GED | Mode dégradé : stockage local provisoire 7 jours puis batch d'archivage |

**Livrable** : PV de test signé RSSI + transmis BAM sur demande.

### 10.4 Mode dégradé en cas d'indisponibilité plateformes externes

**Indisponibilité plateforme e-sign** :
1. Détection automatique (health check ping toutes les 60 s)
2. Bascule mode dégradé : interface affiche bannière, signataires informés
3. Document tagué "à signer physiquement", impression signature manuscrite, scan
4. Reprise : workflow ré-injecte le document signé manuscritement avec hash + photo de preuve

**Indisponibilité GED** :
1. Stockage temporaire dans MinIO local (TTL 7 jours)
2. Job batch nocturne tente le ré-archivage en GED
3. Alerte DSI si non résolu sous 48h
4. Si non résolu sous 7 jours : escalade COPIL et plan B (archivage temporaire conforme dans bucket dédié + transfert ultérieur)

---

## 11. Risques cotés P×I

Échelle : P (1-5) × I (1-5) = 1-25.

### 11.1 Risques sécurité techniques

| ID | Risque | P | I | P×I | Mitigation |
|---|---|---|---|---|---|
| RSEC-01 | Compromission compte AD signataire (phishing) | 3 | 5 | **15** | MFA OTP obligatoire, sensibilisation, détection comportementale SOC, désactivation rapide |
| RSEC-02 | Injection SQL ou XSS exploitable | 2 | 5 | **10** | Prisma ORM, Zod validation, CSP, SAST/DAST en CI |
| RSEC-03 | Fuite documents via journal/log non chiffré | 2 | 5 | **10** | Masking automatique pino, revue logs avant prod, chiffrement at-rest |
| RSEC-04 | Faille bibliothèque tierce (npm) | 4 | 4 | **16** | Snyk + Dependabot, alertes critiques traitées 7j, revue trimestrielle |
| RSEC-05 | Faille image conteneur | 3 | 4 | **12** | Trivy bloquant en CI, base distroless, mise à jour mensuelle |
| RSEC-06 | DDoS interne (saturation API) | 2 | 3 | **6** | Rate limiting Kong, circuit breaker |
| RSEC-07 | Compromission secrets (clés JWT, DB) | 2 | 5 | **10** | Vault, rotation trimestrielle, jamais en code, masking logs |
| RSEC-08 | Modification non autorisée journal audit | 2 | 5 | **10** | Append-only, hash chaîné, WORM si possible, alertes intégrité |

### 11.2 Risques conformité

| ID | Risque | P | I | P×I | Mitigation |
|---|---|---|---|---|---|
| RCONF-01 | Refus / mise en demeure CNDP | 2 | 5 | **10** | Pré-consultation CNDP P0, déclaration formelle exhaustive |
| RCONF-02 | Mise en demeure DGSSI (audit OIV non conforme) | 2 | 5 | **10** | Audit DGSSI annuel, plan d'action sous 30j |
| RCONF-03 | Sanction BAM (gouvernance SI non conforme) | 2 | 5 | **10** | Audit cabinet inscrit BAM annuel, comité sécurité actif |
| RCONF-04 | Sanction AMMC (contrôle interne défaillant — banque cotée) | 2 | 5 | **10** | Journal audit immuable, extraction sous 5j, audit interne semestriel |
| RCONF-05 | Plateforme e-sign non agréée DGSSI → SEQ impossible | 3 | 4 | **12** | Audit P0, plan B Barid eSign, niveau SEA suffisant 95 % cas |
| RCONF-06 | Transfert non autorisé hors Maroc | 1 | 5 | **5** | Hébergement Maroc strict, pas d'AWS UE par défaut |
| RCONF-07 | Conservation > 10 ans violation Loi 09-08 (minimisation) | 2 | 3 | **6** | Politique anonymisation post-10 ans automatique |
| RCONF-08 | Demande d'accès personne concernée non traitée < 30j | 2 | 3 | **6** | Ticketing SLA 25j, alerte 5j avant échéance |
| RCONF-09 | Information privilégiée AMMC affectée par incident parapheur | 1 | 5 | **5** | Le parapheur ne traite pas info privilégiée par construction |
| RCONF-10 | Incident SI majeur non notifié 24h | 2 | 5 | **10** | Procédure formalisée, RSSI banque pivot, runbook |

### 11.3 Risques opérationnels (mode solo)

| ID | Risque | P | I | P×I | Mitigation |
|---|---|---|---|---|---|
| ROP-01 | Indisponibilité dev solo (maladie, départ) | 3 | 5 | **15** | Sponsor banque, référent SI backup, doc exhaustive, repo banque |
| ROP-02 | Dérive qualitative / dette technique | 4 | 3 | **12** | Tests ≥ 80 %, SonarQube, audit externe semestriel |
| ROP-03 | Charge mentale → burnout | 3 | 4 | **12** | Cadence 4j/sem, jalons /2 sem, replanif trimestrielle |
| ROP-04 | Sous-estimation effort | 4 | 4 | **16** | Buffer 20 %, jalons fréquents |
| ROP-05 | Perte sponsorship banque | 3 | 5 | **15** | Lettre mission DG/DSI, reporting mensuel, jalons formels |
| ROP-06 | Indisponibilité plateforme e-sign banque | 3 | 4 | **12** | Plan B Barid eSign, mode dégradé signature physique |
| ROP-07 | Indisponibilité GED banque | 3 | 4 | **12** | Stockage temporaire 7j, mode dégradé |

### 11.4 Synthèse top 10 risques

| Rang | ID | P×I | Mitigation prioritaire |
|---|---|---|---|
| 1 | RSEC-04 | 16 | Snyk + Dependabot, traitement 7j |
| 2 | ROP-04 | 16 | Buffer 20 % + jalons /2 sem |
| 3 | RSEC-01 | 15 | MFA + sensibilisation + SOC |
| 4 | ROP-01 | 15 | Sponsor + backup + doc exhaustive |
| 5 | ROP-05 | 15 | Lettre mission DG/DSI |
| 6 | RSEC-05 | 12 | Trivy bloquant CI |
| 7 | RCONF-05 | 12 | Audit P0 + plan B Barid eSign |
| 8 | ROP-02 | 12 | Tests + SonarQube + audit semestriel |
| 9 | ROP-03 | 12 | Cadence soutenable |
| 10 | ROP-06 | 12 | Plan B + mode dégradé |

---

## 12. Plan d'action conformité et budget

### 12.1 Effort solo en JH conformité

| Activité | JH solo | Période |
|---|---|---|
| Pré-consultation CNDP (mail + RDV) | 2 | P0 |
| Audit P0 plateforme e-sign banque (agrément DGSSI) | 3 | P0 |
| Audit P0 plateforme GED banque (conformité BAM 5/W/2017) | 2 | P0 |
| Rédaction déclaration CNDP formelle | 3 | P5 |
| Dépôt CNDP + suivi instruction | 2 | P5-P9 |
| Cartographie risques + analyse EBIOS-RM (ou ISO 27005) | 4 | P5 |
| Politique de sécurité (PSSI applicative) | 2 | P5 |
| Mapping ISO 27001:2022 ↔ DNSSI ↔ contrôles parapheur | 3 | P6 |
| Procédures (incidents, droits CNDP, exercice droits) | 3 | P7 |
| Préparation audit cabinet inscrit BAM | 2 | P9 |
| Accompagnement audit cabinet BAM (5j cabinet) | 3 | P9 |
| Préparation pentest cabinet agréé DGSSI | 1 | P9 |
| Accompagnement pentest (8j cabinet) | 2 | P9 |
| Plan de remédiation post-audit + post-pentest | 5 | P9 |
| Tests PRA annuels | 2 | P10 |
| Documentation conformité (PSSI, runbook, incidents) | 4 | P5-P10 |
| **Total Build** | **43 JH** | – |
| Run /an : audit annuel + pentest annuel + revue PRA + maintenance déclaration | 8 JH/an | continu |

### 12.2 Budget cash conformité (en MAD)

| Poste | Montant |
|---|---|
| Pré-consultation CNDP | 0-3 000 |
| Déclaration formelle CNDP | 5 000 |
| Audit cabinet inscrit BAM (5 jours × 8 000) | 40 000 |
| Pentest cabinet agréé DGSSI (8 jours × 6 000) | 48 000 |
| Frais juridiques ad hoc (avocat conformité 2j × 5 000) | 10 000 |
| Total CAPEX conformité | **103 000 MAD** |
| Audit annuel cabinet BAM | 40 000 /an |
| Pentest annuel | 48 000 /an |
| Renouvellement déclaration CNDP (si modif majeure) | 0-5 000 /an |
| Total OPEX conformité /an | **88 000 - 93 000 MAD/an** |

### 12.3 Calendrier conformité aligné sur les phases

```
P0 (sem 1-4)     ───── Audit P0 plateformes + pré-consultation CNDP
P5 (sem 5-8)     ───── Déclaration CNDP + cartographie risques + PSSI
P6 (sem 9-20)    ───── Mapping contrôles + implémentation continue
P7 (sem 21-34)   ───── Procédures + intégrations sécurisées
P8 (sem 35-42)   ───── Renforcement + tests sécurité E2E
P9 (sem 43-48)   ───── Audit cabinet BAM + Pentest + plan remédiation
P10 (sem 49-54)  ───── Mise en prod + hypercare + premier test PRA
```

### 12.4 Critères GO / NO-GO conformité (porte P9 → P10)

**GO si tous remplis** :

- ✅ Déclaration CNDP **acceptée** (récépissé reçu)
- ✅ Audit cabinet inscrit BAM **clos sans réserve majeure** (réserves mineures admises avec plan d'action ≤ 3 mois)
- ✅ Pentest cabinet agréé DGSSI **sans vulnérabilité critique non corrigée**
- ✅ DNSSI : 100 % des mesures applicables au parapheur **mises en œuvre** (mesure documentée + preuve)
- ✅ ISO 27001 contrôles critiques **opérationnels** (10 contrôles top § 5.4)
- ✅ PCA testé avec succès en simulation
- ✅ Procédure notification incidents 24h validée par RSSI banque
- ✅ Convention de service interne avec plateformes e-sign + GED **signée**
- ✅ Correspondant CNDP banque **identifié et formé**
- ✅ Logs de production **dirigés vers SIEM banque** (ou SOC banque informé)

**NO-GO si l'un manque** : reprise, replanification, nouveau passage P9.

---

## 13. Annexes

### Annexe A — Glossaire

| Terme | Définition |
|---|---|
| **AMMC** | Autorité Marocaine du Marché des Capitaux |
| **BAM** | Bank Al-Maghrib (banque centrale) |
| **CNDP** | Commission Nationale de Contrôle de la Protection des Données à Caractère Personnel |
| **CNIE** | Carte Nationale d'Identité Électronique |
| **CSO** | Conseil de Conformité Charia (banques participatives) — n/a ici |
| **DGSSI** | Direction Générale de la Sécurité des Systèmes d'Information |
| **DNSSI** | Directive Nationale de la Sécurité des Systèmes d'Information |
| **DOC** | Dahir des Obligations et Contrats |
| **GED** | Gestion Électronique des Documents |
| **HSM** | Hardware Security Module |
| **ICE** | Identifiant Commun de l'Entreprise |
| **IF** | Identifiant Fiscal |
| **LAB-FT** | Lutte Anti-Blanchiment et Financement du Terrorisme |
| **OIV** | Opérateur d'Importance Vitale |
| **PSCo** | Prestataire de Services de Confiance |
| **PSSI** | Politique de Sécurité du Système d'Information |
| **RC** | Registre du Commerce |
| **RCA** | Root Cause Analysis |
| **RPO** | Recovery Point Objective |
| **RTO** | Recovery Time Objective |
| **SEA / SES / SEQ** | Signature Électronique Avancée / Simple / Qualifiée (Loi 43-20) |
| **SIEM** | Security Information and Event Management |
| **SOC** | Security Operations Center |
| **TSA** | Time Stamping Authority |
| **UTRF** | Unité de Traitement du Renseignement Financier |
| **WORM** | Write Once Read Many |

### Annexe B — Mapping ISO 27001:2022 ↔ DNSSI (extrait, contrôles critiques parapheur)

| ISO 27001:2022 | DNSSI | Statut parapheur |
|---|---|---|
| A.5.1 Politique sécurité | Ch. 1.1 | ✅ PSSI banque + complément applicatif |
| A.5.2 Rôles et responsabilités | Ch. 1.2 | ✅ RSSI, DPO/correspondant CNDP, dev solo |
| A.5.7 Threat intelligence | Ch. 2.3 | ✅ Veille DGSSI + Snyk |
| A.5.8 Sécurité dans gestion projet | Ch. 10.1 | ✅ Secure SDLC, DevSecOps |
| A.5.10 Utilisation acceptable | Ch. 3.4 | ✅ Charte banque |
| A.5.15 Contrôle d'accès | Ch. 5.1 | ✅ RBAC × ABAC |
| A.5.16 Identification | Ch. 5.2 | ✅ AD + MFA |
| A.5.17 Information authentification | Ch. 5.3 | ✅ AD politique + MFA |
| A.5.18 Droits d'accès | Ch. 5.4 | ✅ Provisioning AD + revue trimestrielle |
| A.5.23 Sécurité services cloud | Ch. 9.5 | ✅ Hébergement Maroc, mTLS, audit |
| A.5.30 Préparation TIC continuité | Ch. 11.4 | ✅ PCA + test annuel |
| A.5.34 Confidentialité données personnelles | Ch. 5.6 | ✅ Loi 09-08 + chiffrement + RBAC |
| A.6.3 Sensibilisation | Ch. 3.2 | ✅ Sensibilisation banque (cible 100 %) |
| A.7.4 Surveillance physique | Ch. 7.1 | ✅ Datacenter Tier III+ |
| A.8.1 Périphériques utilisateurs | Ch. 5.5 | ✅ MDM banque |
| A.8.2 Privilèges d'accès | Ch. 5.4 | ✅ Principe moindre privilège |
| A.8.5 Authentification sécurisée | Ch. 5.2 | ✅ MFA |
| A.8.7 Protection malware | Ch. 8.1 | ✅ Trivy + EDR banque |
| A.8.9 Configuration | Ch. 8.4 | ✅ IaC + baselines durcies |
| A.8.10 Suppression données | Ch. 8.6 | ✅ Politique 10 ans + anonymisation |
| A.8.12 Prévention fuite données | Ch. 9.4 | ✅ DLP banque + RBAC + chiffrement |
| A.8.13 Sauvegardes | Ch. 11.2 | ✅ Backup quotidien + chiffré + offsite |
| A.8.16 Surveillance activités | Ch. 8.5 | ✅ Logs centralisés SIEM |
| A.8.21 Sécurité services réseau | Ch. 9.1 | ✅ Segmentation, mTLS |
| A.8.22 Ségrégation réseau | Ch. 9.2 | ✅ Zones de sécurité |
| A.8.23 Filtrage web | Ch. 9.3 | ✅ Filtrage banque |
| A.8.24 Cryptographie | Ch. 6 | ✅ AES-256, TLS 1.3, RS256 |
| A.8.25 Cycle vie développement sécurisé | Ch. 10.1 | ✅ Secure SDLC |
| A.8.26 Exigences sécurité applicative | Ch. 10.2 | ✅ Spec sécurité dans US |
| A.8.27 Architecture sécurisée | Ch. 10.3 | ✅ Modèle STRIDE |
| A.8.28 Codage sécurisé | Ch. 10.4 | ✅ SAST + revue + standards |
| A.8.29 Tests sécurité | Ch. 10.5 | ✅ DAST + pentest annuel |
| A.8.31 Séparation environnements | Ch. 10.6 | ✅ Dev/staging/prod cloisonnés |
| A.8.32 Gestion changements | Ch. 8.3 | ✅ CI/CD + approbation prod |
| A.8.33 Information de test | Ch. 10.7 | ✅ Données fictives en tests |

(35 contrôles sur 47 listés. Liste complète disponible sur demande.)

### Annexe C — Modèle déclaration CNDP

Cf. formulaire officiel CNDP : www.cndp.ma/fr/formalites-administratives/

Sections obligatoires :
1. Identification du responsable
2. Désignation du sous-traitant (si applicable)
3. Finalité du traitement
4. Catégories de données
5. Catégories de personnes concernées
6. Destinataires
7. Transferts hors Maroc (si applicable)
8. Durée de conservation
9. Mesures de sécurité (synthèse — ne pas joindre PSSI confidentielle)
10. Procédure d'exercice des droits

Pièces jointes : statuts banque, mandat correspondant CNDP.

### Annexe D — Sources réglementaires (URL)

- Loi 09-08 et décret 2-09-165 : www.cndp.ma
- Loi 43-20 : Bulletin Officiel n° 6912 du 17/09/2020
- Loi 05-20 et décret 2-21-406 : Bulletin Officiel n° 6904 du 23/07/2020
- Bank Al-Maghrib circulaires : www.bkam.ma
- AMMC : www.ammc.ma
- DGSSI / DNSSI : www.dgssi.gov.ma
- Code de commerce 15-95 : Secrétariat Général du Gouvernement
- DOC art. 417-1 et 417-2 : SGG, Code des Obligations et Contrats
- Loi 17-95 (sociétés anonymes) modifiée 20-19 : SGG
- Code Marocain de Bonnes Pratiques de Gouvernance d'Entreprise : CGEM + AMMC
- ISO 27001:2022 : iso.org
- OWASP Testing Guide v4.2 : owasp.org

### Annexe E — Liste des 12 décisions COPIL conformité

(Reprise depuis la révision contextuelle, avec focus P3.)

| ID | Décision | Recommandation |
|---|---|---|
| **D-R1** | Hébergement | Cloud interne banque > DC Maroc Tier III+ — **strict pour OIV** |
| **D-R2** | Niveau signature | SEA pour 100 % cas internes ; SEQ optionnelle si plateforme banque agréée DGSSI pour engagements > 5 M MAD |
| **D-R5** | Authentification | AD banque + MFA OTP (cohérent OIV) |
| **D-R12** | Sponsorship & gouvernance | Lettre mission DG/DSI + correspondant CNDP + référent RSSI + backup SI |

(Décisions D-R3, D-R4, D-R6 à D-R11 sont architecture/métier — détail dans P4 et P2 v2.)

---

*Fin de la Phase 3 v2 — Conformité & sécurité.*

**Prochaine livraison** : Phase 4 v2 — Architecture & intégration.
