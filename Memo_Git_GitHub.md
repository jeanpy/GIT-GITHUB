# Mémo Git et GitHub

**Dossier de travail → index → commit local → dépôt distant.** Tu modifies le fichier, `git add` prépare son contenu, `git commit` enregistre la version, `git push` transmet les commits. Lance les commandes depuis le dépôt. Remplace les noms d’exemple par ceux du projet.

## Commencer

| Je veux… | Commande |
|---|---|
| Créer un dépôt dans le dossier actuel | `git init -b main` |
| Définir mon identité pour ce projet | `git config user.name "Prénom Nom"` puis `git config user.email "prenom@example.org"` |
| Copier un dépôt existant dans un nouveau dossier | `git clone URL` puis entrer dans le dossier créé |
| Connecter Git à mon compte GitHub en HTTPS | `gh auth login --hostname github.com --git-protocol https --web` puis `gh auth setup-git` |
| Associer mon dépôt local à un dépôt GitHub vide | `git remote add origin URL` |
| Vérifier l’adresse et publier ma branche principale | `git remote -v` puis `git push -u origin main` |

## Proposer une modification

```bash
git switch main
git pull --ff-only
git switch -c docs/astreinte
# Modifier README.md dans l’éditeur et enregistrer.
git diff
git add README.md
git diff --staged
git commit -m "Ajouter les consignes d’astreinte"
git push -u origin docs/astreinte
```

**Sur GitHub** — Ouvrir une pull request avec `base: main` et `compare: docs/astreinte`. Relire **Files changed**. Pour répondre à un commentaire, corriger le fichier sur la même branche, créer un commit et lancer `git push`. Après la fusion, exécuter `git switch main` puis `git pull --ff-only`.

## S’orienter et synchroniser

| Question | Commande |
|---|---|
| Sur quelle branche suis-je ? Quels fichiers ont changé ? | `git status -sb` |
| Que reste-t-il à préparer ? Qu’ai-je déjà sélectionné ? | `git diff` puis `git diff --staged` |
| Quelles branches suivent un dépôt distant ? | `git branch -vv` |
| Comment les branches se rejoignent-elles ? | `git log --oneline --graph --decorate --all -15` |
| Qu’a changé le dernier commit ? | `git show HEAD` |
| Quels commits ont modifié mon fichier ? | `git log --oneline -- README.md` |
| Quel était son contenu au commit précédent ? | `git show HEAD~1:README.md` |
| Que contient le serveur depuis mon dernier échange ? | `git fetch origin` puis `git log --oneline HEAD..origin/main` pour examiner les commits de main absents de ma branche |
| Comment mettre ma branche à jour si elle est seulement en retard ? | `git pull --ff-only` sur la branche qui suit sa correspondante distante |

## Corriger selon la situation

| Situation | Geste |
|---|---|
| Fichier préparé trop tôt | `git restore --staged README.md` retire le fichier de l’index et garde les modifications sur disque. |
| Modification locale à abandonner | Relire `git diff -- README.md` puis `git restore -- README.md`. Cette commande efface les modifications non indexées du fichier suivi et reprend le contenu de l’index. |
| Commit partagé à annuler | Trouver son identifiant avec `git log --oneline`, puis `git revert IDENTIFIANT`. Relire le nouveau commit avant de le pousser. |
| Conflit pendant un merge | `git status` → éditer le fichier et supprimer les marqueurs → `git add FICHIER` → `git diff --staged` → `git commit`. |
| Fusion en cours à abandonner | `git merge --abort`. Commencer ses fusions avec un répertoire de travail propre facilite le retour au point de départ. |

**Lire `git status --short`** — Première colonne : index. Seconde : dossier de travail. `??` fichier non suivi · ` M` modification sur disque · `M ` modification préparée · `MM` fichier préparé puis modifié de nouveau · `UU` conflit de modification des deux côtés.

[Référence des commandes Git](https://git-scm.com/docs) · [Pull requests sur GitHub](https://docs.github.com/fr/get-started/using-github/hello-world).
