# Roadmap de Développement Backend - MedVault

## 📋 Vue d'ensemble
Système de gestion sécurisée des dossiers médicaux - Architecture DevSecOps
Backend: Python 3.12 + Flask + SQLAlchemy + PostgreSQL

---

## 🎯 Phase 1: Initialisation & Configuration du Projet

### 1.1 Structure du projet
- [ ] Créer la structure de dossiers
  ```
  MedVault/
  ├── app/
  │   ├── __init__.py
  │   ├── models/
  │   ├── routes/
  │   ├── services/
  │   ├── middleware/
  │   ├── utils/
  │   └── config.py
  ├── tests/
  ├── requirements.txt
  ├── .env
  └── run.py
  ```

### 1.2 Configuration de base
- [ ] Créer `requirements.txt` avec les dépendances
  - Flask==3.0.0
  - Flask-SQLAlchemy==3.1.1
  - Flask-JWT-Extended==4.6.0
  - Flask-CORS==4.0.0
  - psycopg2-binary==2.9.9
  - bcrypt==4.1.2
  - pyotp==2.9.0
  - pycryptodome==3.19.0
  - python-dotenv==1.0.0
  - pytz==2023.3

- [ ] Créer `config.py` avec les configurations
  - Configuration développement/production
  - Clés secrètes (JWT, AES)
  - Configuration PostgreSQL

- [ ] Créer `.env` pour les variables d'environnement
  - DATABASE_URL
  - SECRET_KEY
  - JWT_SECRET_KEY
  - AES_ENCRYPTION_KEY

### 1.3 Initialisation Flask
- [ ] Créer `app/__init__.py` avec l'application Flask
- [ ] Configurer SQLAlchemy
- [ ] Configurer JWT Extended
- [ ] Configurer CORS
- [ ] Créer `run.py` pour lancer l'application

---

## 🔐 Phase 2: Authentification (Endpoints Register & Login)

### 2.1 Modèle User
- [ ] Créer `app/models/user.py`
  - Champs: id, username, email, password_hash, role, otp_secret, is_active, created_at, etablissement_id
  - Enum pour les rôles: ADMIN, MEDECIN, INFIRMIER, PATIENT
  - Méthode hash_password()
  - Méthode verify_password()
  - Méthode generate_otp_secret()
  - Méthode verify_otp()
  - Méthode is_first_login() (pour détecter onboarding OTP)

### 2.2 Base de données
- [ ] Créer les tables avec SQLAlchemy
- [ ] Script de migration initial
- [ ] Seed data (utilisateur admin par défaut)

### 2.3 Services d'authentification
- [ ] Créer `app/services/auth_service.py`
  - register_user(): validation, hachage mot de passe, génération OTP secret
  - login_user(): vérification mot de passe, vérification OTP, génération JWT
  - generate_totp_qr_code(): pour configurer l'app authenticator (onboarding)
  - validate_otp_setup(): validation du premier scan OTP lors de l'onboarding

### 2.4 Routes d'authentification
- [ ] Créer `app/routes/auth.py`
  - POST `/api/auth/register` (Admin uniquement)
    - Input: username, email, password, role, etablissement_id
    - Output: user_id, message
    - Validation: email unique, password strength
    - Génère OTP secret mais ne l'affiche pas encore
  
  - POST `/api/auth/login`
    - Input: username, password, otp_code
    - Output: access_token, refresh_token, user_info, qr_code (si first login)
    - Logic: bcrypt password verify + PyOTP verify + JWT generation
    - Si first login: retourne QR code pour onboarding OTP
  
  - POST `/api/auth/setup-otp`
    - Input: user_id, otp_code (premier code après scan QR)
    - Output: success message
    - Valide la configuration OTP et marque first_login = False

### 2.5 Tests d'authentification
- [ ] Tests unitaires register
- [ ] Tests unitaires login
- [ ] Tests MFA OTP

---

## 👥 Phase 3: Gestion des Utilisateurs (RBAC)

### 3.1 Middleware d'autorisation
- [ ] Créer `app/middleware/auth_middleware.py`
  - jwt_required decorator
  - role_required decorator (vérification rôle)
  - get_current_user helper

### 3.2 Routes utilisateurs (Admin uniquement)
- [ ] Créer `app/routes/users.py`
  - GET `/api/users` (list tous utilisateurs - Admin)
  - GET `/api/users/<id>` (détails utilisateur - Admin)
  - PUT `/api/users/<id>/role` (modifier rôle - Admin)
  - DELETE `/api/users/<id>` (désactiver utilisateur - Admin)

### 3.3 Tests RBAC
- [ ] Tests middleware
- [ ] Tests permissions par rôle

---

## 📋 Phase 4: Modèles de Données Médicales

### 4.1 Modèle Patient
- [ ] Créer `app/models/patient.py`
  - Champs: id, user_id, nom, prenom, date_naissance, numero_securite_sociale, etablissement_id
  - Champs chiffrés: adresse, telephone, antecedents_medicaux (AES-256-GCM)
  - Champ key_version: "v1" (pour rotation future)

### 4.2 Modèle DossierMedical
- [ ] Créer `app/models/dossier_medical.py`
  - Champs: id, patient_id, medecin_id, created_at, updated_at
  - Champs chiffrés: diagnostic, traitement, notes, ordonnances (AES-256-GCM)
  - Champ integrity_hash: SHA-256 pour vérifier intégrité
  - Champ key_version: "v1" (pour rotation future)

### 4.3 Modèle AssignationPatient
- [ ] Créer `app/models/assignation.py`
  - Champs: id, patient_id, medecin_id, infirmier_id, date_assignation
  - Relation: un patient peut avoir plusieurs soignants assignés

### 4.4 Migration base de données
- [ ] Créer les nouvelles tables
- [ ] Relations entre modèles

---

## 🔒 Phase 5: Services de Chiffrement

### 5.1 Service de chiffrement AES-256-GCM
- [ ] Créer `app/services/encryption_service.py`
  - encrypt_data(data, key): chiffrement AES-256-GCM
  - decrypt_data(encrypted_data, key): déchiffrement
  - generate_key(): génération clé AES
  - Clé unique stable dans variables d'environnement (MVP)
  - Structure prête pour rotation future (key_version)

### 5.2 Service d'intégrité SHA-256
- [ ] Créer `app/services/integrity_service.py`
  - calculate_hash(data): calcul SHA-256
  - verify_hash(data, expected_hash): vérification intégrité
  - update_hash_on_change(): mettre à jour hash après modification

### 5.3 Tests chiffrement
- [ ] Tests encrypt/decrypt
- [ ] Tests intégrité SHA-256

---

## 📝 Phase 6: Journal d'Audit (Audit Trail)

### 6.1 Modèle AuditLog
- [ ] Créer `app/models/audit_log.py`
  - Champs: id, user_id, action, resource, resource_id, ip_address, timestamp
  - Champ: entry_hash (SHA-256 de l'entrée)
  - Champ: previous_hash (hash entrée précédente - chaînage)

### 6.2 Service d'audit
- [ ] Créer `app/services/audit_service.py`
  - log_action(): enregistrer action avec horodatage
  - chain_logs(): chaînage des logs (hash précédent inclus) - PRIORITÉ MVP
  - sign_logs(): signature RSA périodique des logs (production)
  - verify_log_integrity(): vérifier qu'aucun log n'a été modifié

### 6.3 Middleware d'audit automatique
- [ ] Créer `app/middleware/audit_middleware.py`
  - Log automatique de chaque requête
  - Capture: user, action, resource, IP, timestamp

### 6.4 Routes audit (Admin)
- [ ] Créer `app/routes/audit.py`
  - GET `/api/audit/logs` (liste logs - Admin)
  - GET `/api/audit/verify` (vérifier intégrité logs - Admin)

### 6.5 Tests audit
- [ ] Tests logging
- [ ] Tests chaînage
- [ ] Tests détection modification

---

## 🏥 Phase 7: Routes Dossiers Médicaux

### 7.1 Routes Patients
- [ ] Créer `app/routes/patients.py`
  - POST `/api/patients` (créer patient - Admin/Médecin)
  - GET `/api/patients` (liste patients - selon rôle)
  - GET `/api/patients/<id>` (détails patient - selon rôle)
  - PUT `/api/patients/<id>` (modifier patient - Admin/Médecin assigné)

### 7.2 Routes Dossiers Médicaux
- [ ] Créer `app/routes/dossiers.py`
  - POST `/api/dossiers` (créer dossier - Médecin)
  - GET `/api/dossiers/patient/<patient_id>` (dossier patient - selon rôle)
  - PUT `/api/dossiers/<id>` (modifier dossier - Médecin assigné)
  - GET `/api/dossiers/<id>/integrity` (vérifier intégrité - tous rôles)

### 7.3 Routes Assignations
- [ ] Créer `app/routes/assignations.py`
  - POST `/api/assignations` (assigner patient à médecin - Admin)
  - GET `/api/assignations/medecin/<medecin_id>` (patients assignés - Médecin)
  - DELETE `/api/assignations/<id>` (retirer assignation - Admin)

### 7.4 Tests dossiers médicaux
- [ ] Tests CRUD patients
- [ ] Tests CRUD dossiers
- [ ] Tests permissions par rôle
- [ ] Tests chiffrement/déchiffrement transparent

---

## 🚨 Phase 8: Gestion des Urgences (Accès Exceptionnel)

### 8.1 Service d'accès d'urgence
- [ ] Créer `app/services/emergency_service.py`
  - request_emergency_access(): demande accès urgence
  - grant_emergency_access(): approbation (Admin ou autre médecin)
  - log_emergency_access(): log spécial dans audit

### 8.2 Routes urgence
- [ ] POST `/api/emergency/request` (demander accès - Médecin)
- [ ] POST `/api/emergency/grant/<request_id>` (approuver - Admin)
- [ ] GET `/api/emergency/logs` (voir logs urgence - Admin)

### 8.3 Tests urgence
- [ ] Tests flux demande/approbation
- [ ] Tests logging urgence

---

## 🌐 Phase 9: Configuration TLS 1.3

### 9.1 Configuration HTTPS
- [ ] Générer certificat SSL (self-signed pour dev)
- [ ] Configurer Flask avec SSL context
- [ ] Forcer HTTPS (HSTS)
- [ ] Configuration TLS 1.3 uniquement

### 9.2 Tests TLS
- [ ] Vérifier connexion HTTPS
- [ ] Vérifier version TLS

---

## 🧪 Phase 10: Tests de Sécurité

### 10.1 Tests d'injection
- [ ] Tests SQL injection
- [ ] Tests XSS (si frontend)

### 10.2 Tests d'authentification
- [ ] Tests brute force login
- [ ] Tests session hijacking
- [ ] Tests JWT expiration

### 10.3 Tests d'autorisation
- [ ] Tests privilege escalation
- [ ] Tests horizontal/vertical access

### 10.4 Tests de chiffrement
- [ ] Tests force brute AES
- [ ] Tests integrity bypass

### 10.5 Tests d'audit
- [ ] Tests log tampering
- [ ] Tests log deletion

---

## 📚 Phase 11: Documentation

### 11.1 Documentation API
- [ ] Swagger/OpenAPI documentation
- [ ] Postman collection

### 11.2 Documentation technique
- [ ] README avec setup instructions
- [ ] Architecture documentation
- [ ] Security guidelines

---

## 🚀 Phase 12: Déploiement

### 12.1 Docker
- [ ] Dockerfile pour l'application
- [ ] docker-compose.yml (app + PostgreSQL)

### 12.2 CI/CD
- [ ] GitHub Actions pour tests
- [ ] Pipeline de déploiement

---

## 📝 Questions de Clarification - RÉPONSES

### 1. Gestion des secrets OTP
**Réponse**: QR Code affiché à l'écran lors de la première connexion (onboarding)
- L'administrateur crée le compte avec mot de passe temporaire
- Lors de la 1ère connexion, le système génère le secret OTP et affiche le QR Code
- L'utilisateur scanne avec Google Authenticator (ou autre)
- Le QR Code disparaît après validation, jamais réaffiché
- **Raison**: Email non sécurisé (circule en clair, reste stocké, risque interception)

### 2. Rotation des clés AES
**Réponse**: Pas pour MVP, mais structure prête (Security by Design)
- MVP: clé unique stable dans variables d'environnement
- Chaque dossier chiffré aura un champ `key_version: "v1"`
- Structure déjà prête pour gérer plusieurs clés si rotation future
- Pas de rechiffrement nécessaire pour MVP

### 3. Multi-établissements / Multi-sites
**Réponse**: Isolation logique (pas physique)
- Une seule base de données PostgreSQL
- Chaque utilisateur/patient a un champ `etablissement_id`
- Middleware filtre systématiquement par établissement
- Un médecin de la Clinique A ne voit que les patients de la Clinique A
- Infrastructure simple à déployer

### 4. Signature RSA des logs
**Réponse**: MVP = chaînage SHA-256 prioritaire
- **Priorité MVP**: Chaînage cryptographique SHA-256 (hash précédent inclus dans chaque entrée)
- **Amélioration production**: Signature RSA périodique (par bloc de logs)
- RSA en temps réel = trop gourmand pour MVP
- Chaînage SHA-256 = simple, rapide, très efficace

### 5. Frontend
**Réponse**: Uniquement backend API
- Pas de développement frontend dans ce projet
- Concentration sur l'API REST Flask

### 6. Base de données
**Réponse**: Setup dockerisé inclus
- PostgreSQL via Docker Compose
- Configuration complète dans docker-compose.yml

---

## Ordre de Priorité Suggéré

1. ✅ Phase 1: Initialisation (fondation)
2. ✅ Phase 2: Authentification (core security)
3. ✅ Phase 3: RBAC (access control)
4. ✅ Phase 5: Chiffrement (data protection)
5. ✅ Phase 4: Modèles médicaux (data structure)
6. ✅ Phase 7: Routes dossiers (core functionality)
7. ✅ Phase 6: Audit (compliance)
8. ✅ Phase 8: Urgences (edge case)
9. ✅ Phase 9: TLS (transport security)
10. ✅ Phase 10: Tests sécurité (validation)
11. ✅ Phase 11: Documentation (maintainability)
12. ✅ Phase 12: Déploiement (production)
