# Marketplace Claude Code de l'organisation

Ce dépôt est **la marketplace Claude Code de votre organisation**, sur votre GitHub. Elle s'appelle
`pixevboard` et charge trois plugins :

| Plugin | D'où il vient |
| --- | --- |
| `pixev-board` | le dépôt GitHub `PixevBoard/board-marketplace-shared`, à la version que vous choisissez |
| `pixev-team` | le même dépôt, à la même version |
| `exemple-team` | ce dépôt, dossier `plugins/exemple-team` — vos propres agents, que vous éditez |

Vous en êtes propriétaire : vous choisissez quand monter de version, et vous y ajoutez vos agents.

> [!IMPORTANT]
> **`vX.Y.Z` n'est pas une version réelle.** Tant que le `ref` des deux entrées Pixev vaut
> `vX.Y.Z`, l'installation de `pixev-board` et de `pixev-team` échoue. Remplacez-le par la version
> qui vous a été communiquée (par exemple `v1.4.0`) avant le premier push — voir « Mise en place ».

## Prérequis du poste

- **Claude Code 2.1.294 ou plus** (`claude --version`) ;
- **git 2.25 ou plus** (`git --version`) : les plugins Pixev sont récupérés par un clone partiel
  (sparse-checkout), que les versions plus anciennes ne savent pas faire ;
- **`gh`**, la ligne de commande GitHub, pour l'authentification.

## Mise en place du dépôt de l'organisation

À faire une fois, par la personne qui administre la marketplace :

1. Créer un dépôt **privé** dans votre organisation GitHub, puis y copier **le contenu** de ce
   dossier à la racine : `.claude-plugin/`, `plugins/` et ce `README.md`.
2. **Renommer le plugin d'exemple** au nom de votre organisation, par exemple `acme-team` :
   - le dossier `plugins/exemple-team` ;
   - le champ `name` de `plugins/<votre plugin>/.claude-plugin/plugin.json` ;
   - l'entrée correspondante de `.claude-plugin/marketplace.json` (`name` et `source`).
3. **Remplacer `vX.Y.Z`** par la version voulue dans **les deux** entrées Pixev de
   `.claude-plugin/marketplace.json` (`pixev-board` et `pixev-team`). Les deux portent toujours la
   même version. N'ajoutez pas de champ `version` à ces entrées : la version affichée est celle du
   plugin lui-même, à la version choisie.
4. Vérifier, puis pousser :

   ```bash
   claude plugin validate --strict .
   git add . && git commit -m "Marketplace de l'organisation" && git push
   ```

Vos agents se rangent ensuite dans `plugins/<votre plugin>/agents/`, un fichier par agent, sur le
modèle de `exemple-agent.md`.

## Authentification du poste

Les deux dépôts lus — celui de votre organisation et `PixevBoard/board-marketplace-shared` — sont
**privés**. Chaque poste s'authentifie auprès de GitHub en HTTPS :

```bash
gh auth login        # compte GitHub de la personne, protocole HTTPS
gh auth setup-git    # git utilise ce compte pour cloner en HTTPS
```

- **Le compte doit lire les deux dépôts** : le dépôt de votre marketplace, et
  `PixevBoard/board-marketplace-shared`, ouvert à votre organisation par une équipe GitHub en
  lecture. Un seul jeton couvre les deux.
- **Mise à jour en arrière-plan** : Claude Code rafraîchit les marketplaces privées sans passer par
  `gh`, avec le jeton qu'il trouve dans `GH_TOKEN` (ou `GITHUB_TOKEN`). Exportez dans le profil
  shell le jeton du compte qui a fait `gh auth login` :

  ```bash
  export GH_TOKEN="$(gh auth token)"    # ou GITHUB_TOKEN
  ```

  - **Ce jeton doit lire les deux dépôts.** Une fois exporté, `GH_TOKEN` (ou `GITHUB_TOKEN`)
    **prime sur l'identifiant enregistré par `gh auth login`**, y compris pour git, à travers
    l'assistant posé par `gh auth setup-git`. Un jeton qui ne lit pas l'un des deux dépôts fait
    échouer l'ajout de la marketplace, l'installation et la mise à jour des plugins, en
    « Repository not found ».
  - **Pas de jeton à granularité fine** (*fine-grained personal access token*) : GitHub n'en
    rattache chacun qu'à **un seul propriétaire** — votre organisation **ou** `PixevBoard` —, il ne
    peut donc pas lire à la fois le dépôt de votre marketplace et
    `PixevBoard/board-marketplace-shared`.
  - **Attention** : le jeton de `gh auth token` porte **les droits d'écriture du compte** sur tous
    ses dépôts, et chaque session Claude Code, ses serveurs MCP et les commandes qu'elle lance le
    reçoivent.
  - **Pour réduire ces droits** : un **compte GitHub dédié à la lecture**, membre en **lecture
    seule** des deux dépôts — côté `PixevBoard`, par l'équipe GitHub ouverte à votre organisation.
    Exportez le jeton de ce compte : un jeton classique (*Personal access tokens (classic)*) de
    portée `repo`, ou celui que rend `gh auth token` connecté à ce compte. Ses droits effectifs se
    limitent à ce que ce compte peut lire.

- **SSO SAML** : si votre organisation l'impose, le jeton doit être **autorisé pour
  l'organisation** (GitHub, *Settings* → *Applications* ou *Tokens* → *Configure SSO*), sinon
  GitHub le traite comme un jeton sans accès.
- **Accès manquant** : GitHub ne répond pas « accès refusé » mais **« Repository not found »**.
  Ce message signifie presque toujours que le jeton ne lit pas le dépôt — pas qu'il n'existe pas.

## Ajouter la marketplace et installer les plugins

Toujours par l'**URL HTTPS complète** du dépôt, jamais par la forme courte `organisation/dépôt`
(résolue en SSH, que `gh auth setup-git` ne configure pas) :

```bash
claude plugin marketplace add https://github.com/<organisation>/<dépôt>.git
claude plugin install pixev-board@pixevboard
claude plugin install pixev-team@pixevboard
claude plugin install exemple-team@pixevboard     # le nom de votre plugin, une fois renommé
claude plugin list
```

## Variables du board

Le plugin `pixev-board` parle au Pixev Board avec deux variables, **à poser sur chaque poste** :

| Variable | Rôle |
| --- | --- |
| `PIXEV_BOARD_URL` | l'adresse de votre board, communiquée à la mise en service |
| `PIXEV_BOARD_API_KEY` | la clé d'API **personnelle** de la personne, générée sur le board ; Claude Code agit sur le board en son nom |

Où les poser : **exportées dans le profil shell** (`~/.bashrc`, `~/.zshrc`) — le serveur du board
les lit dans l'environnement de Claude Code. Le fichier `~/.config/pixev/pixev.env` peut les
porter aussi pour les scripts du plugin ; l'environnement gagne toujours. Le README du plugin
`pixev-board` détaille les variables facultatives. Aucune valeur ne s'écrit dans ce dépôt : la clé
d'API est personnelle et secrète.

## Monter de version

Une nouvelle version publiée ne s'installe **jamais toute seule** : c'est vous qui la choisissez.

1. Dans `.claude-plugin/marketplace.json`, changer le `ref` des **deux** entrées Pixev vers la
   nouvelle version (par exemple de `v1.4.0` à `v1.5.0`), puis pousser.
2. Sur chaque poste :

   ```bash
   claude plugin marketplace update pixevboard
   claude plugin update pixev-board@pixevboard
   claude plugin update pixev-team@pixevboard
   ```

3. Redémarrer la session Claude Code. `claude plugin list` affiche la version de chaque plugin
   telle que ce plugin la déclare à la version choisie : elle peut différer du numéro de version
   publié, et d'un plugin à l'autre.

## Adapter un rôle de l'équipe : fichiers de conventions

Les agents de `pixev-team` lisent, dans le **dépôt de travail** (celui du projet, pas celui-ci), un
fichier de conventions propre à leur rôle :

```
.claude/pixev/<role>.md
```

- `<role>` est **exactement** le nom de l'agent : `expert-qa`, `expert-securite`,
  `business-analyst`, `cto`… Un fichier au nom d'un autre rôle, ou d'un rôle qui n'existe pas
  (`qa.md`), n'est lu par personne.
- Un seul fichier par rôle, sans inclusion ni sous-dossier.
- Le fichier **complète** les règles du rôle : une forme de rapport, un contrôle en plus, votre
  vocabulaire, les zones sensibles de votre produit.
- Il **ne lève jamais** une porte — recette QA, audit de sécurité, plan, merge —, ni un verdict
  obligatoire, ni une règle de sécurité. Une consigne qui le tenterait est écartée, le reste du
  fichier s'applique, et l'agent le signale dans son rapport.
- Sans fichier, ou avec un fichier vide, l'agent se comporte exactement comme sans lui.

Ce mécanisme est livré par le plugin `pixev-team` (ticket AGY-102) ; il ne demande rien dans cette
marketplace.
