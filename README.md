# Test d'installation — Plugin Happyneuron sur Windows on ARM

Vérifie en CI GitHub Actions que l'installeur NSIS du Plugin Happyneuron s'installe,
se lance et se désinstalle correctement sur **Windows 11 ARM64** (runner `windows-11-arm`),
avec un job témoin sur Windows x64 (`windows-latest`).

## Ce que le workflow vérifie

1. Installation silencieuse (`/S`) et code de sortie.
2. Présence de l'entrée de désinstallation dans le registre + fichiers installés.
3. Architecture du binaire installé (x86 / x64 / ARM64) — sur ARM64, un binaire
   x64 tourne sous émulation (Prism), c'est attendu tant qu'il n'y a pas de payload `app-arm64`.
4. Lancement de l'app : process vivant après 30 s, ports en écoute du serveur local,
   requête HTTPS de fumée sur ces ports.
5. Présence du certificat racine dans les magasins `Root`.
6. Désinstallation silencieuse et suppression du dossier.

Les logs applicatifs (`%APPDATA%\Plugin Happyneuron`, …) sont uploadés en artifact à chaque run.

## Fournir l'installeur au workflow (2 options)

**Option A — asset de release (simple).** ⚠️ Sur un repo public, l'asset est
téléchargeable par n'importe qui. Acceptable si l'installeur est déjà distribué
publiquement aux utilisateurs finaux, sinon préférer l'option B.

```bash
cp "/c/Users/adm-gmasset/Downloads/Plugin Happyneuron(1).exe" /tmp/Plugin-Happyneuron-1.4.0.exe
gh release create installer /tmp/Plugin-Happyneuron-1.4.0.exe \
  --repo DromadaireEtDemi/pluginOnArm \
  --title "Installeur 1.4.0" --notes "Plugin Happyneuron 1.4.0 (x86 stub, payloads ia32+x64)"
```

**Option B — URL secrète.** Héberger l'exe sur un stockage privé (S3 présigné longue
durée, etc.) et créer le secret ; le workflow l'utilise en priorité si présent :

```bash
gh secret set INSTALLER_URL --repo DromadaireEtDemi/pluginOnArm --body "https://…"
```

## Lancer le test

```bash
gh workflow run test-install-woa --repo DromadaireEtDemi/pluginOnArm
gh run watch
```

Ou via l'onglet *Actions* → *test-install-woa* → *Run workflow*.

## Limites du test CI

- Session non interactive : pas de validation visuelle de l'UI (tray, fenêtres).
  Pour ça : VM Windows 11 ARM sur Mac Apple Silicon, ou vrai portable Snapdragon.
- Le runner tourne en admin ; un poste utilisateur standard peut se comporter
  différemment (élévation UAC pour le certificat machine, ACL…).
- Les performances sous émulation ne sont pas mesurées ici.
