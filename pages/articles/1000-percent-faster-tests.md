# Make your Angular tests 1000% faster by switching from Karma to Jest

!https://res.cloudinary.com/practicaldev/image/fetch/s--7b456hOH--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://thepracticaldev.s3.amazonaws.com/i/9516njx8ysv4tzgu3g5r.jpg!

It sounds sensationalist but it's true. 

One of the projects I'm working on has an Angular 8 frontend with over 1000 unit/component tests. These used to all run in Karma and take around 15mins but now they take about 1 min.

# But why?
What fast tests not good enough for you?
Some other things I've been loving:
* Nice error messages
* Easy debugging in VS Code (finally!)
* Really nice auto run and error message plugins for VS code
* Ability to write to disk (Maybe not that useful but I found it handy for some tests)

# But how?
Well, let me tell ye a story.

Actually scrap that, you're reading this because you want to convert to jest, maybe you've tried it before and failed, maybe you just want to give it a go - either way let's dig into it.

# The approach
If you have a decent sized project, (as with anything in Software) the best way to do it is incrementally.

As we have over 1000 tests, we knew it would take a while to convert them and couldn't do the "big bang" approach as we have about 5 different teams working on the app at any one time - we knew we'd need to run karma and jest side-by-side for a period of time. For us, this ended up being nearly a week but it could have taken way longer!

We naturally are following best software dev practices, so at the end of each step we should be able to create a pull request, run our build, tests and merge to master safely.

Just remember, this is a marathon not a sprint (pardon the pun). As soon as you get a test suite/file passing, commit it. Don't commit broken tests (sounds obvious but you can forget this in the heat of a conversion like this). And don't forget to enlist the help of your fellow developers. This will affect them too so they will want to help out - let them!

With this in mind, our basic approach was this:
- Install jest
- Get the first test running with Jest (perhaps a brand new test)
- Migrate an old test suite/file, using what we've learnt
- Write a script to migrate an old suite (based on the manual process we just went though)
- Migrate the next test suite using the script, adding anything to the script that is missing
- Rinse & Repeat until all the tests are migrated.

Remember, as soon as a test is green -> commit it!
(Jests `--onlyChanged` flag is very handy here)

# Getting started
We start by setting up the jest basics.

Install it:

`npm install --save-dev jest @types/jest jest-preset-angular glob @angular-builders/jest`

Create a `jest.config` (for Angular) in the project folder:
```javascript
var preset = require("jest-preset-angular/jest-preset");
module.exports = {
  ...preset,
  preset: "jest-preset-angular",
  setupFilesAfterEnv: [
    "<rootDir>/node_modules/jest-preset-angular/build/setupJest.js"
  ],
  testMatch: ["**/*.test.ts"],
  globals: {
    ...preset.globals,
    "ts-jest": {
      ...preset.globals["ts-jest"],
      tsConfig: "src/tsconfig.test.json",
      isolatedModules: true
    }
  }
};
```

Create a `tsconfig.test.json` for jest:
This should be very similar to your main tsconfig, but with jest types added.

```
{
  "compilerOptions": {
    "baseUrl": "",
    "declaration": false,
    "downlevelIteration": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "mapRoot": "./",
    "module": "commonjs",
    "moduleResolution": "node",
    "outDir": "../dist/out-tsc",
    "sourceMap": true,
    "target": "es5",
    "lib": ["es2015", "dom"],
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "typeRoots": ["../node_modules/@types"]
  }
}
```

If you use `jasmine.createSpy` or `jasmine.createSpyObj`, to aid in the migration, you may need a create-spy.ts:
```
export const createSpyObj = (
  baseName,
  methodNames?
): { [key: string]: jest.Mock<any> } => {
  let obj: any = {};

  for (let i = 0; i < methodNames.length; i++) {
    obj[methodNames[i]] = jest.fn();
  }

  return obj;
};

export const createSpy = (
  baseName?
) => {
  return jest.fn();
};
```
Import this where ever you have broken tests (after running the migration script) relating to creatSpy or createSpyObj.


In order to get jest to actually run, you'll need to create a new test config for karma in your `angular.json` and replace the existing one with jest:
```
        "test": {
          "builder": "@angular-builders/jest:run",
          "options": {
            "tsConfig": "<rootDir>/src/tsconfig.test.json"
          }
        },
```
If you simply replace karma with jest, you will not be able to run karma and jest tests side-by-side!

Instead, create a new project config inside your `angular.json` that resembles something like this:

```
    "karma": {
      "root": "",
      "sourceRoot": "src",
      "projectType": "application",
      "architect": {
        "test": {
          "builder": "@angular-devkit/build-angular:karma",
          "options": {
            "main": "src/test.ts",
            "tsConfig": "src/tsconfig.json",
            "karmaConfig": "./karma.conf.js",
            "polyfills": "src/polyfills.ts",
            "scripts": [

            ],
            "styles": [],
            "assets": []
          }
        }
      }
    },
```

Then add another script to your `package.json`
`"test-karma": "ng test karma"`

From now on, `jest` will run your jest tests and `npm run test-karma` will run the leftover karma tests.

# Visualising Progress
Since this is a big job, we want to see some progress and get others involved, so having a script that outputs the percentage of tests that have been converted is also a really good morale boost.

Here is the script we used. We simply ran it at the end of our builds.

Create a file and name it something painfully obvious, like `check-progress.js`:
```javascript
var glob = require("glob")

Reset = "\x1b[0m"
FgRed = "\x1b[31m"
FgGreen = "\x1b[32m"
FgYellow = "\x1b[33m"
FgWhite = "\x1b[37m"

let specs = glob.sync("src/**/*.spec.ts");
let tests = glob.sync("src/**/*.test.ts");

console.log(FgYellow, `${specs.join('\n')}`, Reset)

if (specs.length) {
  console.log(FgRed, specs.length + " slow karma tests")
} else {
  console.log(FgGreen, 'Wooooooooooooooooooooo! Jest conversion complete!')
}
console.log(FgWhite, tests.length + " fast jest tests")
console.log(FgGreen, (tests.length * 100 / (tests.length + specs.length)).toFixed(2) + "% complete in switching tests to jest", Reset)
```

Then just run `node check-progress.js`


# Plugins
If you are using `VS Code`, you may find the plugins `Jest` and `Jest Runner` very handy for running and also debugging your tests (Finally!).

# The actual migration
With all our setup out of the way, we should be able to start incrementally converting tests.
There are tools out there like `jest-codemods` that are meant to do the conversion for you but we didn't have any luck with this, so we built our own. Below is the simple script we used. When we found a case or type of test it couldn't handle, we simply added to the script. You will likely need to continue that pattern for your tests, but this might be a good start.

Note that since we want to run karma specs alongside jest tests (until we've finished converting all the tests), we have chosen the convention of `spec.ts` for karma tests and `test.ts` for jest tests. The script below will, after conversion, rename the spec to `*.test.ts` so your git diff will likely show a bunch of deleted files (the spec.ts files). For this reason it's probably best to just run this on a single test file to start with.

Create a file called `convert-to-jest.js`:

```javascript
var fs = require('fs')
var filename = process.argv[2]

if (!filename) {
  let specs = require('glob').sync("src/**/*.spec.ts");
  for (spec of specs) {
    if (!spec.includes('pact')) {
      convertToJest(spec);
    }
  }
} else {
  convertToJest(filename);
}

function convertToJest(filename) {
  if (!filename.startsWith('C:')) {
    filename = './' + filename
  }

  fs.readFile(filename, 'utf8', function (err, data) {
    if (err) {
      return console.log(err);
    }
    var result = data;
    result = result.replace(' } from \'@ngneat/spectator\';', ', SpyObject } from \'@ngneat/spectator/jest\';');
    result = result.replace('} from \'@ngneat/spectator\';', ', SpyObject } from \'@ngneat/spectator/jest\';');
    result = result.replace(/SpyObj</g, 'SpyObject<');
    result = result.replace(/\.and\.returnValue/g, '.mockReturnValue');
    result = result.replace(/\.spec\'/g, '.test');
    result = result.replace(/jasmine\.SpyObj/g, 'SpyObj');
    result = result.replace(/spyOn/g, 'jest.spyOn');
    result = result.replace(/spyOnProperty/g, 'spyOn');
    result = result.replace(/expect\((.*)\.calls\.first\(\)\.args\)\.toEqual\(\[(.*)\]\);/g, 'expect($1).toHaveBeenCalledWith($2);')
    result = result.replace(/expect\((.*)\.calls\.any\(\)\)\.toBe\((.*)\);/g, 'expect($1).toHaveBeenCalledWith($2);');
    result = result.replace(/expect\((.*)\.calls\.mostRecent\(\)(\.args\[.*\])?\)\.toEqual\((.*)\);/g, 'expect($1).toHaveBeenCalledWith($2);');
    result = result.replace(/expect\((.*)\.calls\.count\(\)\)\.toBe\((.*)\);/g, 'expect($1).toHaveBeenCalledTimes($2);');
    result = result.replace(/expect\((.*)\.calls\.count\(\)\)\.toEqual\((.*)\);/g, 'expect($1).toHaveBeenCalledTimes($2);');
    result = result.replace(/\.calls\.first\(\).args/g, '.mock.calls[0].args');
    result = result.replace(/and.callFake/g, 'mockImplementation');
    // result = result.replace(/createService\(/g, 'createServiceFactory(');
    // result = result.replace(/createService,/g, 'createServiceFactory,');

    if (result.includes('createSpyObj')) {
      result = result.replace(/jasmine\.createSpyObj/g, 'createSpyObj');
      result = result.replace(/createSpyObject/g, 'createSpyObj');

      var numberOfSlashesinFilename = (filename.replace('./src/app/', '').match(/\//g) || []).length;
      var prefix = "./"
      for (var i = 0; i < numberOfSlashesinFilename; i++) {
        prefix += "../"
      }

      result = 'import { createSpyObj } from \'' + prefix + 'shared/testing/SpyObj\';\r\n' + result;
    }

    result = result.replace('import SpyObj = SpyObj;', '');
    result = result.replace('import Spy = jasmine.Spy;', '');
    result = result.replace('import createSpyObj = createSpyObj;', '');
    result = result.replace(/ Spy;/g, ' jest.SpyInstance;');
    result = result.replace(/jasmine\.Spy;/g, 'jest.SpyInstance;');

    if (!result.includes('@ngneat/spectator') && result.includes('SpyObject')) {
      result = 'import { SpyObject } from \'@ngneat/spectator/jest\';\r\n' + result;
    }
    if (result.includes('MatDialog') && !result.includes('@angular/material/dialog')) {
      result = result.replace(/import \{(.*)MatDialog, (.*)\}/g, 'import {$1$2}');
      result = result.replace(/import \{(.*)MatDialogModule, (.*)\}/g, 'import {$1$2}');
      result = result.replace(/import \{(.*)MatDialogModule(.*)\}/g, 'import {$1$2}');
      result = result.replace(/import \{(.*)MAT_DIALOG_DATA, (.*)\}/g, 'import {$1$2}');
      result = result.replace(/import \{(.*)MatDialogRef, (.*)\}/g, 'import {$1$2}');
      result = 'import { MatDialog, MatDialogModule, MAT_DIALOG_DATA, MatDialogRef } from \'@angular/material/dialog\';\r\n' + result;
    }

    if (result.includes('withArgs')) {
      result = result.replace(/(.*)\.withArgs\((.*)\)\.mockReturnValue\((.*)\)/g, `$1.mockImplementation(flag => {
        switch (flag) {
          case $2:
            return $3;
        }
      })`);
    }

    result = result.replace(/jest\.jest/g, 'jest');

    let newFile = filename.replace('.spec.ts', '.test.ts');
    fs.writeFile(newFile, result, 'utf8', function (err) {
      if (err)
        return console.log(err);
      console.log('Successfully wrote ' + newFile);
      if (newFile != filename) {
        fs.unlinkSync(filename);
      }
    });
  });
}
```
You'll just need to run:
`node convert-to-jest.js <optional path to specific test>`

# The interesting bit
Now we get to the interesting bit - running the test.
Assuming you've setup your `angular.json` for jest correctly, you should be able to just run `ng test`.

I call this "the interesting bit" because I can't really give you much more guidance if it doesn't work. You'll need to figure out why your tests aren't working, for yourself. Of course, if you get lucky and they just work, it's time to convert the next test!

You may also find that if you bulk convert all the tests, there may be some that "just work". If this is the case, you can simply commit these and move on with the rest. You'll also find one command very handy:
`ng test --onlyChanged`
This is git aware and only runs tests that have changes sitting uncommitted in your git repo. You will find this very handy if you try to bulk convert your tests.

Also since jest outputs a lot of error info, when there are failures, you may want to additionally add:
`ng test --onlyChanged --bail`
This means that jest will stop on the first test failure, allowing you to focus on that.

Armed with these simple techniques alone, you should be able to convert a bulk of your tests quite quickly.


# Results (Check my maths)

Our builds used to take about 15mins to run 1200 tests. After converting to jest our tests now take about 1.5mins. Thats a change from 80 test/min up to 800 test/min - 1000% faster! Ok technically I could just say 10x faster but bigger numbers are better right?
