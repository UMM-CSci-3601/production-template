**Caveat:** I make not promises that all of this is exactly correct. I think it's _mostly_ correct, but there are almost certainly some omissions and errors.

The key config files:

- `build.gradle`
- `client/build.gradle`
- `client.package.json`
- `client/web pack.*.js`
- `server/build.gradle`

Let's walk through an trimmed version of the output of running `./gradlew build` on the project.

The main thing the `gradlew` script does is download `gradle` for us so we have it even if it wasn't previously installed on the computer we're sitting in front of. It then passes on any arguments (like `build` in this example) to `gradle`.

After `gradle` is downloaded (if necessary), then `gradle` performs the specified tasks (in this case `build`). To see the list of sub-tasks `gradle` will execute for a given task, use the `-m` flag; if we do `./gradlew -m build` we get:

```
:server:compileJava SKIPPED
:server:processResources SKIPPED
:server:classes SKIPPED
:server:jar SKIPPED
:server:startScripts SKIPPED
:server:distTar SKIPPED
:server:distZip SKIPPED
:server:assemble SKIPPED
:client:nodeSetup SKIPPED
:client:yarnSetup SKIPPED
:client:yarn SKIPPED
:client:yarn_run_test SKIPPED
:client:runClientTests SKIPPED
:server:compileTestJava SKIPPED
:server:processTestResources SKIPPED
:server:testClasses SKIPPED
:server:test SKIPPED
:server:check SKIPPED
:server:build SKIPPED
```

These appear in the order in which they will need to be executed, which is sort of the reverse of the dependency graph. If a `gradle` task (e.g., `build`) depends on another `gradle` task (e.g., `check`), then the dependent task (`check`) appears first because it has to be completed before `build` can be considered done. The exact order you get might vary, especially if you (or we) have made some changes to the configuration or the codebase since this was written. Since the dependency graph is really a _graph_ there can be sets of tasks where there is no specified or required ordering among them. At the top level, for example, this has some of the `:server` tasks before the `:client` tasks, but I think that could have done all or most of the `:server` tasks after doing all the `:client` tasks.

Now let's consider the output of running the `build` task on a completely clean checkout of the project, so essentially everything will need to be done. The whole "download `gradle`" business is silent, but after that there is a ton of output. Note that anything that starts with a colon (`:`) in this output is the name of a `gradle` task. These first lines are actually just lists of `gradle` tasks, where one "calls" (requires) the next, which requires the next, etc.:

```
:server:compileJava
:server:processResources UP-TO-DATE
:server:classes
:server:jar
:server:startScripts
:server:distTar
:server:distZip
:server:assemble
```

These first few tasks compile all the Java (server) code, and construct the various deployment artifacts (the `jar` files, the startup scripts, and the `tar` and `zip` archives. Note that this happens _without running any tests_, which is why `./gradlew assemble` is faster than `./gradlew build`, but more dangerous because you can `assemble` things that simply don't work without any feedback indicating that there is a problem.

Then we make sure we have `node` and `yarn` installed and configured, including an output of the huge package dependency tree for `yarn` (which I've trimmed here).

```
:client:nodeSetup
:client:yarnSetup
/Users/mcphee/Documents/Courses/CSci3601/digital-display-garden-iteration-1-rayquaza/client/yarn/yarn-latest/bin/yarnpkg -> /Users/mcphee/Documents/Courses/CSci3601/digital-display-garden-iteration-1-rayquaza/client/yarn/yarn-latest/lib/node_modules/yarn/bin/yarn.js
/Users/mcphee/Documents/Courses/CSci3601/digital-display-garden-iteration-1-rayquaza/client/yarn/yarn-latest/bin/yarn -> /Users/mcphee/Documents/Courses/CSci3601/digital-display-garden-iteration-1-rayquaza/client/yarn/yarn-latest/lib/node_modules/yarn/bin/yarn.js
/Users/mcphee/Documents/Courses/CSci3601/digital-display-garden-iteration-1-rayquaza/client/yarn/yarn-latest/lib
└─┬ yarn@0.21.3 
  ├─┬ babel-runtime@6.23.0 
  │ ├── core-js@2.4.1 
  │ └── regenerator-runtime@0.10.3 
…
```

Then `gradle` runs the `:client:yarn` task, which starts by downloading any unmet JS dependencies (e.g., the `@angular` libraries). Our dependencies are all specified in `client/package.json`, but most of these have their own dependencies, which have their own dependencies, etc., etc., so there's a lot to be downloaded.

The warnings come from parts of the Angular code depending on different versions of things than the versions we are explicitly downloading somewhere else. We have version `5.0.0-beta.6` specified in `client/package.json`, but Angular obviously wants to use the newer `5.0.1`. (I have no idea why we're specifying the older one; we should probably update that in our repo, and you might try updating it in yours. Similarly, I have no idea why we're using version `2.4.7` of `@angular/forms`, but `2.4.6` for most of the other Angular bits.) 

```
:client:yarn
yarn install v0.21.3
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
warning "@angular/core@2.4.6" has incorrect peer dependency "rxjs@^5.0.1".
warning "@angular/http@2.4.6" has incorrect peer dependency "rxjs@^5.0.1".
warning "@angular/router@3.4.6" has incorrect peer dependency "rxjs@^5.0.1".
warning "@angular/forms@2.4.7" has incorrect peer dependency "@angular/core@2.4.7".
warning "@angular/forms@2.4.7" has incorrect peer dependency "@angular/common@2.4.7".
[4/4] Building fresh packages...
Done in 10.74s.
```

Now we start the `:client:yarn_run_test` task, which will force all our client code to be transpiled and all the Angular magic to be set into motion. The `webpack` library does most of that work for us, so we actually now have `gradle` (which started things off) starting `yarn` which then starts `webpack` to do the JS building.

I have no idea what the "Critical dependency" warnings are about. Someone should probably Google those and see if we can make them go away.

(I've trimmed the huge pile of transpilation output here.)

```
:client:yarn_run_test
yarn run v0.21.3
$ karma start 
webpack: wait until bundle finished: 

[at-loader] Using typescript@2.1.5 from typescript and "tsconfig.json" from /Users/mcphee/Documents/Courses/CSci3601/digital-display-garden-iteration-1-rayquaza/client/tsconfig.json.


[at-loader] Checking started in a separate process...

[at-loader] Ok, 0.66 sec.

[at-loader] Checking started in a separate process...

[at-loader] Ok, 0.509 sec.
Hash: be38ffb5057238d2bbcb
Version: webpack 2.2.1
Time: 6692ms
                                 Asset       Size  Chunks                    Chunk Names
  f4769f9bdb7466be65088239c12046d1.eot    20.1 kB          [emitted]         
  89889688147bd7575d6327160d64e760.svg     109 kB          [emitted]         
  e18bbf611f2a2e43afc071aa2f4e1512.ttf    45.4 kB          [emitted]         
 fa2772327f55d8198301fdb8bcfc8158.woff    23.4 kB          [emitted]         
448c34a56d699c29117adc64c43affeb.woff2      18 kB          [emitted]         
                                vendor    2.25 MB       0  [emitted]  [big]  vendor
                                   app    3.32 MB       1  [emitted]  [big]  app
                           src/test.js    8.71 MB       2  [emitted]  [big]  src/test.js
                            index.html  436 bytes          [emitted]         
chunk    {0} vendor (vendor) 2.02 MB [entry] [rendered]
   [20] ./~/rxjs/Subscription.js 5.98 kB {0} {1} {2} [built]
  [112] ./~/@angular/platform-browser/index.js 635 bytes {0} {1} {2} [built]
  …
     + 893 hidden modules

WARNING in ./~/@angular/core/src/linker/system_js_ng_module_factory_loader.js
71:15-36 Critical dependency: the request of a dependency is an expression

WARNING in ./~/@angular/core/src/linker/system_js_ng_module_factory_loader.js
87:15-102 Critical dependency: the request of a dependency is an expression
Child html-webpack-plugin for "index.html":
         Asset     Size  Chunks  Chunk Names
    index.html  2.82 kB       0  
    chunk    {0} index.html 263 bytes [entry]
        [0] ./~/html-webpack-plugin/lib/loader.js!./src/index.html 263 bytes {0} [built]
webpack: Compiled with warnings.

```

We then actually run the tests, which is `gradle` calling `yarn` calling `karma`. `PhantomJS` is a "fake browser" that is used to run all your JS code. If you think about it, your client tests are running code that is expected to be run in a browser, so there has to be something that plays that role. There are ways (e.g., Selenium) to set up tests that run in real browsers like Chrome or Firefox, but that's a fairly heavyweight solution. `PhantomJS` gives us a lighter weight option that can stand in for the browser and run our JS code.

```
06 04 2017 19:30:21.263:INFO [karma]: Karma v1.4.1 server started at http://0.0.0.0:9876/
06 04 2017 19:30:21.265:INFO [launcher]: Launching browser PhantomJS with unlimited concurrency
06 04 2017 19:30:21.316:INFO [launcher]: Starting browser PhantomJS
06 04 2017 19:30:22.197:INFO [PhantomJS 2.1.1 (Mac OS X 0.0.0)]: Connected on socket TBn0DnVGWf8M2_LsAAAA with id 5857662
PhantomJS 2.1.1 (Mac OS X 0.0.0): Executed 8 of 8 SUCCESS (0.508 secs / 0.595 secs)
Done in 18.82s.
:client:runClientTests
```

Lastly we compile and run the Java code and tests, and then we're done!

```
:server:compileTestJava
:server:processTestResources UP-TO-DATE
:server:testClasses
:server:test
:server:check
:server:build

BUILD SUCCESSFUL

Total time: 46.578 secs
```

