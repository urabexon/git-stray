# git-stray

[English](./README.md) · [日本語](./README.ja.md) · **Français**

Trouve le travail qui n'existe que sur votre disque.

```
$ git stray ~

git-stray  scanned 165 repositories

AT RISK  ~/courses/swift/Egg
    no remote configured — 1 commit exists only on this disk (last 558d ago)
    7 uncommitted files — untouched for 558d
AT RISK  ~/hackathon/team-20-app
    12 unpushed commits — main(12) (last 336d ago)
STALE    ~/work/old-service
    stash 1922d old — stash@{0}: WIP on feature/settings

2 at risk, 1 stale.
```

## Pourquoi

Tous les outils d'état de dépôt répondent à la question « qu'est-ce qui a
changé ? ». Aucun ne répond à la seule question qui compte le jour où un disque meurt :
**qu'est-ce qui se trouve ici et nulle part ailleurs ?**

Une branche jamais poussée, un dépôt auquel on n'a jamais associé de remote,
un stash vieux de cinq ans : rien de tout cela n'est cassé, donc rien
ne s'en plaint jamais. `git status` se tait, parce que localement tout va bien.
Ce sont simplement des fichiers qui n'existent qu'en un seul exemplaire.

git-stray parcourt tous les dépôts sous un chemin donné et ne signale que
cela : le travail sans seconde copie.

## Ce qu'il ne fait délibérément pas

**Il ne vous dit pas qu'un projet est ancien.**

L'inactivité n'est pas un problème. Vous avez le droit de ne pas toucher un
projet pendant huit mois ; c'est en général une question d'agenda, pas une
erreur. Un outil qui vous récite vos projets dormants est un outil que vous
finirez par désinstaller.

Les modifications non commitées et les stash ne sont donc signalés qu'au-delà
de `--days` (14 par défaut), et rien n'est jamais signalé au seul motif d'être
inactif. Les commits non poussés et les dépôts sans dépôt distant sont
toujours signalés, car ceux-là n'existent réellement qu'en un exemplaire.

## Installation

```sh
curl -o /usr/local/bin/git-stray \
  https://raw.githubusercontent.com/urabexon/git-stray/main/git-stray
chmod +x /usr/local/bin/git-stray
```

Tout exécutable nommé `git-*` présent dans votre `PATH` devient une
sous-commande git : vous pouvez donc l'invoquer avec `git stray`.

Nécessite bash 3.2 ou plus récent (le bash fourni avec macOS suffit) et git.

## Utilisation

```sh
git stray                  # analyse sous le répertoire courant
git stray ~                # analyse tout le répertoire personnel
git stray --days 30 ~      # ne signale que ce qui dépasse un mois
git stray --days 0 ~       # signale tout, même le plus récent
git stray --fetch ~        # rafraîchit les refs distantes d'abord (réseau, lent)
git stray --hidden ~       # inclut les dépôts des répertoires cachés
git stray --depth 6 ~      # profondeur de recherche (4 par défaut)
```

### Codes de sortie

| Code | Signification |
|------|---------------|
| `0`  | aucun travail n'existe en un seul exemplaire |
| `1`  | au moins un élément n'existe qu'en un seul exemplaire |

## Ce qu'il signale

| Gravité | Constat |
|----------|--------|
| AT RISK | Des commits sur une branche locale qu'aucune ref distante ne contient |
| AT RISK | Idem, sur un HEAD détaché |
| AT RISK | Un dépôt sans aucun remote — chacun de ses commits n'existe qu'ici |
| STALE | Des modifications non commitées de fichiers suivis, plus anciennes que `--days` |
| STALE | Des stash plus anciens que `--days` |

Les dépôts situés dans des répertoires cachés (`~/.nvm`, `~/.oh-my-zsh` et
consorts) sont ignorés par défaut : ce sont les clones d'autres personnes, pas
votre travail.

Les âges proviennent des dates de commit et de stash, ainsi que des dates de
modification des fichiers. Lorsqu'aucun fichier ne permet de lire une date —
une suppression, un renommage — l'âge se rabat sur la date du dernier commit
et s'affiche avec un `~`. Le constat n'est jamais silencieusement abandonné.

## Précision

La comparaison se fait avec les refs de suivi déjà présentes sur votre disque :
rapide et hors ligne, mais elle ne vaut que ce que vaut votre dernier `fetch`. Utilisez
`--fetch` pour les rafraîchir, au prix d'un aller-retour réseau par dépôt.

« Non poussé » est calculé par rapport à *toutes* les refs distantes, et non
par rapport à la branche amont configurée. Une branche sans amont défini, ou
poussée sous un autre nom, est donc correctement considérée comme poussée.

## Voir aussi

- [git-brink](https://github.com/urabexon/git-brink) — l'outil frère.
  git-brink empêche de partir ce qui ne devrait jamais partir ; git-stray
  trouve ce qui n'est jamais parti alors qu'il aurait dû.
- [gita](https://github.com/nosarthur/gita), [mr](https://myrepos.branchable.com/) —
  agissent sur plusieurs dépôts à la fois. Objectif différent : ils montrent
  tout, celui-ci ne montre que ce qui est en danger.
- [gitleaks](https://github.com/gitleaks/gitleaks) — analyse le contenu déjà
  commité à la recherche de secrets. Problème sans rapport, souvent confondu.

## Licence

[MIT](./LICENSE)
