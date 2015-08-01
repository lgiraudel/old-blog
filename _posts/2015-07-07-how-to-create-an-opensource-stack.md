---
layout: post
title:  "How to create an opensource stack"
date:   2015-07-07
comments: true

translated: true
translation_url: 2015/07/07/comment-creer-un-environnement-open-source
translation_language: french
---

Let's try to create an opensource stack with [Github](https://github.com), [TravisCI](https://travis-ci.org) and [CodeClimate](https://codeclimate.com).

First, let's create a github repository with a simple project. For this example, I will create a npm module called `mooh` which only draws a cow on the console log.

## Sample project creation

* Create the repository on github
* Clone the repository

{% highlight bash %}
$ cd /var/www
$ git clone git@github.com:lgiraudel/mooh.git
$ cd mooh
{% endhighlight %}

* And now let's add some code in it

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

## Pepper with some tests

Before pluging Travis, we need to add some tests in our application.

We will use the combo [Mocha](http://mochajs.org/) + [Chai](http://chaijs.com/) + [Sinon](http://sinonjs.org/). Mocha is a test framework, Chai is an assert library and Sinon is a Mocking / Stubbing library.

Let's add a `package.json` file with `npm init` command:

{% highlight bash %}
$ npm init -f
$ npm install --save-dev mocha chai sinon
{% endhighlight %}

In the generated `package.json` file, there is a `scripts` property:

{% highlight json %}
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
}
{% endhighlight %}

The command associated to the `test` property is launched when we use the `npm test` command:

{% highlight bash %}
$ npm test

> mooh@1.0.0 test /var/www/mooh
> echo "Error: no test specified" && exit 1

Error: no test specified
npm ERR! Test failed.  See above for more details.
{% endhighlight %}

We use mocha to launch our tests so we can replace the error message by a call to mocha tool:
{% highlight json %}
"scripts": {
  "test": "node_modules/.bin/mocha"
}
{% endhighlight %}

Or simply:
{% highlight json %}
"scripts": {
  "test": "mocha"
}
{% endhighlight %}

Now, when we launch `npm test` command:
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

Ok, it seems that mocha search a `test` directory by default so let's help him and try again:

{% highlight bash %}
$ mkdir test
$ npm test

> mooh@1.0.0 test /var/www/mooh
> mocha

  0 passing (11ms)
{% endhighlight %}

Cool we are ready to write a test! Let's create `mooh.spec.js` file:
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

Is the test working ?

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

Yeah! We're done with tests, commit all this stuff:
{% highlight bash %}
$ git add package.json test index.js
$ git commit -m 'First commit with code and tests!'
$ git push origin master
{% endhighlight %}

And now, let's plug Travis!

## Travis configuration

Go to [Travis](https://travis-ci.org/) and log in with your github account then activate Travis for the mooh project.

![Travis activation](/assets/images/posts/how-to-create-an-opensource-stack/travis-activation.png)

That's all ? Almost, we just need to add a `.travis.yml` file in our application:
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

As you can see, in this yaml file we specify that our project is a Node project and we can define all Node (or IO) version to build the project with.

Commit this file:
{% highlight bash %}
$ git add .travis.yml
$ git commit -m 'Add of Travis configuration file'
$ git push origin master
{% endhighlight %}

If you go back to Travis and check the status of your mooh project, you will see that Travis is building your project with all specified versions of Node & IO.

![First build in Travis](/assets/images/posts/how-to-create-an-opensource-stack/first-build.png)

![Build error in Travis](/assets/images/posts/how-to-create-an-opensource-stack/sinon_error.png)

Node 0.6 & 0.8 versions seems to have some troubles with Mocha, Chai and Sinon, let's remove those versions in the `.travis.yml` file. Let's also remove IO versions in order to have a faster build.
{% highlight yaml %}
# .travis.yml

language: node_js
node_js:
  - "0.12"
  - "0.11"
  - "0.10"
{% endhighlight %}

Let's commit again, Travis should be green now :)

![Green build in Travis](/assets/images/posts/how-to-create-an-opensource-stack/successful_build.png)

Ok, that's it for Travis for now.

## Embrace the power of pull requests

The main strength of Github is the Pull Request mechanism. To illustrate it, we will use a new account who will represents any internet user interested by your project.

Create a new Github account and fork the `Mooh` project.

![Project fork](/assets/images/posts/how-to-create-an-opensource-stack/fork.png)

Clone the forked project:
{% highlight bash %}
$ cd /var/www
$ git clone https://github.com/wilsonbuntu/mooh.git mooh-forked
$ cd mooh-forked
{% endhighlight %}

Ok, now let's update the code by... hmmm... allowing user to pass a string to the `mooh()` method.
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

Let's commit and push it.
{% highlight bash %}
$ git add index.js
$ git commit -m 'Allow to pass a string to the mooh() method'
$ git push origin master
{% endhighlight %}

At this point, the modifications are in the forked repository (owned by the second user). We have to create a Pull Request on the original repository in order to propose our code update.

Go to [https://github.com/lgiraudel/mooh/compare/master...lgiraudel-purch:master](https://github.com/lgiraudel/mooh/compare/master...lgiraudel-purch:master) and create a Pull Request.

![Pull request creation](/assets/images/posts/how-to-create-an-opensource-stack/pull_request_creation.png)

If you wait few secondes and then refresh the Pull Request, you'll notice that it's updated by Travis with some informations telling us that tests are currently running (with a link to see details in Travis).

![Pull request running](/assets/images/posts/how-to-create-an-opensource-stack/pr_running.png)

At the end, we learn that some checks have failed.

![Pull request failed](/assets/images/posts/how-to-create-an-opensource-stack/pr_failed_tests.png)

If we open the details of one of the 3 builds, we can see that test have failed. Of course, we forgot to update the tests!

![Travis failure](/assets/images/posts/how-to-create-an-opensource-stack/travis_fail.png)

If you open the PR with the original user, you'll see that you can still merge the PR: Travis just adds some informations, it doesn't block the merging process.

![Owner still can merge](/assets/images/posts/how-to-create-an-opensource-stack/owner_can_merge.png)

Let's update our tests to pass them green.
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

We don't need to do anything on Github: when we update our forked repository, the existing Pull Request is automatically updated in the original repository so Travis will execute tests again and update the PR accordingly.

Once Travis has finished his job, the PR is tagged as green and can be merged with confidence.

![Travis success](/assets/images/posts/how-to-create-an-opensource-stack/travis_success.png)

![Pull request is green](/assets/images/posts/how-to-create-an-opensource-stack/pr_green.png)

Travis tracks PR but other branches too. When a PR is merged into a branch (like master), the tests are run on the modified branch.

## Code quality with CodeClimate

Last but least, we can add some quality analysis tools like CodeClimate.

Go to [CodeClimate](http://codeclimate.com) and log in with your github account then click on the "Add Open Source repo" and add your the repository (the original one).

CodeClimate will do some checks for us but if we want test coverage analysis, we have to send him data. But before sending data, we have to generate it.

We can easily generate coverage report for javascript code with Istanbul plugin. Let's install it first:
{% highlight bash %}
$ npm install --save-dev istanbul
{% endhighlight %}

We can launch istanbul with the following command: `./node_modules/.bin/istanbul cover ./node_modules/.bin/_mocha`. Istanbul will execute the tests then generate some coverage report:
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

So we can replace the call to mocha in `package.json` by the call to istanbul:
{% highlight json %}
  "scripts": {
    "test": "istanbul cover _mocha"
  },
{% endhighlight %}

To send data to CodeClimate, we need to install the `codeclimate-test-reporter` module:
{% highlight bash %}
$ npm install --save-dev codeclimate-test-reporter
{% endhighlight %}

Finally, we have to tell Travis to send coverage report (lcov format) with the codeclimate-test-reporter after executing the tests. And we need to add our CodeClimate token.
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

Let's commit and push all this stuff:
{% highlight bash %}
$ git add .travis.yml package.json
$ git commit -m 'Adding coverage analysis'
$ git push origin master
{% endhighlight %}

Travis will detect a new commit, run the tests and send the result to CodeClimate.

In CodeClimate, you should be able to see the code coverage report here: [https://codeclimate.com/github/lgiraudel/mooh/coverage](https://codeclimate.com/github/lgiraudel/mooh/coverage)

![Code coverage in CodeClimate](/assets/images/posts/how-to-create-an-opensource-stack/codeclimate_coverage.png)

You also can see that we let some code style errors in the code here: [https://codeclimate.com/github/lgiraudel/mooh/index.js](https://codeclimate.com/github/lgiraudel/mooh/index.js)

![Code style in CodeClimate](/assets/images/posts/how-to-create-an-opensource-stack/codeclimate_codestyle.png)

## Add some dynamic info in your GitHub repository

It could be awesome to display in your Github repository the build state (red or green) and the coverage status and it's super easy !

In Travis, click on the "build unknown" (or "build passing") tag, it will open a popup. Choose markdown syntax and copy the generated text to past it in your Readme file of your repository (you can edit it online).

![Tag creation in Travis](/assets/images/posts/how-to-create-an-opensource-stack/travis_tag.png)

You can do the same thing with CodeClimate by clicking on the "coverage 100%" tag. In the new page, choose the markdown syntax and copy the generated text to past it in the Readme file.

![Tag creation in CodeClimate](/assets/images/posts/how-to-create-an-opensource-stack/codeclimate_tag.png)

![Readme.md edition](/assets/images/posts/how-to-create-an-opensource-stack/readme_edit.png)

![Tags display](/assets/images/posts/how-to-create-an-opensource-stack/readme.png)

That's it, now you just need to find many contributors to your project :)