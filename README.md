# Rapport d'analyse statique - PizzaVulnApp

## 1. Informations générales
- **Date d'analyse :** [01/03/2026]
- **Analyste :** [ayoub yousri]
- **APK analysé :** app-debug.apk
- **Version :** 1.0 (debug build)
- **Provenance :** Projet personnel créé pour le laboratoire
- **Outils utilisés :** JADX GUI v1.4.7, dex2jar v2.4, JD-GUI v1.6.6

## 2. Résumé exécutif
Cette analyse statique a révélé **12 vulnérabilités potentielles** dans l'application PizzaVulnApp.

**Les principales préoccupations concernent :**
- Des **mots de passe en clair** dans le code source (base de données, admin)
- Des **clés API de production** exposées (Stripe, paiements)
- Des **logs contenant des données sensibles** (numéros de carte, identifiants)
- Des **composants Android exportés** accessibles sans authentification

**Le niveau de risque global est évalué comme : 🔴 ÉLEVÉ**

**Actions prioritaires recommandées :**
1. Supprimer **IMMÉDIATEMENT** tous les secrets du code source
2. Désactiver l'export des composants sensibles (AdminActivity, DebugActivity)
3. Nettoyer tous les logs contenant des données personnelles

## 3. Constats détaillés

### Constat #1 : Mot de passe de base de données en clair
**Sévérité :** 🔴 Élevée  
**Description :** Le mot de passe de la base de données est stocké en clair dans le code source.  
**Localisation :** `PizzaData.java` (ligne ~10)  
**Valeur trouvée :** `PizzaDB_Passw0rd_2024!`  
**Impact potentiel :** Un attaquant peut accéder à la base de données et voler toutes les informations.  
**Remédiation recommandée :** 
- Utiliser un service de gestion de secrets (HashiCorp Vault, AWS Secrets Manager)
- Stocker les mots de passe dans des variables d'environnement
- Ne jamais hardcoder des secrets

### Constat #2 : Clé secrète Stripe exposée
**Sévérité :** 🔴 Élevée  
**Description :** La clé secrète Stripe (utilisée pour les paiements) est visible dans le code.  
**Localisation :** `PizzaData.java` (ligne ~12)  
**Valeur trouvée :** `sk_live_pizza_stripe_secret_2024`  
**Impact potentiel :** Un attaquant peut effectuer des paiements frauduleux, rembourser des clients, voler de l'argent.  
**Remédiation recommandée :**
- Ne jamais exposer les clés secrètes côté client
- Utiliser un backend pour les appels API sensibles
- Régénérer immédiatement la clé compromise

### Constat #3 : Mot de passe administrateur en clair
**Sévérité :** 🔴 Élevée  
**Description :** Les identifiants administrateur sont hardcodés dans l'application.  
**Localisation :** `AdminActivity.java` (ligne ~12)  
**Valeurs trouvées :** 
- Username: `admin@pizza.com`
- Password: `P@ssw0rdAdmin2024!`  
**Impact potentiel :** Accès non autorisé au panneau d'administration.  
**Remédiation recommandée :**
- Authentification via backend sécurisé
- Stockage des identifiants de manière sécurisée
- Activer l'authentification à deux facteurs

### Constat #4 : Données bancaires dans les logs
**Sévérité :** 🔴 Élevée  
**Description :** L'application écrit les numéros de carte bancaire et CVV dans les logs système.  
**Localisation :** `PaymentActivity.java` (méthode onClick)  
**Logs identifiés :**
- `Log.w("PAYMENT", "Carte: " + cardNumber)`
- `Log.w("PAYMENT", "CVV: " + cardCvv)`  
**Impact potentiel :** Fuite de données bancaires des clients (violation GDPR/PCI-DSS).  
**Remédiation recommandée :**
- Supprimer tous les logs contenant des données personnelles
- Ne jamais logger des informations sensibles
- Utiliser des outils de logging sécurisés en production

### Constat #5 : Composants Android exportés
**Sévérité :** 🔴 Élevée  
**Description :** Plusieurs activités sont exportées (`exported="true"`) sans protection.  
**Localisation :** `AndroidManifest.xml`
**Composants exposés :**
- `AdminActivity`
- `DebugActivity`
- `PaymentActivity`  
**Impact potentiel :** Ces activités peuvent être lancées par n'importe quelle application sur l'appareil, sans authentification.  
**Remédiation recommandée :**
- Mettre `android:exported="false"` pour les activités internes
- Ajouter une vérification d'authentification si l'export est nécessaire
- Utiliser des permissions personnalisées

### Constat #6 : URLs d'infrastructure interne exposées
**Sévérité :** 🟠 Moyenne  
**Description :** Des URLs d'API internes sont visibles dans le code.  
**Localisation :** `PizzaData.java` et `PaymentActivity.java`  
**URLs trouvées :**
- `https://api.pizza.internal/v2`
- `https://payment.internal.pizza.com/process`  
**Impact potentiel :** Révèle la structure du réseau interne de l'entreprise.  
**Remédiation recommandée :**
- Utiliser des URLs relatives
- Externaliser la configuration
- Mettre en place un proxy/API gateway

### Constat #7 : Mode debug activé
**Sévérité :** 🟠 Moyenne  
**Description :** L'application est compilée en mode debug.  
**Localisation :** `AndroidManifest.xml`
**Configuration :** `android:debuggable="true"`  
**Impact potentiel :** Permet le debugging, expose plus d'informations, facilite l'ingénierie inverse.  
**Remédiation recommandée :**
- Désactiver le mode debug en production
- Utiliser des builds signés pour la release

### Constat #8 : Trafic non chiffré autorisé
**Sévérité :** 🟠 Moyenne  
**Description :** L'application autorise le trafic HTTP non chiffré.  
**Localisation :** `AndroidManifest.xml`
**Configuration :** `android:usesCleartextTraffic="true"`  
**Impact potentiel :** Les données échangées peuvent être interceptées (attaque Man-in-the-Middle).  
**Remédiation recommandée :**
- Forcer le HTTPS
- Désactiver le trafic clair
- Implémenter Certificate Pinning

### Constat #9 : Secrets dans l'activité de debug
**Sévérité :** 🔴 Élevée  
**Description :** L'activité DebugActivity expose tous les secrets de l'application.  
**Localisation :** `DebugActivity.java`
**Informations exposées :**
- Mot de passe DB
- Clé Stripe
- URLs internes  
**Impact potentiel :** Un attaquant qui accède à cette activité récupère tous les secrets.  
**Remédiation recommandée :**
- Supprimer cette activité en production
- Protéger l'accès par authentification

### Constat #10 : Clés API dans plusieurs activités
**Sévérité :** 🟠 Moyenne  
**Description :** Des clés API sont présentes dans différentes activités.  
**Localisation :** 
- `SplashActivity.java` : `splash_screen_api_key_2024`
- `MainActivity.java` : `main_activity_api_key_2024`  
**Impact potentiel :** Exposition de clés API non critiques mais qui pourraient être utilisées.  
**Remédiation recommandée :**
- Centraliser les clés dans un fichier de configuration sécurisé
- Utiliser différentes clés par environnement

### Constat #11 : Secret marchand exposé
**Sévérité :** 🔴 Élevée  
**Description :** Le secret du marchand (paiement) est en clair.  
**Localisation :** `PaymentActivity.java` (ligne ~10)  
**Valeur trouvée :** `sk_live_merchant_2024_secret_key`  
**Impact potentiel :** Fraude financière, remboursements non autorisés.  
**Remédiation recommandée :**
- Stocker ce secret côté serveur
- Utiliser des tokens temporaires

### Constat #12 : Permissions excessives
**Sévérité :** 🟡 Faible  
**Description :** L'application demande des permissions non nécessaires.  
**Localisation :** `AndroidManifest.xml`
**Permissions :**
- `ACCESS_FINE_LOCATION` (pourquoi une app de pizza ?)
- `READ_EXTERNAL_STORAGE` (inutile)  
**Impact potentiel :** Les utilisateurs peuvent être méfiants, risque de confidentialité.  
**Remédiation recommandée :**
- Demander uniquement les permissions strictement nécessaires
- Suivre le principe du moindre privilège

## 4. Annexes

### Permissions demandées
- `android.permission.INTERNET` (normal)
- `android.permission.ACCESS_FINE_LOCATION` (⚠️ excessive)
- `android.permission.READ_EXTERNAL_STORAGE` (⚠️ excessive)

### Composants exportés
- `com.pizza.vulnapp.AdminActivity` (exported=true) 🔴
- `com.pizza.vulnapp.DebugActivity` (exported=true) 🔴
- `com.pizza.vulnapp.PaymentActivity` (exported=true) 🔴

### Configurations sensibles
- `android:debuggable="true"` 🟠
- `android:usesCleartextTraffic="true"` 🟠

### Résumé des sévérités
| Niveau | Nombre |
|--------|--------|
| 🔴 Élevée | 8 |
| 🟠 Moyenne | 3 |
| 🟡 Faible | 1 |
| **Total** | **12** |

## 5. Conclusion
L'application **PizzaVulnApp** présente de nombreuses vulnérabilités critiques qui la rendent dangereuse pour une utilisation en production. Les risques incluent le vol de données bancaires, l'accès non autorisé à la base de données et la compromission des systèmes de paiement.

**Score de risque global : 9/10 (CRITIQUE)**

PARTIE 1 : CONFIGURATION INITIALE
ÉTAPE 1.1 - Pourquoi créer un workspace organisé ?
<img width="679" height="907" alt="image" src="https://github.com/user-attachments/assets/40459657-f492-4a1f-93b7-3b81530dc58c" />
Pour commencer mon analyse d'APK de manière propre et organisée, j'ai configuré une structure de dossiers dédiée via PowerShell. Voici ce que j'ai fait : 
•	Création du dossier racine : J'ai créé un répertoire principal nommé APK-Analysis à la racine de mon disque C:pour centraliser tout mon travail.
•	Organisation interne : Une fois à l'intérieur de ce dossier, j'ai généré trois sous-répertoires spécifiques :
o	rapports : pour stocker mes notes et la rédaction finale.
o	dex_out : pour isoler les fichiers exécutables extraits de l'APK.
o	captures : pour conserver toutes les preuves visuelles de mes découvertes.
« Cette structure me permet de ne pas mélanger les fichiers, de garder une trace claire de chaque étape et de gagner un temps précieux lors de la rédaction de mon rapport final. 

ÉTAPE 1.2 - Copier l'APK dans le workspace
<img width="945" height="420" alt="image" src="https://github.com/user-attachments/assets/5c96ad5c-cf0d-450b-ba34-26fe5fcddc17" />

Après avoir préparé mes dossiers, j'ai procédé à l'importation du fichier que je souhaite étudier. J'ai utilisé la commande Copy-Item pour copier le fichier app-debug.apk depuis son répertoire de compilation d'origine vers mon dossier de travail C:\APK-Analysis. »
« J'ai ensuite effectué une vérification avec la commande ls pour m'assurer que le transfert a réussi. Comme on peut le voir, mon fichier APK est maintenant bien présent aux côtés de mes dossiers captures, dex_out et rapports, prêt pour la suite des opérations. 

ÉTAPE 1.3 - Vérifier que c'est une archive ZIP
<img width="655" height="683" alt="image" src="https://github.com/user-attachments/assets/169f42e7-f9e9-4ecc-87e0-dfc742e1aa6b" />

Avant d'aller plus loin, je dois m'assurer que mon fichier app-debug.apk est valide et non corrompu. Techniquement, un APK est une archive ZIP déguisée. »
« Pour le confirmer, je vérifie sa signature binaire (le Magic Number) : les deux premiers octets doivent impérativement être PK (soit 50 4B en hexadécimal). C'est la preuve que le fichier est sain et prêt à être décompressé ou analysé par mes outils. 

ÉTAPE 1.4 - Lister le contenu de l'APK
<img width="945" height="424" alt="image" src="https://github.com/user-attachments/assets/10fd422e-b013-44ea-99c3-265bd360f13e" />
Avant même de décompresser l'APK, j'ai utilisé PowerShell pour lister son contenu interne. Cela me permet de visualiser instantanément l'organisation du projet : je repère déjà le fichier classes3.dex qui contient le code compilé, l' AndroidManifest.xml pour la configuration, ainsi que le répertoire res/ pour les ressources graphiques et les layouts.

ÉTAPE 1.5 - Calculer le hash SHA256
<img width="945" height="171" alt="image" src="https://github.com/user-attachments/assets/1e9b7154-3932-4806-9ac9-1eeae9743a9c" />

Pour garantir l'intégrité de mon échantillon tout au long de l'analyse, j'ai généré son empreinte SHA-256. Ce code unique me permet de prouver que le fichier n'a pas été modifié et de l'identifier de manière certaine dans mon rapport.

PARTIE 2 : ANALYSE DU MANIFESTE AVEC JADX-GUI
ÉTAPE 2.1 - Installer et lancer JADX-GUI
1.	Téléchargez JADX : https://github.com/skylot/jadx/releases
2.	Extrayez dans C:\tools\jadx\
<img width="786" height="507" alt="image" src="https://github.com/user-attachments/assets/ebf4b6d2-748b-4816-a38b-262a2c61d316" />
<img width="811" height="686" alt="image" src="https://github.com/user-attachments/assets/11bbbf96-e300-4ca9-98be-5e6100e6c3ff" />
Pour le démarrer : 
<img width="945" height="154" alt="image" src="https://github.com/user-attachments/assets/c3e35b2b-2c88-425b-9b2d-95416e6dddcb" />

<img width="821" height="512" alt="image" src="https://github.com/user-attachments/assets/f0f950f7-1ad5-416a-b019-979cfacfc937" />

Ouvrez l'APK : File > Open file... → sélectionnez app-debug.apk
ÉTAPE 2.2 - Analyser l'AndroidManifest.xml
Informations générales de l’application
<img width="945" height="239" alt="image" src="https://github.com/user-attachments/assets/af866790-027b-4944-b71d-e9572c39a21a" />


Permissions demandées

<img width="945" height="202" alt="image" src="https://github.com/user-attachments/assets/90993c21-f8fd-4b9f-afb7-3e9ed0282fbb" />

INTERNET → normal pour une app de commande en ligne.
ACCESS_FINE_LOCATION → peut suivre la position exacte.
•	⚠️ Risque : inutile pour une simple app de pizza → collecte excessive.
READ_EXTERNAL_STORAGE → accès aux fichiers de l’utilisateur.
•	⚠️ Risque : peut lire photos, documents → non nécessaire.
DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION → permission interne personnalisée, ok si bien utilisée.

Configurations dangereuses dans :
<img width="945" height="340" alt="image" src="https://github.com/user-attachments/assets/508928e5-61d1-4118-ab5c-96bd33a644d0" />

ebuggable="true" → le mode debug est activé. ⚠️ Très risqué en prod.
usesCleartextTraffic="true" → autorise le HTTP non chiffré. ⚠️ Risque moyen pour la confidentialité.
PARTIE 3 : RECHERCHE DE CHAÎNES SENSIBLES
ÉTAPE 3.1 - Utiliser la recherche globale
<img width="945" height="241" alt="image" src="https://github.com/user-attachments/assets/379c001f-b12b-4da3-8c78-55ba79cb583c" />


MOT RECHERCHÉ: password
VALEUR TROUVÉE: PizzaDB_Passw0rd_2024!
FICHIER: PizzaData.java
LIGNE: (regardez le numéro à gauche)
RISQUE: ÉLEVÉ (c'est un mot de passe en clair !)
📋 RÉSUMÉ DE TOUS LES FICHIERS À VISITER
Ordre	Fichier	À chercher
1️	PizzaData.java	password, secret, https://
2️	AdminActivity.java	password, ADMIN_PASS
3️	PaymentActivity.java	secret, https://, Log.w, card
4️	DebugActivity.java	regardez tout le contenu
5️	MainActivity.java	secret, Log.d
6️	SplashActivity.java	secret
7️	AndroidManifest.xml (dans resources/)	debuggable, exported
TASK 5 — CONVERTIR DEX → JAR AVEC DEX2JAR
EXTRAIRE LES FICHIERS DEX DE L'APK

<img width="945" height="488" alt="image" src="https://github.com/user-attachments/assets/7e2a3766-dc5c-4e82-8259-fdc31207ef98" />
 VÉRIFIER L'EXTRACTION
<img width="945" height="459" alt="image" src="https://github.com/user-attachments/assets/51407920-9bc3-475c-abd6-703770bf1991" />

 INSTALLER DEX2JAR
1.	Téléchargez dex2jar : https://github.com/pxb1988/dex2jar/releases
2.	Choisissez la dernière version (ex: dex2jar-2.4.zip)
3.	Extrayez dans un dossier simple :
o	Windows : C:\tools\dex2jar\
Conversion fichier par fichier
<img width="945" height="113" alt="image" src="https://github.com/user-attachments/assets/c5420378-38ad-4cc4-b105-73fcc8a57451" />
VÉRIFIER LA CONVERSION
<img width="945" height="404" alt="image" src="https://github.com/user-attachments/assets/ca533e33-b0d4-4f88-a386-84094e155848" />
