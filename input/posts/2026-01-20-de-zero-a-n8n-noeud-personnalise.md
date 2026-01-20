---
title: "De zéro à n8n: créer son premier node personnalisé"
Published: 2026-01-20
categories: post-fr
tags: [n8n,nodejs,oss,tutorial,demo,low-code]
---

# De zéro à n8n : créer son premier nœud personnalisé

Récemment, j'ai décidé de créer un cutom node pour [n8n](https://n8n.io/), l'outil d'automatisation de flux de travail que j'utilise. Je ne suis pas un expert en développement Node.js, mais je voulais comprendre comment les node n8n fonctionnent sous le capot. Cet article partage mon parcours et les étapes qui ont réellement fonctionné pour moi.

![n8n node build successful](../content/images/2026/01/n8n-node build0successful_600.png)

## Pourquoi j'ai fait ça

Avant de commencer ce projet, j'étais curieux de savoir comment les node n8n sont construits. La meilleure façon d'apprendre quelque chose est de le faire, alors j'ai décidé de créer un cutom node simple en suivant le tutoriel officiel de n8n. Maintenant que je comprends les bases, je prévois construire un node plus complexe avec des capacités d'IA Vision, mais c'est pour un autre billet sur ce blogue!

## Le défi

J'ai commencé avec le tutoriel officiel de n8n: [Build a declarative-style node](https://docs.n8n.io/integrations/creating-nodes/build/declarative-style-node/). Bien que le tutoriel soit bien écrit, j'ai rencontré quelques problèmes en cours de route. Les étapes ne fonctionnaient pas exactement comme décrit, alors j'ai dû comprendre ce qui manquait. Cet article documente ce qui a réellement fonctionné pour moi, au cas où vous feriez face à des défis similaires. J'ai déjà une instance de n8n qui roule dans un conteneur. À l'**étape 8**, j'expliquerai comment j'exécute une deuxième instance pour des fins de développement.

## Prérequis

Avant de commencer, vous aurez besoin de :

- **Node.js et npm** - J'ai utilisé Node.js version 24.12.0
- **Compréhension de base de JavaScript/TypeScript** - vous n'avez pas besoin d'être un expert

## Étape 1 : Corriger les prérequis manquants

Je n'avais pas Node.js installé sur ma machine, donc ma première étape a été de régler ça. Au lieu d'installer Node.js directement, j'ai utilisé **nvm** (Node Version Manager), qui facilite la gestion de différentes versions de Node.js. Les détails d'installation sont disponibles sur le [dépôt GitHub de nvm](https://github.com/nvm-sh/nvm). Une fois nvm configuré, j'ai installé Node.js version 24.12.0.

La plupart du temps, j'utilise VS Code comme éditeur de code. J'ai créé un nouveau profil et utilisé le modèle pour le développement Node.js afin d'obtenir les bonnes extensions et paramètres.

## Étape 2 : Cloner le dépôt de démarrage

n8n fournit un [n8n-nodes-starter sur GitHub](https://github.com/n8n-io/n8n-nodes-starter) qui inclut tous les fichiers de base et les dépendances dont vous avez besoin. Vous pouvez le cloner ou l'utiliser comme modèle pour votre propre projet. Puisque c'était seleument un « exercice d'apprentissage » pour moi, j'ai cloné le dépôt directement :

```bash
git clone https://github.com/n8n-io/n8n-nodes-starter
cd n8n-nodes-starter
```

## Étape 3 : Commencer avec le tutoriel

Je ne répéterai pas le tutoriel complet ici; il est assez clair, mais je soulignerai quelques détails en cours de route que j'ai trouvés utiles.

Le tutoriel vous fait créer un node « NasaPics » et fournit un logo pour celui-ci. C'est super, mais je suggère d'utiliser vos propres images de logo et d'avoir des versions claires et sombres. Ajoutez les deux images dans un nouveau dossier `icons` (au même niveau que les dossiers `nodes` et `credentials`). Avoir deux versions du logo rendra votre node plus beau, peu importe le thème que l'utilisateur utilise dans n8n (clair ou sombre). Le tutoriel ajoute seulement le logo dans `NasaPics.node.ts`, mais j'ai trouvé que l'ajouter aussi dans le fichier de credentials `NasaPicsApi.credentials.ts` rend le node plus cohérent.

Remplacez ou ajoutez la ligne du logo avec ceci, et ajoutez `Icon` à la déclaration d'importation en haut du fichier :

```typescript
icon: Icon = { light: 'file:MyLogo-dark.svg', dark: 'file:MyLogo-light.svg' };
```

> Note : le logo plus foncé devrait être utilisé en mode clair, et vice versa.

## Étape 4 : Suivre le tutoriel (avec ajustements)

C'est là que les choses sont devenues intéressantes. J'ai suivi le tutoriel officiel pour créer les fichiers de node, mais j'ai dû faire quelques ajustements qui n'étaient pas mentionnés dans la documentation.

### Ajustement 1 : Rendre le node utilisable comme outil

Dans le fichier `NasaPics.node.ts`, j'ai ajouté cette ligne juste avant le tableau `properties` :

```typescript
requestDefaults: {
      baseURL: 'https://api.nasa.gov',
      headers: {
         Accept: 'application/json',
         'Content-Type': 'application/json',
      },
   },
   usableAsTool: true, // <-- Ajouté cette ligne
   properties: [
      // Resources and operations will go here
```

Ce paramètre permet au node d'être utilisé comme outil dans les flux de travail n8n et corrige aussi les avertissements de l'outil de lint.

### Ajustement 2 : Sécuriser le champ de clé API

Dans le fichier `NasaPicsApi.credentials.ts`, j'ai ajouté un `typeOptions` pour faire du champ de clé API un champ de mot de passe. Cela garantit que la clé API est cachée lorsque les utilisateurs la saisissent, ce qui est une bonne pratique de sécurité.

```typescript
properties: INodeProperties[] = [
   {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true }, // <-- Ajouté cette ligne
      default: '',
   },
];
```

### Une note sur les erreurs

J'ai remarqué qu'il y avait quelques autres erreurs qui apparaissaient dans le fichier de credentials. Si vous lisez le message d'erreur, vous verrez qu'il se plaint de propriétés `test` manquantes. Pour corriger cela, j'ai ajouté une propriété `test` à la fin de la classe qui implémente `ICredentialTestRequest`. J'ai aussi ajouté l'importation de l'interface en haut du fichier.

```typescript
authenticate: IAuthenticateGeneric = {
   type: 'generic',
   properties: {
      qs: {
         api_key: '={{$credentials.apiKey}}',
      },
   },
};

// Ajoutez ceci à la fin de la classe
test: ICredentialTestRequest = {
   request: {
      baseURL: 'https://api.nasa.gov/',
      url: '/user',
      method: 'GET',
   },
};
```

## Étape 5 : Construire et lier le paquet

Une fois que j'avais tous mes fichiers prêts, il était temps de construire le node. Depuis la racine du dossier de mon projet de node, j'ai exécuté :

```bash
npm i
npm run build
npm link
```

Pendant le processus de construction, faites attention au nom du paquet qui est généré. Dans mon cas, c'était `n8n-nodes-nasapics`. Vous aurez besoin de ce nom dans les prochaines étapes.

```bash
> n8n-nodes-nasapics@0.1.0 build
> n8n-node build

┌   n8n-node build
│
◓  Building TypeScript files│
◇  TypeScript build successful
│
◇  Copied static files
│
└  ✓ Build successful
```

## Étape 6 : Configurer le dossier personnalisé de n8n

n8n cherche les node personnalisés dans un emplacement spécifique : `~/.n8n/custom/`. Si ce dossier n'existe pas, vous devez le créer :

```bash
mkdir -p ~/.n8n/custom
cd ~/.n8n/custom
```

Ensuite, initialisez un nouveau paquet npm dans ce dossier : exécutez `npm init` et appuyez sur Entrée pour accepter tous les défauts.

## Étape 7 : Lier votre node à n8n

Maintenant vient la partie magique - lier votre cutom node pour que n8n puisse le trouver. Remplacez `n8n-nodes-nasapics` par le nom de votre paquet. Depuis le dossier `~/.n8n/custom`, exécutez :

```bash
npm link n8n-nodes-nasapics
```

## Étape 8 : Exécuter n8n

C'est ici que ma configuration diffère du tutoriel standard. Comme mentionné au début, j'ai déjà une instance de n8n qui roule dans un conteneur et je ne voulais pas l'installer. Alors j'ai décidé d'exécuter un deuxième conteneur en utilisant un port différent. Voici la commande que j'ai utilisée :

```bash
docker run -d --name n8n-DEV -p 5680:5678 \
  -e N8N_COMMUNITY_PACKAGES_ENABLED=true \
  -v ~/.n8n/custom/node_modules/n8n-nodes-nasapics:/home/node/.n8n/custom/node_modules/n8n-nodes-nasapics \
  n8nio/n8n
```

Laissez-moi expliquer ce que cette commande fait :

- `-d` : Exécute le conteneur en mode détaché (en arrière-plan)
- `--name n8n-DEV` : Nomme le conteneur pour une référence facile
- `-p 5680:5678` : Mappe le port 5678 du conteneur au port 5680 sur ma machine pour qu'il n'entre pas en conflit avec mon instance existante de n8n
- `-e N8N_COMMUNITY_PACKAGES_ENABLED=true` : Active les paquets communautaires — vous en avez besoin pour utiliser des node personnalisés
- `-v` : Monte mon dossier de cutom node dans le conteneur, ce qui me permet d'essayer mon cutom node sans avoir à le publier.
- `n8nio/n8n` : L'image de conteneur officielle de n8n

Si vous exécutez n8n directement sur votre machine (pas dans un conteneur), vous pouvez simplement le démarrer.

## Étape 9 : Tester votre node

Une fois que **n8n-DEV** roule, ouvrez votre navigateur et naviguez vers celui-ci. Créez un nouveau flux de travail et recherchez votre node. Dans mon cas, j'ai cherché « NasaPics » et mon cutom node est apparu!

Pour le tester :

1. Ajoutez votre node au flux de travail
2. Configurez les credentials avec une clé API de la NASA (vous pouvez en obtenir une gratuitement sur [api.nasa.gov](https://api.nasa.gov/))
3. Exécutez le node
4. Vérifiez si les données sont récupérées correctement

## Mettre à jour votre node

Pendant le développement, vous devrez probablement apporter des modifications à votre code (aka node). Une fois terminé, vous devez reconstruire `npm run build` et redémarrer le conteneur n8n `docker restart n8n-DEV` pour voir les changements.

## Quelle est la suite?

Maintenant que je comprends les bases de la construction de node personnalisés n8n, je suis prêt à m'attaquer à quelque chose de plus ambitieux. Mon prochain projet sera de créer un node qui utilise des capacités d'IA Vision. Alerte au spoiler : C'est fait et je partagerai les détails dans un prochain article de blogue!

Si vous êtes intéressé à créer vos propres node personnalisés, je vous encourage à essayer. Commencez avec quelque chose de simple, comme je l'ai fait, et construisez à partir de là. N'ayez pas peur d'expérimenter et de faire des erreurs, c'est comme ça qu'on apprend!

#### Ressources

- [Documentation officielle de n8n](https://docs.n8n.io/)
- [Tutoriel Build a declarative-style node](https://docs.n8n.io/integrations/creating-nodes/build/declarative-style-node/)
- [Dépôt n8n-nodes-starter](https://github.com/n8n-io/n8n-nodes-starter)
- [nvm (Node Version Manager)](https://github.com/nvm-sh/nvm)
- [Forum communautaire de n8n](https://community.n8n.io/)
