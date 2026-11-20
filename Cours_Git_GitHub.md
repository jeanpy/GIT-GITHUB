# Git et GitHub en pratique

**Mission ·** Tu travailles dans une ESN. L’équipe d’astreinte veut retrouver la dernière procédure de contrôle de Nginx, savoir qui l’a modifiée et relire les changements avant leur diffusion. Tu vas livrer cette procédure sous forme de fichier Markdown et conserver la trace de chaque décision.

**Public et durée ·** Débutants à l’aise avec un terminal. Compter environ 2 heures, défi compris. Les exercices fonctionnent en solo ou en binôme. Aucun serveur Nginx n’est nécessaire. Les commandes Nginx et HTTP seront écrites dans la documentation.

**Poste de travail ·** Utilise Bash sous Debian, WSL ou Git Bash sous Windows, un éditeur de texte et Git 2.28 ou ultérieur. Pour la partie en ligne, prévois un compte GitHub personnel ou de formation autorisé à créer un dépôt, et GitHub CLI (`gh`) pour la connexion HTTPS. Sous Debian, les paquets se nomment `git` et `gh`. Sous Windows, installe Git for Windows et GitHub CLI puis rouvre Git Bash.

## TP 1 · Créer le carnet d’exploitation (15 min)

**Objectif** — Enregistrer une première version sur ton poste. Git conserve l’historique dans le dossier caché `.git`. GitHub hébergera ensuite une copie du dépôt pour l’équipe.

Dans un répertoire de travail, crée un **nouveau** dossier. Garde le même terminal pour la suite.

```bash
git --version
mkdir carnet-nginx
cd carnet-nginx
git init -b main
```

Configure l’identité qui figurera dans les commits de ce dépôt. Remplace les deux valeurs. Tu peux utiliser l’adresse de confidentialité fournie dans les paramètres e-mail de ton compte GitHub. L’identité du commit et la connexion à GitHub se règlent séparément.

```bash
git config user.name "Prénom Nom"
git config user.email "prenom@example.org"
printf '%s\n' '# Carnet d’exploitation Nginx' '' '## Vérifier le service' 'Consulter l’état avec `systemctl status nginx`.' > README.md
git status --short
```

**À observer** — `?? README.md` signifie que Git voit un nouveau fichier. Prépare son contenu dans l’**index**, la sélection destinée au prochain commit.

```bash
git add README.md
git diff --staged
git commit -m "Documenter la vérification du service Nginx"
git log -1 --oneline
git status --short
```

**Contrôle** — Le journal affiche un identifiant court suivi du message. Le dernier `status --short` reste vide. Tu viens de créer un **commit**, une version de référence du projet.

## TP 2 · Choisir exactement ce que contient un commit (15 min)

L’équipe souhaite un contrôle HTTP et un résultat attendu. Prépare le contrôle, puis ajoute le résultat **après** le premier `git add`.

```bash
printf '%s\n' '' '## Vérifier HTTP' 'Exécuter `curl -I http://localhost/`.' >> README.md
git add README.md
printf '%s\n' 'Pour la page du lab, le code attendu est 200.' >> README.md
git status --short
git diff
git diff --staged
```

**À expliquer** — Pourquoi la ligne concernant le code 200 apparaît-elle dans un seul des deux écarts ? Dans `MM README.md`, la première colonne décrit l’index par rapport au dernier commit. La seconde décrit le fichier sur disque par rapport à l’index. Un `git add` sélectionne le contenu présent à cet instant.

Retire le fichier de l’index, vérifie que tes deux ajouts sont toujours sur disque, puis enregistre-les ensemble.

```bash
git restore --staged README.md
git diff
git add README.md
git diff --staged
git commit -m "Ajouter le contrôle HTTP et son résultat attendu"
git show --stat HEAD
```

**Mini-exercice** — Crée un journal de test local et indique à Git de l’ignorer.

```bash
printf '%s\n' '*.log' > .gitignore
printf '%s\n' 'Test HTTP du lab' > essai.log
git status --short --ignored
git add .gitignore
git commit -m "Ignorer les journaux de test locaux"
```

**Contrôle** — `!! essai.log` signale un fichier ignoré. Le fichier `.gitignore` est partagé avec l’équipe. Une règle d’exclusion concerne les fichiers qui ne sont pas déjà suivis.

## TP 3 · Publier le dépôt et vérifier sa copie (20 min)

1. Dans GitHub, choisis **New repository**. Nomme le dépôt **carnet-nginx** et choisis la visibilité **Private** pour cet exercice.
2. Crée-le vide. Laisse décochés le README, la licence et le modèle de fichier gitignore. L’historique initial existe déjà sur ton poste.
3. Dans le terminal, connecte GitHub CLI par navigateur, puis configure Git pour utiliser cette connexion.

```bash
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
gh auth status
```

Copie l’URL HTTPS affichée par GitHub. Dans la commande suivante, remplace toute l’URL d’exemple par celle du dépôt.

```bash
git remote add origin https://github.com/VOTRE-COMPTE/carnet-nginx.git
git remote -v
git push -u origin main
git branch -vv
```

**Contrôle** — Actualise la page du dépôt. Le README s’affiche avec ses titres et ses commandes. Dans le terminal, `main` suit désormais `origin/main`. L’option `-u` a établi ce lien pour les prochains échanges.

**Vérifier la livraison comme un collègue** — Depuis la racine du projet, clone le dépôt dans un dossier voisin. Le dernier ordre consulte cette copie sans quitter ton dossier initial.

```bash
git clone "$(git remote get-url origin)" ../carnet-lecture
git -C ../carnet-lecture log -1 --oneline
```

Le dernier identifiant doit correspondre à celui de ton dépôt initial. La copie contient le README et le fichier d’exclusion. Le journal de test est resté sur ton poste.

## TP 4 · Faire relire une modification avec une pull request (25 min)

**Demande de l’astreinte** — Ajouter la commande qui affiche les vingt dernières lignes du journal du service. Une **branche** permet de préparer cette modification pendant que `main` conserve la procédure en vigueur.

```bash
git switch -c docs/journal
printf '%s\n' '' '## Consulter les événements récents' 'Afficher les vingt dernières entrées avec `journalctl -u nginx -n 20`.' >> README.md
git diff
git add README.md
git commit -m "Documenter la consultation du journal Nginx"
git push -u origin docs/journal
```

1. Ouvre **Pull requests → New pull request**. Sélectionne **base = main** et **compare = docs/journal**. Examine les lignes ajoutées.
2. Clique sur **Create pull request**. Donne-lui le titre « Documenter le journal Nginx ». Dans la description, explique quand l’astreinte consultera ce journal.
3. Dans **Files changed**, repère la commande. En binôme, le collègue ayant accès au dépôt dépose un commentaire sur cette ligne. En solo, dépose toi-même ce commentaire de relecture : « Peut-on afficher le résultat directement dans le terminal, sans pagination ? »
4. Dans ton éditeur, sur la branche `docs/journal`, ajoute `--no-pager` à la commande du README. Enregistre le fichier, puis pousse la correction.

```bash
git diff
git add README.md
git commit -m "Afficher le journal sans pagination"
git push
```

5. Retrouve le second commit dans **la même pull request**. Vérifie la ligne corrigée dans **Files changed**. Dans **Conversation**, choisis **Merge pull request**, puis **Confirm merge**. Le parcours suppose un dépôt personnel avec l’option de commit de fusion activée.
6. Ramène la procédure validée sur ton poste.

```bash
git switch main
git pull --ff-only
git log --oneline --graph --decorate -6
```

**Contrôle** — Le README de `main` contient `--no-pager` et la pull request porte la mention **Merged**. La copie de lecture du TP 3 conserve son ancienne version jusqu’à sa propre synchronisation. Mets-la à jour avec `git -C ../carnet-lecture pull --ff-only`.

## TP 5 · Résoudre un conflit reproductible (20 min)

Deux collègues veulent préciser **la même ligne** de diagnostic. Simule leurs propositions dans deux branches locales issues de la même version. Exécute les commandes depuis `carnet-nginx`, après le TP 4. Le fichier `diagnostic.md` est créé pour l’exercice.

```bash
git switch -c atelier/conflit main
printf '%s\n' '# Diagnostic' 'Premier contrôle à effectuer.' > diagnostic.md
git add diagnostic.md
git commit -m "Créer la fiche de diagnostic"
git switch -c atelier/http
printf '%s\n' '# Diagnostic' 'Vérifier la réponse HTTP.' > diagnostic.md
git commit -am "Proposer le contrôle HTTP"
git switch atelier/conflit
printf '%s\n' '# Diagnostic' 'Vérifier l’état du service.' > diagnostic.md
git commit -am "Proposer le contrôle du service"
git merge atelier/http
```

**Arrête-toi au message de conflit.** Les deux versions dérivent du même texte. Git te demande laquelle conserver. `git status --short` affiche `UU diagnostic.md`. Ouvre le fichier. Entre `<<<<<<< HEAD` et `=======` se trouve ta version actuelle. Entre `=======` et `>>>>>>> atelier/http` se trouve la proposition entrante.

**Décision de l’équipe** — Conserver les deux contrôles dans leur ordre d’exécution. Dans l’éditeur, remplace le contenu du fichier par le texte ci-dessous et enregistre.

```markdown
# Diagnostic

1. Vérifier l’état du service.
2. Vérifier la réponse HTTP.
```

```bash
git add diagnostic.md
git diff --staged
git diff --cached --check
git commit -m "Réunir les deux contrôles de diagnostic"
git log --oneline --graph --decorate -6
git status --short
git switch main
```

**Contrôle** — Le graphe de la branche d’atelier montre deux chemins réunis par un commit de fusion. En revenant sur `main`, tu retrouves la procédure publiée. Pour consulter la résolution depuis `main`, exécute `git show atelier/conflit:diagnostic.md`.

## TP 6 · Corriger une erreur en gardant sa trace (10 min)

Une consigne erronée a été enregistrée. Entraîne-toi sur une branche d’essai. `HEAD` désigne ici le dernier commit de la branche active.

```bash
git switch -c atelier/correction main
printf '%s\n' 'Le code attendu pour la page du lab est 404.' >> README.md
git commit -am "Indiquer 404 comme code attendu"
git show HEAD
git revert --no-edit HEAD
git log -2 --oneline
git diff main HEAD -- README.md
git switch main
```

**Contrôle** — Le journal conserve l’erreur et sa correction. La dernière comparaison reste vide : le README a retrouvé le contenu de `main`. Pour corriger un commit déjà partagé, cette méthode conserve un historique que les collègues peuvent synchroniser.

## Défi autonome · Livrer une fiche d’astreinte (15 min)

L’ESN demande une fiche qu’un collègue puisse suivre lorsqu’un utilisateur signale une page inaccessible. Depuis `main` à jour, crée une branche `docs/astreinte`, rédige `astreinte.md` et ajoute dans le README un lien relatif vers cette fiche. Sépare la rédaction de la fiche et son référencement en deux commits. Fais relire puis fusionner la proposition sur GitHub.

**Recette de livraison**

- Depuis le README affiché sur GitHub, le lien ouvre la fiche.
- La fiche présente les contrôles du service, de la réponse HTTP et du journal. Chaque contrôle indique ce que doit observer le collègue.
- L’onglet **Commits** de la pull request montre les deux étapes. Sa description explique comment vérifier le résultat.
- Depuis la copie `carnet-lecture`, un `git pull --ff-only` récupère la fiche validée.

**Coup de pouce** — En Markdown, le lien s’écrit `[Consulter la fiche d’astreinte](astreinte.md)`. Pour retrouver le déroulé d’une contribution, reprends le TP 4 ou consulte le mémo.

## Si une manipulation bloque

| Symptôme | Vérification | Action |
|---|---|---|
| `not a git repository` | Affiche le dossier actuel avec `pwd`. | Reviens dans le dossier `carnet-nginx` du TP 1. |
| `Author identity unknown` | Exécute `git config user.email`. | Configure le nom et l’adresse du TP 1, puis relance le commit. |
| `remote origin already exists` | Lis `git remote -v`. | Si l’URL est correcte, passe au push. Pour corriger une erreur de saisie, utilise `git remote set-url origin URL` avec l’URL du dépôt. |
| `Repository not found` | Compare l’URL avec celle de GitHub et vérifie le compte dans `gh auth status`. | Utilise le compte ayant accès au dépôt et corrige l’URL si nécessaire. |
| Push refusé avec `non-fast-forward` | Lance `git fetch origin`, puis `git log --oneline --graph --all -12`. | Si ta branche est seulement en retard, utilise `git pull --ff-only`. Si elle a divergé sur ta branche de travail, termine le travail local en cours, fusionne la branche distante correspondante et relis le résultat avant de pousser. |

**Pour approfondir** — [Manuel Git en français](https://git-scm.com/book/fr/v2) · [Première pull request sur GitHub](https://docs.github.com/fr/get-started/using-github/hello-world) · [Connexion avec GitHub CLI](https://cli.github.com/manual/gh_auth_login).
