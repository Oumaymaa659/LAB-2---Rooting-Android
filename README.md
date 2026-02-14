# 📱 LAB-2 - Audit de Sécurité Android : Rooting & Environnement

## 🛡️ Introduction
Ce projet est un audit de sécurité méthodologique d'un environnement Android, réalisé dans le cadre d'un cursus de Cybersécurité . Il s'appuie sur les standards de l'industrie, notamment l'**OWASP MASVS** (Mobile App Security Verification Standard) et l'**OWASP MASTG** (Mobile App Security Testing Guide).

L'objectif principal est de comprendre en profondeur les mécanismes de sécurité d'Android, en particulier le processus de **Rooting**, la **Chaîne de Confiance** (Verified Boot), et la mise en place d'un environnement d'analyse de vulnérabilités contrôlé.

> [!WARNING]
> **DISCLAIMER** : Ce laboratoire est réalisé dans un but strictement **pédagogique et éthique**, au sein d'un environnement cloisonné (Lab). Les techniques présentées ici ne doivent être utilisées que sur des systèmes dont vous êtes propriétaire ou pour lesquels vous disposez d'une autorisation explicite.

---

## 🛠️ Outils Utilisés

Ce laboratoire repose sur les outils standards de l'écosystème Android et du Pentesting Mobile :

*   **Android Studio** (Gestionnaire AVD)
*   **ADB** (Android Debug Bridge)
*   **Fastboot** (Mode Bootloader)
*   **Magisk** (Solution de Rooting Systemless)
*   **Emulator** (QEMU/Goldfish)

---

## 📋 Étapes du Laboratoire

### 1. Rooter l'AVD
L'objectif est d'obtenir les privilèges "SuperUser" (root) sur l'émulateur Android.
*   **Méthode utilisée :** Utilisation de **Magisk** via le script **rootAVD**.
*   **Pourquoi ?** Pour pouvoir inspecter les dossiers système, modifier les fichiers protégés et utiliser des outils d'analyse dynamique (ex: Frida) sans restrictions.
*   **Commande clé :**
    ```bash
    ./rootAVD.sh List  # Lister les images disponibles
    ./rootAVD.sh Install <ImageID> # Patch de l'image boot.img
    ```

### 2. Fiche périmètre
Documenter l'environnement cible avant toute action.
*   **Cible :** AVD Pixel 4 (par exemple)
*   **API Level :** Android 11 (API 30) ou plus récent
*   **Architecture :** x86_64 (pour émulateur)
*   **Type d'image :** Google APIs (non-Production, build `userdebug` si possible)
*   **Outils installés :** Magisk Manager, Burp Suite (certificat CA).

### 3. Démarrer un AVD propre
Démarrer l'émulateur dans un état connu et stable.
*   **Commande :**
    ```bash
    emulator -avd <Nom_AVD> -writable-system -no-snapshot-load
    ```
*   **Vérification :** `adb devices` doit lister l'appareil.

### 4. Installer et lancer l'app de test
Déploiement de l'application cible (ex: application vulnérable type *AndroGoat* ou *InsecureShop*).
*   **Commande :**
    ```bash
    adb install -r application-cible.apk
    adb shell am start -n com.package.name/.MainActivity
    ```

### 5. Définir 3 scénarios simples
Exemples de vecteurs d'attaque à tester dans ce laboratoire :
1.  **Contournement de la détection de Root :** L'app refuse-t-elle de se lancer sur un appareil rooté ?
2.  **Extraction de données locales :** Trouver des clés API ou mots de passe dans `/data/data/<package>/`.
3.  **Analyse du trafic réseau :** Interception HTTPS (Man-in-the-Middle) malgré le SSL Pinning.

### 6. Lire Android Security
Se référer à la documentation officielle pour comprendre le modèle de sécurité.
*   **Source :** [Android Security and Privacy](https://source.android.com/security)
*   **Concepts clés :** Sandbox applicative, UIDs, Permissions, SELinux.

### 7. Verified Boot (idée générale + check AVD)
Le **Verified Boot** assure l'intégrité du logiciel de l'appareil au démarrage.
*   **Sur un émulateur rooté :** Cette chaîne de confiance est nécessairement brisée ou désactivée pour permettre le chargement de composants modifiés (comme Magisk).
*   **Indicateur :** Présence du message "Your device is corrupt" ou état `orange` au boot.
*   **Vérification :**
    ```bash
    adb shell getprop ro.boot.verifiedbootstate
    # Retourne souvent 'orange' ou 'red' après rooting
    ```

### 8. AVB (Android Verified Boot)
Mécanisme technique (basé sur `dm-verity`) qui vérifie l'intégrité des partitions (boot, system, vendor).
*   Structure `vbmeta` : Contient les hachages pour la vérification.
*   **Impact du Lab :** Pour rooter, nous devons souvent désactiver la vérification (disable-verity) ou signer une image modifiée avec une clé personnalisée.

### 9. Définir le rooting
**Rooter** signifie obtenir l'accès au compte utilisateur `root` (UID 0) sur le système Linux sous-jacent d'Android.
*   **Implications :** Contrôle total du matériel et du logiciel.
*   **Risque :** Brise le modèle de sécurité (Sandbox) ; une app malveillante root peut tout voler.

### 10. Intérêt labo (non opérationnel)
Pourquoi faire cela en Lab ?
*   Observer le comportement réel du système sans risquer un appareil physique personnel.
*   Comprendre les empreintes laissées par le root (fichiers `su`, packages `com.topjohnwu.magisk`).

### 11. Matrice de risques
Évaluation des risques liés à l'exécution d'applications sur un appareil rooté.

| Menace | Impact | Probabilité (Si rooté) |
| :--- | :--- | :--- |
| Vol de données bancaires | Critique | Élevée |
| Installation de Keylogger | Critique | Élevée |
| Modification du système | Majeur | Très Élevée |

### 12. Mesures défensives
Comment les développeurs protègent leurs apps (et comment on apprend à les tester) :
*   **Root Detection :** Vérification de la présence de binaires `su`.
*   **SafetyNet / Play Integrity API :** Attestation côté serveur de l'intégrité de l'appareil.
*   **Obfuscation :** (ProGuard/R8) pour compliquer le Reverse Engineering.

### 13. OWASP MASVS
Standard de vérification (Ce qu'il faut tester).
*   **Référence :** **MASVS-RESILIENCE** (Resilience Against Reverse Engineering and Tampering).
*   *Exemple :* "L'application détecte, et répond à, la présence d'un appareil rooté."

### 14. OWASP MASTG
Guide de test (Comment tester).
*   **Référence :** **MASTG-TEST-0001** (Testing for Root Detection).
*   Méthodologie pour identifier les checks anti-root et les contourner (Hooking via Frida).

### 15. Commandes de rooting (rappel synthèse)
Commandes ADB essentielles pour gérer l'accès root :
```bash
adb root       # Redémarre adbd en mode root (sur builds userdebug)
adb shell      # Ouvre un shell
su             # Dans le shell, passe en root (nécessite Magisk/SuperSU)
whoami         # Doit retourner 'root'
```

### 16. Traçabilité : fiche environnement
Garder une trace de l'état du lab pour la reproductibilité.
*   Noter la version de Magisk.
*   Noter le hash de l'APK testé.
*   Captures d'écran des configurations réussies.

### 17. Remise à zéro AVD
Procédure pour nettoyer l'émulateur après le lab.
*   Dans AVD Manager : Action > **Wipe Data**.
*   Permet de repartir sur une base saine pour le prochain audit.

### 18. Remise à zéro device labo
(Si appareil physique utilisé)
*   Flasher la **Factory Image** officielle via Fastboot pour supprimer toute trace de modification et reverrouiller le chargeur de démarrage (Bootloader Lock).

### 19. Livrables
Ce qui doit être rendu à la fin du laboratoire :
1.  Rapport d'audit (PDF).
2.  Preuves de concept (Screenshots, scripts Frida).
3.  Fiche de synthèse des vulnérabilités trouvées.

### 20. Checklist finale
- [ ] AVD rooté et fonctionnel.
- [ ] Accès ADB root confirmé.
- [ ] Application cible installée.
- [ ] Trafic réseau interceptable.
- [ ] Environnement nettoyé après usage.
