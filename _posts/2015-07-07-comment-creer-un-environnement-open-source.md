---
layout: post
title:  "Comment créer un environnement open source"
date:   2015-07-07
comments: true
translation: true
banner_image: posts/how-to-create-an-opensource-stack/amalfi-coast-by-night.jpg

translated: true
translation_url: 2015/07/07/how-to-create-an-opensource-stack
translation_language: english
---

Essayons de mettre en place un environnement open source avec [Github](https://github.com), [TravisCI](https://travis-ci.org) et [CodeClimate](https://codeclimate.com).

D'abord, on va commencer par créer le repository GitHub d'un projet tout simple. Pour cet exemple, je vais créer un module npm nommé `mooh` qui se contentera de dessiner une vache dans les logs de la console. Bref du vrai code de pro, vous vous en doutez bien...

## Création du projet GitHub

* Commencez par créer le repository sur GitHub
* Clonez le repository sur votre environnement local :

{% highlight bash %}
$ cd /var/www
$ git clone git@github.com:lgiraudel/mooh.git
$ cd mooh
{% endhighlight %}

* Et maintenant ajoutons un peu de code dedans

{% highlight javascript %}
// index.js

function mooh() {
  console.log('')
  console.log('             Mooooh!');
  console.log('')
  console.log('         (__)');
  console.log('         (oo)');
  console.log('  /-------\\/');
  console.log(' / |     ||');
  console.log('*  ||----||');
  console.log('   ^^    ^^');
  console.log('');
}

module.exports = {
  mooh: mooh
}
{% endhighlight %}

## Saupoudrez de quelques tests

Avant qu'on se plug sur Travis, on va rajouter quelques tests sur notre module.

On va utiliser le combo [Mocha](http://mochajs.org/) + [Chai](http://chaijs.com/) + [Sinon](http://sinonjs.org/). Mocha est un framework de tests, Chai est une bibliothèque d'assertion et Sinon est une bibliothèque pour mocker / stubber des objets.

Commençons par ajouter un fichier `package.json` avec la commande `npm init` :

{% highlight bash %}
$ npm init -f
$ npm install --save-dev mocha chai sinon
{% endhighlight %}

Dans le fichier `package.json` généré, vous devriez voir une propriété `scripts` :

{% highlight json %}
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
}
{% endhighlight %}

La commande associée à la propriété `test` est exécutée lorsqu'on lance la commande `npm test` :

{% highlight bash %}
$ npm test

> mooh@1.0.0 test /var/www/mooh
> echo "Error: no test specified" && exit 1

Error: no test specified
npm ERR! Test failed.  See above for more details.
{% endhighlight %}

Vu que nous allons utiliser mocha pour lancer nos tests, nous pouvons remplacer le message d'erreur par l'appel à l'exécutable de mocha :
{% highlight json %}
"scripts": {
  "test": "node_modules/.bin/mocha"
}
{% endhighlight %}

Ou plus simplement :
{% highlight json %}
"scripts": {
  "test": "mocha"
}
{% endhighlight %}

Maintenant, si nous relançons la commande `npm test` :
{% highlight bash %}
$ npm test

> mooh@1.0.0 test /var/www/mooh
> mocha

/var/www/mooh/node_modules/mocha/lib/utils.js:591
      if (!files.length) throw new Error("cannot resolve path (or pattern) '"
                               ^
Error: cannot resolve path (or pattern) 'test'
    at Object.lookupFiles (/var/www/mooh/node_modules/mocha/lib/utils.js:591:32)
    at /var/www/mooh/node_modules/mocha/bin/_mocha:320:30
    at Array.forEach (native)
    at Object.<anonymous> (/var/www/mooh/node_modules/mocha/bin/_mocha:319:6)
    at Module._compile (module.js:460:26)
    at Object.Module._extensions..js (module.js:478:10)
    at Module.load (module.js:355:32)
    at Function.Module._load (module.js:310:12)
    at Function.Module.runMain (module.js:501:10)
    at startup (node.js:129:16)
    at node.js:814:3
npm ERR! Test failed.  See above for more details.
{% endhighlight %}

Ok, visiblement mocha cherche un répertoire `test` par défaut, aidons le un peu en le créant et ressayons :

{% highlight bash %}
$ mkdir test
$ npm test

> mooh@1.0.0 test /var/www/mooh
> mocha

  0 passing (11ms)
{% endhighlight %}

Cool, on est prêt à écrire notre premier test ! Créons un fichier `mooh.spec.js` :
{% highlight javascript %}
// test/mooh.spec.js

var mooh   = require('../index');
var sinon  = require('sinon');
var assert = require('chai').assert;

describe('Mooh tests', function() {
  beforeEach(function() {
    sinon.spy(console, 'log');
  });

  afterEach(function() {
    console.log.restore();
  });

  it('should say Mooooh!', function() {
    mooh.mooh();

    assert.equal(console.log.callCount, 10);
    sinon.assert.calledWithMatch(console.log, 'Mooooh!');
  });
});
{% endhighlight %}

Est-ce que notre test fonctionne bien ?

{% highlight bash %}
$ npm test

> mooh@1.0.0 test /var/www/mooh
> mocha



  Mooh tests

             Mooooh!

         (__)
         (oo)
  /-------\/
 / |     ||
*  ||----||
   ^^    ^^

    ✓ should say Mooooh!


  1 passing (54ms)
{% endhighlight %}

Super ! Maintenant qu'on a nos tests, commitons tout ça sur le repo :
{% highlight bash %}
$ git add package.json test index.js
$ git commit -m 'First commit with code and tests!'
$ git push origin master
{% endhighlight %}

Et maintenant, allons brancher Travis !

## Configuration de Travis

Allez sur [Travis](https://travis-ci.org/) et connectez vous avec votre compte GitHub, puis activez Travis sur le projet `mooh`.

![Activation de Travis](/assets/images/posts/how-to-create-an-opensource-stack/travis-activation.png)

C'est tout ? Presque, il faut juste ajouter un fichier `.travis.yml` dans notre application :
{% highlight yaml %}
# .travis.yml

language: node_js
node_js:
  - "0.12"
  - "0.11"
  - "0.10"
  - "0.8"
  - "0.6"
  - "iojs"
  - "iojs-v1.0.4"
{% endhighlight %}

Comme vous le voyez, dans ce fichier yaml nous spécifions que notre projet est un projet NodeJS et nous définissions toutes les versions de Node (ou d'IO) pour lesquelles nous voulons que Travis build notre projet.

Commitons ce nouveau fichier :
{% highlight bash %}
$ git add .travis.yml
$ git commit -m 'Add of Travis configuration file'
$ git push origin master
{% endhighlight %}

Si vous retournez sur Travis et que vous jetez un oeil au projet `mooh`, vous verrez que Travis est en train de le builder sur toutes les versions de Node et de IO spécifiées.

![Premier buid dans Travis](/assets/images/posts/how-to-create-an-opensource-stack/first-build.png)

![Erreur de build dans Travis](/assets/images/posts/how-to-create-an-opensource-stack/sinon_error.png)

Visiblement Node 0.6 et 0.8 ont qulques soucis avec Mocha, Chai et Sinon. Supprimons ces versions du fichier `.travis.yml`. Retirons aussi les versions de IO pour que le build soit plus rapide.
{% highlight yaml %}
# .travis.yml

language: node_js
node_js:
  - "0.12"
  - "0.11"
  - "0.10"
{% endhighlight %}

On re-commit, Travis devrait être vert maintenant :)

![Build vert dans Travis](/assets/images/posts/how-to-create-an-opensource-stack/successful_build.png)

Ok, c'est tout pour Travis.

## La puissance des Pull Requests

L'un des plus gros atouts de GitHub est son système de Pull Requests. Pour l'illustrer, nous allons utiliser un nouveau compteur utilisateur qui représentera n'importe qui intéressé par votre projet.

Créons un nouveau compte GitHub et forkons le projet `Mooh`.

![Fork du projet](/assets/images/posts/how-to-create-an-opensource-stack/fork.png)

Clonons le projet forké :
{% highlight bash %}
$ cd /var/www
$ git clone https://github.com/wilsonbuntu/mooh.git mooh-forked
$ cd mooh-forked
{% endhighlight %}

Ok, maintenant modifions le code en... hmmm... permettant à l'utilisateur de passer une chaîne de caractères à la méthode `mooh()`.
{% highlight bash %}
$ git diff
diff --git index.js index.js
index f1635be..6ba7c18 100644
--- index.js
+++ index.js
@@ -1,6 +1,6 @@
-function mooh() {
+function mooh(msg) {
   console.log('')
-  console.log('             Mooooh!');
+  console.log('             ' + msg);
   console.log('')
   console.log('         (__)');
   console.log('         (oo)');
{% endhighlight %}

On commit et on push...
{% highlight bash %}
$ git add index.js
$ git commit -m 'Allow to pass a string to the mooh() method'
$ git push origin master
{% endhighlight %}

A ce stade, les modifications ne se trouvent que sur le projet forké (appartenant au second utilisateur). Pour proposer ces modifications au projet originel, nous devons créer une Pull Request sur le repository d'origine.

Allez sur [https://github.com/lgiraudel/mooh/compare/master...lgiraudel-purch:master](https://github.com/lgiraudel/mooh/compare/master...lgiraudel-purch:master) et créer une Pull Request.

![Création d'une pull request](/assets/images/posts/how-to-create-an-opensource-stack/pull_request_creation.png)

Si vous attendez quelques secondes et que vous rafraîchissez la Pull Request, vous remarquerz qu'elle est mise à jour par Travis avec plusieurs informations nous indiquant que les tests sont en cours d'exécution (avec un lien pour aller voir les détails sur Travis).

![Pull request en cours d'exécution](/assets/images/posts/how-to-create-an-opensource-stack/pr_running.png)

Au bout d'un moment, nous apprenons que quelque chose s'est mal passé.

![Pull request en erreur](/assets/images/posts/how-to-create-an-opensource-stack/pr_failed_tests.png)

Si nous ouvrons les détails de l'un des 3 builds dans Travis, nous pouvons constater qu'effectivement notre test a planté. Mais bien sûr ! Nous n'avons pas pensé à mettre à jour notre test pour prendre en compte les changements dans le code !

![Echec sur Travis](/assets/images/posts/how-to-create-an-opensource-stack/travis_fail.png)

Si nous ouvrons la Pull Request avec le propriétaire du repository d'origine, vous constaterez que celui-ci a la possibilité de merger la PR : Travis ne fait que rajouter des informations, il ne bloque pas la possibilité de merger.

![Le propriétaire du repo peut toujours merger](/assets/images/posts/how-to-create-an-opensource-stack/owner_can_merge.png)

Mettons à jour notre test pour le faire passer au vert.
{% highlight bash %}
$ git diff
diff --git test/mooh.spec.js test/mooh.spec.js
index ad700a7..ca62ac1 100644
--- test/mooh.spec.js
+++ test/mooh.spec.js
@@ -12,7 +12,7 @@ describe('Mooh tests', function() {
   });

   it('should say Mooooh!', function() {
-    mooh.mooh();
+    mooh.mooh('Mooooh!');

     assert.equal(console.log.callCount, 10);
     sinon.assert.calledWithMatch(console.log, 'Mooooh!');
$ git add test/mooh.spec.js
$ git commit -m 'Fixing test'
$ git push origin master
{% endhighlight %}

Il n'y a strictement rien à faire sur GitHub : lorsqu'on met à jour le repoitory de fork, la Pull Request existante se met automatiquement à jour sur le repository d'origine. Du coup Travis se lance tout seul pour faire tourner les tests sur la PR modifiée et la remet à jour pour indiquer que les tests sont en cours de run.

Une fois que Travis a finit son boulot, la PR est considéré comme verte et peut être mergée en toute confiance.

![Succés sur Travis](/assets/images/posts/how-to-create-an-opensource-stack/travis_success.png)

![La pull request est verte](/assets/images/posts/how-to-create-an-opensource-stack/pr_green.png)

Travis inspecte les PR mais également les autres branches. Quand une PR est mergée dans une branche (par exemple `master`), les tests sont de nouveau exécutés sur la branche modifiée.

## Analyse de qualité de code avec CodeClimate

Enfin, nous pouvons mettre en place des outils d'analyse de code avec CodeClimate.

Allez sur [CodeClimate](http://codeclimate.com) et connectez-vous avec votre compte GitHub puis cliquez sur "Add Open Source repo" et ajoutez votre repository (le repository d'origine, on ne parle plus du fork là).

CodeClimate va faire des tests sur notre code mais si nous voulons une couverture de tests, nous devons lui envoyer nous-mêmes les données. Mais avant ça, nous devons les générer.

Nous pouvons facilement générer des rapports de couverture de tests sur du code Javascript avec le plugin Istanbul. Installons-le pour commencer :
{% highlight bash %}
$ npm install --save-dev istanbul
{% endhighlight %}

Nous pouvons lancer Istanbul avec la commande suivante : `./node_modules/.bin/istanbul cover ./node_modules/.bin/_mocha`. Istanbul va exécuter les tests et générer un rapport :
{% highlight bash %}
$ ./node_modules/.bin/istanbul cover ./node_modules/.bin/_mocha


  Mooh tests

             Mooooh!

         (__)
         (oo)
  /-------\/
 / |     ||
*  ||----||
   ^^    ^^

    ✓ should say Mooooh!


  1 passing (10ms)

=============================================================================
Writing coverage object [/var/www/mooh/coverage/coverage.json]
Writing coverage reports at [/var/www/mooh/coverage]
=============================================================================

=============================== Coverage summary ===============================
Statements   : 100% ( 12/12 )
Branches     : 100% ( 0/0 )
Functions    : 100% ( 1/1 )
Lines        : 100% ( 12/12 )
================================================================================
{% endhighlight %}

Du coup, nous pouvons remplacer l'appel à mocha dans le fichier `package.json` par l'appel à Istanbul :
{% highlight json %}
  "scripts": {
    "test": "istanbul cover _mocha"
  },
{% endhighlight %}

Pour envoyer nos données à CodeClimate, nous devons installer le module `codeclimate-test-reporter` :
{% highlight bash %}
$ npm install --save-dev codeclimate-test-reporter
{% endhighlight %}

Pour finir, nous devons dire à Travis d'envoyer le rapport de couverture (au format lcov) en utilisant codeclimate-test-reporter après l'exécution des tests. Et nous devons ajouter notre token CodeClimate pour que l'envoi des données fonctionne.
{% highlight yaml %}
language: node_js
node_js:
  - "0.12"
  - "0.11"
  - "0.10"

addons:
  code_climate:
    repo_token: 8e9f83479b416f682142c375e206de906b687ff125cc85f3a19874421845aee6

after_script:
  - codeclimate-test-reporter < coverage/lcvov.info
{% endhighlight %}

Commitons et pushons tout ça :
{% highlight bash %}
$ git add .travis.yml package.json
$ git commit -m 'Adding coverage analysis'
$ git push origin master
{% endhighlight %}

Travis va détecter notre commit, lancer les tests et envoyer le résultat à CodeClimate.

Dans CodeClimate, vous devriez pouvoir voir la couverture de tests ici : [https://codeclimate.com/github/lgiraudel/mooh/coverage](https://codeclimate.com/github/lgiraudel/mooh/coverage)

![Couverture de tests dans CodeClimate](/assets/images/posts/how-to-create-an-opensource-stack/codeclimate_coverage.png)

Vous pouvez également voir un rapport de codestyle ici : [https://codeclimate.com/github/lgiraudel/mooh/index.js](https://codeclimate.com/github/lgiraudel/mooh/index.js)

![Code style dans CodeClimate](/assets/images/posts/how-to-create-an-opensource-stack/codeclimate_codestyle.png)

## Ajout d'informations dynamiques dans le repo GitHub

Ca serait super si on pouvait ajouter dans le repository Github l'était du dernier build (rouge ou vert) et la couverture de tests. Ca tombe bien, parce que c'est super facile à mettre en place !

Dans Travis, cliquez sur l'étiquette "build unknown" (ou "build passing"), cela va ouvrir une popup. Choisissez la syntaxe Markdown et copiez le code généré dans le fichier Readme du repository (vous pouvez l'éditer en ligne).

![Création de tag dans Travis](/assets/images/posts/how-to-create-an-opensource-stack/travis_tag.png)

Vous pouvez faire la même chose dans CodeClimate en cliquant sur l'étiquette "coverage 100%". Dans la page où vous arrivez, choisissez la syntaxe Markdown et copiez le code généré dans le fichier Readme.

![Creation de tag dans CodeClimate](/assets/images/posts/how-to-create-an-opensource-stack/codeclimate_tag.png)

![Edition du fichier Readme.md](/assets/images/posts/how-to-create-an-opensource-stack/readme_edit.png)

![Affichage des Tags](/assets/images/posts/how-to-create-an-opensource-stack/readme.png)

Et voilà, il ne vous reste plus qu'à trouver plein de contributeurs pour votre super projet =)