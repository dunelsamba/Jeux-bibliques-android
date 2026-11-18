# Jeux bibliques – application Android

Ce dossier est un projet Android complet. Le jeu est intégré dans l'application et fonctionne sans Internet.

Deux fichiers peuvent en sortir :

| Fichier | À quoi il sert |
|---|---|
| `JeuxBibliques-partage.apk` | À envoyer par WhatsApp, Bluetooth, e-mail… et à installer directement sur un téléphone |
| `JeuxBibliques-playstore.aab` | À déposer sur Google Play (le Play Store n'accepte plus les APK, seulement les AAB) |

Il faut un ordinateur avec les outils Android pour les fabriquer. Deux façons de faire, au choix.

---

## Option A – Sans rien installer : GitHub compile pour vous (gratuit)

1. Créez un compte sur https://github.com puis un nouveau dépôt (bouton **New**), par exemple `jeux-bibliques`.
2. Dans le dépôt : **Add file → Upload files**, glissez **tout le contenu** de ce dossier, puis **Commit changes**.
   - Vérifiez que le fichier `.github/workflows/build-apk.yml` est bien présent dans le dépôt. Certains navigateurs n'envoient pas les dossiers commençant par un point : dans ce cas, faites **Add file → Create new file**, tapez le nom `.github/workflows/build-apk.yml` et collez le contenu du fichier.
3. Ouvrez l'onglet **Actions**. La compilation démarre seule (environ 5 minutes). Sinon, cliquez sur **Construire l'application Android → Run workflow**.
4. Quand la ligne devient verte, cliquez dessus et téléchargez **JeuxBibliques-android** en bas de la page. Le zip contient `JeuxBibliques-partage.apk`.

Pour obtenir aussi la version Play Store signée, ajoutez la clé de signature (voir plus bas), puis relancez **Run workflow**. Le zip contiendra en plus `JeuxBibliques-signe.apk` et `JeuxBibliques-playstore.aab`.

## Option B – Avec Android Studio (sur votre PC)

1. Installez Android Studio : https://developer.android.com/studio
2. **File → Open** et choisissez ce dossier. Attendez la fin de la synchronisation (barre en bas).
3. APK de partage : **Build → Build App Bundle(s) / APK(s) → Build APK(s)**. Le fichier est dans `app/build/outputs/apk/debug/`.
4. Version Play Store : **Build → Generate Signed App Bundle or APK → Android App Bundle**. L'assistant permet de créer la clé de signature.

---

## Partager l'APK par téléphone

Envoyez `JeuxBibliques-partage.apk` par WhatsApp, Bluetooth, Telegram ou e-mail. Sur le téléphone qui le reçoit :
1. Ouvrir le fichier.
2. Accepter **Autoriser l'installation depuis cette source** quand Android le demande.
3. Si Play Protect affiche un avertissement (application inconnue), choisir **Installer quand même**.

Si une personne installe d'abord l'APK de partage puis veut la version du Play Store, elle doit désinstaller l'APK avant : les deux ne sont pas signés avec la même clé.

## Créer la clé de signature (une seule fois, à conserver précieusement)

La clé prouve que les mises à jour viennent de vous. Si vous la perdez, il faut passer par le support Google pour la remplacer.

Dans un terminal (la commande `keytool` est fournie avec Java ou Android Studio) :

```
keytool -genkeypair -v -keystore release.jks -alias jeuxbibliques -keyalg RSA -keysize 2048 -validity 10000
```

**Avec Android Studio** : créez à la racine du projet un fichier `keystore.properties` :

```
storeFile=release.jks
storePassword=VOTRE_MOT_DE_PASSE
keyAlias=jeuxbibliques
keyPassword=VOTRE_MOT_DE_PASSE
```

**Avec GitHub** : ne déposez jamais `release.jks` dans le dépôt. Allez dans **Settings → Secrets and variables → Actions → New repository secret** et créez :

| Nom du secret | Valeur |
|---|---|
| `KEYSTORE_BASE64` | le fichier `release.jks` encodé en base64 (commande ci-dessous) |
| `KEYSTORE_PASSWORD` | le mot de passe du fichier |
| `KEY_ALIAS` | `jeuxbibliques` |
| `KEY_PASSWORD` | le mot de passe de la clé |

Sous Windows (PowerShell), cette commande copie le base64 dans le presse-papiers :

```
[Convert]::ToBase64String([IO.File]::ReadAllBytes("release.jks")) | Set-Clipboard
```

---

## Publier sur Google Play

1. **Compte développeur** : https://play.google.com/console (25 $ une seule fois, pièce d'identité demandée).
2. **Compte personnel créé après le 13 novembre 2023** : Google exige un **test fermé avec au moins 12 testeurs pendant 14 jours d'affilée** avant la publication publique. Un compte « organisation » (église, association, entreprise avec numéro D-U-N-S) en est dispensé.
3. **Créer l'application**, puis remplir **Contenu de l'application** :
   - Politique de confidentialité : mettez en ligne `play-store/politique-confidentialite.html` et collez son adresse.
   - Sécurité des données : aucune donnée collectée ni partagée.
   - Classification du contenu : questionnaire IARC (aucune violence, aucun achat).
   - Public cible : si vous incluez les moins de 13 ans, les règles « Familles » de Google s'appliquent (l'application les respecte : pas de pub, pas de collecte).
4. **Fiche du Play Store** : les textes sont dans `play-store/fiche-play-store.md`, l'icône, la bannière et 5 captures dans `play-store/`.
5. **Test fermé**, puis **Production** : déposez `JeuxBibliques-playstore.aab`.

## Avant le premier envoi sur Google Play

- L'identifiant de l'application est `app.jeuxbibliques.quiz`. Il devient **définitif** après le premier envoi. Pour le changer, modifiez `applicationId` dans `app/build.gradle`.
- À chaque mise à jour, augmentez `versionCode` (1, 2, 3…) et `versionName` dans `app/build.gradle`.

## Modifier les questions

Le jeu se trouve dans `app/src/main/assets/www/index.html`. Les questions sont en haut du script : `QZ` (quiz), `WHO` (Qui suis-je ?), `VF` (Vrai ou faux), `JEO` (Jeopardy). Après modification, recompilez.

## Caractéristiques techniques

- Android 8.0 et plus (minSdk 26), cible Android 16 (targetSdk 36) comme l'exige Google Play depuis le 31 août 2026.
- Aucune autorisation demandée, pas même Internet.
- L'écran reste allumé pendant le jeu. Le bouton retour revient à l'écran précédent.
