# MobSF : Analyse avec Architecture Distribuée (Windows & Mobexler)

## 📌 Présentation

Ce document présente la mise en place de **MobSF (Mobile Security Framework)** pour réaliser une **analyse statique** et une **analyse dynamique** de l'application **DIVA (Damn Insecure and Vulnerable App)**.

Nous utilisons ici une **architecture distribuée** particulière, conçue pour faire communiquer un émulateur hébergé sur une machine hôte Windows avec une instance MobSF conteneurisée sur une machine virtuelle Linux (Mobexler) :

```text
L'architecture finale :
Machine Hôte Windows (192.168.100.12)
 └── Android Studio AVD (émulateur)
      └── ADB écoute sur 127.0.0.1:5555
               ↑
          Port Proxy Windows
          redirige 0.0.0.0:5555 → 127.0.0.1:5555
               ↑
          Réseau WiFi (192.168.100.x)
               ↑
Machine Mobexler VirtualBox (192.168.100.103)
 └── MobSF (Natif)
      └── ADB connecté à 192.168.100.12:5555
```

L'objectif est de montrer un workflow simple pour :

* configurer le proxy ADB sur Windows
* connecter un émulateur Android distant à MobSF (VM)
* analyser une APK en statique et dynamique
* utiliser les tests TLS/SSL et les outils Frida intégrés
* consulter logs et résultats

---

## 🛠️ Prérequis

Avant de commencer, voici la configuration requise :

* **Machine Hôte (Windows - 192.168.100.12)** :
  * **Android Studio** avec au moins un **AVD**
  * Une règle de proxy/port-forwarding configurée pour exposer ADB
  * L'application **DIVA** (ou autre APK)
* **Machine Virtuelle (Mobexler - 192.168.100.103)** :
  * Instanciation complète de **Mobexler** (qui intègre nativement MobSF)
  * Connectivité réseau vers l'hôte Windows

---

## 1. Préparation de l'environnement (Émulateur et Proxy Windows)

Sur la machine Windows, assurez-vous qu'un **Android Virtual Device (AVD)** est démarré via le Device Manager d'Android Studio. Cet émulateur reste indispensable pour l'analyse dynamique.

![Android Device Manager](./Images/2.png)

**Figure 1 :** Émulateurs Android disponibles sur Windows.

**Redirection de port ADB :**
Par défaut, ADB écoute sur `127.0.0.1:5555`. Pour rendre l'émulateur accessible à la machine virtuelle, un tunnel proxy a été configuré sur Windows. Le trafic reçu sur `0.0.0.0:5555` est directement redirigé vers l'interface locale `127.0.0.1:5555`.

---

## 2. Configuration et Lancement de MobSF (sur VM Mobexler)

Sur votre machine Mobexler (192.168.100.103), MobSF est généralement déjà pré-installé. Vous avez simplement besoin d'indiquer l'adresse de votre proxy ADB Windows avant de le démarrer.

Depuis le répertoire de MobSF, exportez la variable d'environnement puis lancez le script serveur `run.sh` :

```bash
cd /home/mobexler/Mobile-Security-Framework-MobSF  # Ou le chemin de l'installation existante
export MOBSF_ANALYZER_IDENTIFIER=192.168.100.12:5555
./run.sh 0.0.0.0:8000
```

MobSF initialise son environnement sur la VM et démarre le serveur web sur le port **8000**, tout en reliant nativement ADB à travers le réseau WiFi.

![Logs de démarrage MobSF](./Images/4-2.png)

**Figure 2 :** Logs de démarrage MobSF (serveur sur le port 8000).


---

## 3. Analyse Statique

Accédez à `http://127.0.0.1:8000`, et importez votre APK (ex: **DivaApplication.apk**). MobSF génère un rapport de sécurité contenant le score, les activités/services exportés, le code décompilé (Java/Smali) et le manifeste.

![Analyse statique DIVA](./Images/5.png)

**Figure 6 :** Résultat de l'analyse statique.

---

## 4. Analyse Dynamique et Tests

Cliquez sur **Start Dynamic Analysis** pour préparer l'instrumentation. MobSF met en place un proxy, Frida, et des scripts d'observation. Pendant l'analyse, vous pouvez interagir avec l'application sur l'émulateur, consulter les **logs Logcat**, et lancer des **tests TLS/SSL**.

![Préparation dynamique](./Images/6.png)

**Figure 7 :** Interface de l'analyse dynamique.

![Logs Logcat](./Images/7.png)

**Figure 8 :** Consultation des logs Logcat.

Les tests TLS/SSL évaluent la configuration réseau (misconfiguration, pinning, trafic en clair).

![Exécution tests TLS](./Images/8.png)

**Figure 9 :** Exécution des tests SSL.

![Résultats tests TLS](./Images/14.png)

**Figure 10 :** Résultats du module TLS/SSL.

L'interaction manuelle avec l'émulateur permet de déclencher des vulnérabilités (Credentials, Input Validation...).

![Écran API Credentials](./Images/9.png)

**Figure 11 :** Découverte de credentials API.

![Écran Input Validation](./Images/10.png)

**Figure 12 :** Problèmes de validation d'entrée.

---

## 5. Instrumentation Avancée avec Frida

MobSF inclut des **scripts Frida auxiliaires** (ex: bypass de détection d'émulateur) qui peuvent être injectés dynamiquement dans des activités ciblées via **Spawn**, **Inject**, ou **Attach**.

![Chargement script Frida](./Images/11.png)

**Figure 13 :** Chargement d'un script de bypass.

![Injection et sélection activité](./Images/12.png)

**Figure 14 :** Instrumentation via sélection d'activité.

