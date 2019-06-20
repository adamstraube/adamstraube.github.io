---
layout: post
title: Unit testing legacy JavaScript/jQuery - Part 2
comments: true
tags: Unit-testing Frontend JavaScript
date: 2019-06-20 11:30 +1000
---
If you have followed on from my last post then read on below. If you have come here directly then the first part concerns the refactoring most likely required to be performed on legacy code before implementing tests. You can read about it [here]{{ site.baseurl }}({% post_url 2019-06-13-unit-testing-legacy-javascript-jquery-part-1 %}).

## Requirements

The following dependencies are required before continuing:
- Node.js
- Git + Github with credentials setup (or your versioning system of choice) - 

## Save code into a Version Control System (VCS)

We are checking our code into a VCS for these reasons:
- You can verify what has been added to the code
- A history of the code is easily accessible and can be reverted to an older version relatively easily

Versioning paired with meaningful commit messages quickly provides context for the code at that point in time to further help isolate when bugs/performance issues were introduced.

To create the repository in Github firstly you need to create the repository in the web interface. Once that is done you can follow the commands below.

Let's initialise the git repository, add the `linkBuilder.js` file and commit the changes with a message.

`git init`
`git add linkBuilder.js`
`git commit -m "First commit"`

This repository is still on our local machine. We want to push it to Github in our case so we run the following commands (replace <username> with your Github username):

`git remote add origin git@github.com:<username>/linkBuilder.git`
`git push -u origin master`

The code is now checked into VCS remotely.

## Create configuration files

Jest requires some configuration files before we can use it.
The first one is `package.json`, this is the Node.js dependency configuration. Create it in the project root directory and put in the following.

```Json
{
  "scripts": {
    "test": "jest"
  },
  "jest": {
    "transform": {}
  },
  "devDependencies": {
    "babel-plugin-transform-es2015-modules-commonjs": "^6.26.2",
    "jest": "^24.8.0"
  }
}
```

Next we need a configuration file that tells Jest how to read this project. Create `jest.config.js` with the following code.

```Javascript
module.exports = {
    clearMocks: true,
    moduleFileExtensions: [
        "js",
    ],
    verbose: true,
    testMatch: [
        '<rootDir>/tests/*.test.js'
    ],
};
```

We are using the `export` function which Node.js does not decode this natively. To get around this we need to import the ECMAScript module, create `.babelrc` with the following

```Json
{
  "env": {
    "test": {
      "plugins": ["transform-es2015-modules-commonjs"]
    }
  }
}
```

Finally create a `.gitignore` file to stop files we do not want to be uploaded to git.

```
.idea
node_modules
```

## Install dependencies

The configuration is ready so we can now install Jest and all required dependencies.

Run `npm install` in the project root directory. 

## Create a test

Create a `tests` directory under the project root and create the first test in there. All test files need to be named with the `*.test.js` pattern to be picked up by Jest. 

Create the file `linkBuilder.test.js` and copy in the following code.

```javascript
const linkBuilder = require('../linkBuilder');

/**
 * Check if linkBuilder works correctly
 */
test('linkBuilder: Check if link build works' , () => {
    expect(linkBuilder.linkBuilder('www.test.com'))
        .toBe('https://rewriteprefix.com/login?qurl=www.test.com');
});
```

Now is a good time to check in to Git all the new files we have created.
`git add *`
`git commit -m "Added Jest configuration files and created first test"`
`git push`

## Run the test

Go back to the project root directory and run `npm test`.
If everything worked you should see the following

```shell
> jest

 PASS  tests/linkBuilder.test.js
  √ linkBuilder: Check if link build works (6ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        6.558s
Ran all test suites.
```

## Final notes

[Github link for this post](https://github.com/adamstraube/Jest-Javascript-Testing)

Remember that creating tests is a long term task. As additional functions are added and new bugs are encountered remember to add new tests as part of the process to document expectations of the code. The long term results are overwhelmingly positive, you can be more sure that the code is doing what is expected with passing unit tests than no tests.  

These two posts have only scratched the surface of testing. There is a wealth of information out there so here are a couple of links to continue your journey:
- [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Jest](https://jestjs.io/docs/en/getting-started).

