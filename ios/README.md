# Portage iOS — état de préparation

**Aucune version iOS jouable ni IPA n'est disponible dans cette branche.**
Le fork et l'analyse des dépendances sont préparés ; le portage du moteur,
la compilation et les essais sur appareil restent à réaliser.

Base examinée : `Buku313/Skate3-Mobile` au commit
`e1b28c185d578c22e3a630f6f3b13ed123462a43`, avec ReXGlue
`edd4344723ecac3ffa18c5dcd2fcc268f468ff9e`.

## Fichiers nécessaires

Le dépôt ne contient pas le jeu ni les sources générées à partir du jeu.
La recompilation statique nécessite les fichiers d'une copie Xbox 360 :

- `game/default.xex` ;
- `game/data/webkit/EAWebkit.xex` ;
- les autres données du jeu dans `game/` pour l'exécution ;
- le paquet Title Update 3, fourni séparément via
  `SKATE3_TITLE_UPDATE_PACKAGE`.

Un ISO seul doit d'abord être extrait. Les exécutables et la mise à jour doivent
correspondre à la version prise en charge par le projet Android. Les avoir est
nécessaire, mais ne garantit pas que le portage iOS fonctionnera.
Ne pas ajouter ces fichiers ni les sources recompilées au dépôt Git.

## Adaptations à réaliser

| Élément | Constat dans les sources | Travail restant |
| --- | --- | --- |
| Compilation | Le CMake principal classe toutes les cibles Apple comme macOS et lie Cocoa. Le SDK exige l'identifiant de compilateur Clang et rejette AppleClang. | Ajouter une cible iOS ARM64 distincte, un bundle, une chaîne de compilation compatible et séparer les outils de génération macOS des bibliothèques iOS. |
| Exécution | `rex/platform.h` reconnaît déjà iOS. Des adaptations mémoire, SDL et fibres ARM64 existent. Le CMake ne sélectionne les fibres ARM64 que pour Android. | Raccorder et compiler ces chemins sur iOS ; vérifier la réservation mémoire et les threads sur appareil. |
| Entrée et cycle de vie | `windowed_app_main_sdl.cpp` inclut `SDL_main.h` seulement pour Android. | Utiliser l'entrée UIKit/SDL iOS et vérifier pause, reprise, audio et pression mémoire. |
| Graphismes | Le rendu utilise Vulkan ; le staging Apple actuel copie des bibliothèques macOS. | Intégrer MoltenVK pour iOS, vérifier le chargement Vulkan et les fonctions requises, puis tester les shaders et la présentation Metal. |
| Import des fichiers | Le sélecteur Apple appelle `NSOpenPanel` via Cocoa. | Ajouter un sélecteur UIKit, les accès aux documents et le stockage dans le conteneur de l'application. |
| Mise à jour du jeu | Le chemin Apple lance un processus `curl` avec `std::system`. | Remplacer ce téléchargement par une API iOS et conserver la vérification des fichiers. |
| Commandes | L'interface tactile Android dépend de son intégration Android. | Adapter les commandes tactiles et tester une manette sur iOS. |
| Installation | Aucun projet/bundle iOS installable n'est fourni par cette branche. | Configurer identifiant, équipe, signature et provisioning, puis installer et vérifier le démarrage réel. |

## Environnement vérifié

Le Mac dispose de Xcode 26.6, CMake et XcodeGen. Un iPhone appairé est détecté.
La commande `security find-identity -v -p codesigning` n'a trouvé aucune identité
de signature valide lors de la préparation. Aucun build ni test de gameplay iOS
n'a été exécuté. Aucun fichier du jeu n'a été fourni.

Pour préparer la signature, connecter son compte Apple dans **Xcode > Settings >
Apple Accounts**. L'équipe et les capacités nécessaires seront à vérifier avec
la future cible iOS ; ne pas promettre la compatibilité de toutes les capacités
mémoire avec une équipe personnelle avant essai.

## Critères avant de déclarer le portage testable

1. Générer localement le code du jeu à partir des fichiers compatibles.
2. Compiler et lier le moteur, ses dépendances et le jeu pour `iphoneos/arm64`.
3. Signer le bundle et l'installer sur l'iPhone.
4. Atteindre une partie, vérifier affichage, audio, commandes et sauvegarde.
5. Vérifier la reprise après passage en arrière-plan et relever les limites
   de mémoire, stabilité et performances sur l'appareil testé.

## Références

- [Projet Android et prérequis des données](https://github.com/Buku313/Skate3-Mobile)
- [Intégration de MoltenVK](https://github.com/KhronosGroup/MoltenVK/blob/main/Docs/MoltenVK_Runtime_UserGuide.md)
- [Exécution sur appareil avec Xcode](https://developer.apple.com/documentation/xcode/running-your-app-on-simulated-or-physical-devices)
