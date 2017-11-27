# Tutorial: Continuous Integration for Serverless Applications

Continuous Integration is the practice of merging all working copies of
developer code into one shared mainline several times a day. Best practices
include automation of builds and deployments, with fast and self-testing
builds, as well as production-like testing environments. With serverless, the
continuous integration pipeline evolves from a one-lane, one-way street into a
multiple-lane, two-way highway.

In this tutorial, we take a simple serverless application and walk you through
the steps to set up unit testing, end-to-end testing, code coverage, code
analysis, code security, code performance, and peer code review. You can learn
how to use AWS serverless components in combination with GitHub, Travis-CI,
CodeClimate, Snyk and other serverless-friendly services.

## Getting Started

### Step 1: Your Serverless Application

> Step 1 Branch:
https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step1

For this tutorial, we went with Single Page Application from
[todomvc.com](http://todomvc.com), transformed by our team into
[TodoMVC Serverless App](https://github.com/MitocGroup/deep-microservices-todomvc).
Similar approach can be applied to serverless applications described in
[Serverless Single Page Apps](https://pragprog.com/book/brapps/serverless-single-page-apps)
or [Serverless Stack](https://serverless-stack.com),
as well as any other serverless applications from
[Curated List of Awesome Serverless](https://github.com/anaibol/awesome-serverless).

```shell
# download locally todomvc serverless application codebase
git clone git@github.com:MitocGroup/deep-microservices-todomvc.git

# download locally tutorial repository codebase
git clone git@github.com:MitocGroup/tutorial-ci-for-serverless.git

# copy serverless application into a new branch, part of tutorial repository
cd ./tutorial-ci-for-serverless
git checkout -b tutorial-step1
cp -R ../deep-microservices-todomvc/src ../deep-microservices-todomvc/bin .
git commit -a -m "initial serverless application"
git push --set-upstream origin tutorial-step1
```

### Step 2: Setup Travis CI

> Step 2 Branch:
https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step2

Travis CI takes care of running your tests and deploying your apps. Quoting
official [Getting Started](https://docs.travis-ci.com/user/getting-started/)
guide:

> To start using Travis CI, make sure you have all of the following:
> - GitHub login
> - Project hosted as a repository on GitHub
> - Working code in your project
> - Working build or test script

Here below is our initial `.travis.yml` file:

```yaml
language: node_js
dist: trusty
git:
  depth: 1
cache:
  bundler: true
  directories:
    - node_modules
    - "$(npm root -g)"
    - "$(npm config get prefix)/bin"
node_js:
  - 6
  - 8
script: echo "Hello World!"
```

Once we have updated `.travis.yml`, we can commit the code back to our
repository:

```shell
# assuming you're still inside the tutorial repository
git checkout -b tutorial-step2
git add .travis.yml
git commit -m "enable travis ci"
git push --set-upstream origin tutorial-step2
```

And there you go, our first successful build:
https://travis-ci.org/MitocGroup/tutorial-ci-for-serverless/builds/283669984

### Step 3: Setup Unit Testing

> Step 3 Branch:
https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step3

Travis CI inspired us with its simplicity and streamlined process. On the other
hand, we were surprised that triggering tests execution is not part of the
service. And we couldn't find any other tool that does it well, so we wrote
one: https://www.npmjs.com/package/recink

To get started with this tool, just install it on your machine and configure:

```shell
# install recink as global nodejs library
npm install -g recink

# configure recink in your repository
recink configure recink
```

Last command will dump an example of `recink.yml` (conceptually the same
approach as `.travis.yml`), which helps to simplify and streamline the
execution of unit tests, e2e tests and more:

```yaml
---
$:
  npm:
    scripts: []
    dependencies:
      chai: 'latest'
  emit:
    pattern:
      - /.+\.js$/i
    ignore:
      - /^(.*\/)?bin(\/?$)?/i
      - /^(.*\/)?node-bin(\/?$)?/i
      - /^(.*\/)?node_modules(\/?$)?/i
      - /^(.*\/)?vendor(\/?$)?/i
  test:
    mocha:
      options:
        ui: 'bdd'
        reporter: 'spec'
    pattern:
      - /.+\.spec\.js$/i
    ignore: ~

deep-todomvc:
  root: 'src/deep-todomvc'
```

In order to let Travis CI know that we have some tests to execute, we go
back to `.travis.yml` and change the script that will be executing from 
`script: echo "Hello World!"` to this:

```yaml
before_install:
  - npm install -g recink
script: recink run unit
```

Once we have updated `.recink.yml` and `.travis.yml`, we can commit the code
back to our repository:

```shell
# assuming you're still inside the tutorial repository
git checkout -b tutorial-step3
git add .travis.yml .recink.yml
git commit -m "enable unit testing"
git push --set-upstream origin tutorial-step3
```

### Step 4: Setup Code Coverage

> Step 4 Branch:
https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step4

No matter what unit testing tool you might be using, most of them include code
coverage capability, which is a measurement in percentage used to describe the
degree to which the source code is executed when a particular test suite runs.
Higher percentage in code coverage has lower probability of bugs and undetected
issues, especially when you need to make changes to the code.

To enable code coverage, you need to simply add the following config block to
`.recink.yml`, after test config block:

```yaml
---
$:
  [...]
  test:
    [...]
  coverage:
    reporters:
      text-summary: ~
    pattern:
      - /.+\.js$/i
    ignore:
      - /.+\.spec\.js$/i
      - /.+\.e2e\.js$/i
      - /^(.*\/)?node_modules(\/?$)?/i

deep-todomvc:
  root: 'src/deep-todomvc'
```

No changes are necessary to `.travis.yml`.

Once we have updated `.recink.yml`, we can commit the code back to our
repository:

```shell
# assuming you're still inside the tutorial repository
git checkout -b tutorial-step4
git add .travis.yml .recink.yml
git commit -m "enable code coverage"
git push --set-upstream origin tutorial-step4
```

### Step 5: Setup Code Climate

> Step 5 Branch:
https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step5

Code Climate empowers organizations to take control of their code quality by
incorporating fully configurable test coverage and maintainability data
throughout the development workflow. This service allowed us to:

1. Report code analysis over time and enforce industry accepted coding standards
2. Report code coverage over time and enforce qualitative processes like
    - fail when code coverage percentage drops lower than X percent
    - fail when code coverage difference between two consecutive reports drops
      more than Y percentage points

In order to do that, we need to update both `.travis.yml` and `.recink.yml`
like we did in previous step. Travis CI will store securely Code Climate token,
while recink allow us to cache code coverage reports and enforce above
described quality metrics.

We imported GitHub repository into Code Climate (follow
[this link](https://docs.codeclimate.com/v1.0/docs/importing-repositories)
for detailed instructions) and retrieved the code coverage token (follow
[this link](https://docs.codeclimate.com/v1.0/docs/finding-your-test-coverage-token)
for detailed instructions).

Update `.recink.yml` with below configuration and replace with your token:

```yaml
---
$:
  [...]
  coverage:
    [...]
  codeclimate:
    token: '[replace_with_your_token]'

deep-todomvc:
  root: 'src/deep-todomvc'
```

Support for Code Climate was implemented as an independent recink component,
therefore we need to update `.travis.yml` by changing `before_install` and
`script` lines, as shown below:

```yaml
before_install:
  - npm install -g recink
  - npm install -g recink-codeclimate
script: recink run unit -c recink-codeclimate
```

Once we have updated `.recink.yml` and `.travis.yml`, we can commit the code
back to our repository:

```shell
# assuming you're still inside the tutorial repository
git checkout -b tutorial-step5
git add .travis.yml .recink.yml
git commit -m "enable code climate"
git push --set-upstream origin tutorial-step5
```

[Click to Continue with Next Step](https://github.com/MitocGroup/tutorial-ci-for-serverless/tree/tutorial-step6#step-6-setup-encryption)