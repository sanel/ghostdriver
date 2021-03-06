[![Stories in Ready](http://badge.waffle.io/detro/ghostdriver.png)](http://waffle.io/detro/ghostdriver)  
# Ghost Driver

Ghost Driver is a pure JavaScript implementation of the
[WebDriver Wire Protocol](http://code.google.com/p/selenium/wiki/JsonWireProtocol)
for [PhantomJS](http://phantomjs.org/).
It's a Remote WebDriver that uses PhantomJS as back-end.

GhostDriver is designed to be integral part of PhantomJS itself, but it's developed in isolation and progress is tracked
by this Repository.

* Current _GhostDriver_ stable version is `"1.0.3"`
* Current _PhantomJS-integrated_ version is `"1.0.3"`: contained in PhantomJS `"1.9.x"`
* Current _PhantomJSDriver_ (Java binding) stable version is `"1.0.3"`

For more info, please take a look at the [changelog](https://github.com/detro/ghostdriver/blob/master/CHANGELOG.md).

The project was created and is lead by [Ivan De Marino](https://github.com/detro).

## Requirements (for users)

* PhantomJS `">= 1.8.0`": latest stable GhostDriver will always be part of latest stable PhantomJS
* Selenium version `">= 2.28.0`"

## Requirements (for developers): checkout and compile Ivan De Marino's PhantomJS `ghostdriver-dev` branch:

1. Prepare your machine for building PhantomJS as documented [here](http://phantomjs.org/build.html), then...
2. Add `detro` remote to local PhantomJS repo: `git remote add detro https://github.com/detro/phantomjs.git`
3. Checkout the `ghostdriver-dev` branch: `git fetch detro && git checkout -b detro-ghostdriver-dev remotes/detro/ghostdriver-dev`
4. Build: `./build.sh`
5. Go make some coffee (this might take a while...)
6. `phantomjs --webdriver=8080` to **launch PhantomJS in Remote WebDriver mode**

**NOTE:** Type `phantomjs -h` for more options.

## (Java) Bindings

This project provides WebDriver bindings for Java under the name _PhantomJSDriver_.
[Here is the JavaDoc](http://cdn.ivandemarino.me/phantomjsdriver-javadoc/index.html).

Bindings for other languages (C#, Python, Ruby, ...) are developed and maintained
under the same name within the [Selenium project](http://docs.seleniumhq.org/docs/) itself.

## How to use it

Launching PhantomJS in Remote WebDriver mode it's simple:
```bash
$ phantomjs --webdriver=PORT
```
Once started, you can use any `RemoteWebDriver` implementation to send commands to it. I advice to take a look to the
`/test` directory for examples.

### Run the tests

Here I show how to clone this repo and kick start the (Java) tests. You need
[Java SDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
to run them (I tested it with Java 7, but should work with Java 6 too).

1. `git clone https://github.com/detro/ghostdriver.git`
2. Configure `phantomjs_exec_path` inside `ghostdriver/test/config.ini` to point at the build of PhantomJS you just did
3. `cd ghostdriver/test/java; ./gradlew test`

### Run GhostDriver yourself and launch tests against that instance

1. `phantomjs --webdriver=PORT`
2. Configure `driver` inside `ghostdriver/test/config.ini` to point at the URL `http://localhost:PORT`
3. `cd ghostdriver/test/java; ./gradlew test`

### Register GhostDriver with a Selenium Grid hub

1. Launch the grid server, which listens on 4444 by default: `java -jar /path/to/selenium-server-standalone-2.25.0.jar -role hub`
2. Register with the hub: `phantomjs --webdriver=8080 --webdriver-selenium-grid-hub=http://127.0.0.1:4444`
3. Now you can use your normal webdriver client with `http://127.0.0.1:4444` and just request `browserName: phantomjs`

### Include Java Bindings in your Maven project

Just add the following to your `pom.xml`:

```xml
<dependency>
    <groupId>com.github.detro.ghostdriver</groupId>
    <artifactId>phantomjsdriver</artifactId>
    <version>1.0.3</version>
</dependency>
```

### Include Java Bindings in your Gradle project

Just add the following to your `build.gradle`:

```gradle
dependencies {
    ...
    testCompile "com.github.detro.ghostdriver:phantomjsdriver:1.0.3"
    ...
}
```

## Project Directory Structure

Here follows the output of the `tree` command, trimmed of files and "build directories":

```bash
.
bb�b� binding
bB B  bb�b� java
bB B      bb�b� jars            <--- JARs containing Binding, related Source and related JavaDoc
bB B      bb�b� src             <--- Java Binding Source
|
bb�b� src                     <--- GhostDriver JavaScript core source
bB B  bb�b� request_handlers    <--- JavaScript "classes/functions" that handle HTTP Requests
bB B  bb�b� third_party         <--- Third party/utility code
bB B      bb�b� webdriver-atoms <--- WebDriver Atoms, automatically imported from the Selenium project
|
bb�b� test
bB B  bb�b� java
bB B  bB B  bb�b� src             <--- Java Tests
bB B  bb�b� python              <--- Python Tests
|
bb�b� tools                   <--- Tools (import/export)
```

### WebDriver Atoms

Being GhostDriver a WebDriver implementation, it embeds the standard/default WebDriver Atoms to operate inside open
webpages. In the specific, the Atoms cover scenarios where the "native" PhantomJS `require('webpage')` don't stretch.

Documentation about how those work can be found [here](http://code.google.com/p/selenium/wiki/AutomationAtoms)
and [here](http://www.aosabook.org/en/selenium.html).

How are those Atoms making their way into GhostDriver? If you look inside the `/tools` directory you can find a bash
script: `/tools/import_atoms.sh`. That script accepts the path to a Selenium local repo, runs the
[CrazyFunBuild](http://code.google.com/p/selenium/wiki/CrazyFunBuild) to produce the compressed/minified Atoms,
grabs those and copies them over to the `/src/third_party/webdriver-atoms` directory.

The Atoms original source lives inside the Selenium repo in the subtree of `/javascript`. To understand how the build
works, you need to spend a bit of time reading about
[CrazyFunBuild](http://code.google.com/p/selenium/wiki/CrazyFunBuild): worth your time if you want to contribute to
GhostDriver (or any WebDriver, as a matter of fact).

One thing it's important to mention, is that CrazyFunBuild relies on the content of `build.desc` file to understand
what and how to build it. Those files define what exactly is built and what it depends on. In the case of the Atoms,
the word "build" means "run Google Closure Compiler over a set of files and compress functions into Atoms".
The definition of the Atoms that GhostDriver uses lives at `/tools/atoms_build_dir/build.desc`.

Let's take this small portion of our `build.desc`:
```
js_deps(name = "deps",
  srcs = "*.js",
  deps = ["//javascript/atoms:deps",
          "//javascript/webdriver/atoms:deps"])

js_fragment(name = "get_element_from_cache",
  module = "bot.inject.cache",
  function = "bot.inject.cache.getElement",
  deps = [ "//javascript/atoms:deps" ])

js_deps(name = "build_atoms",
  deps = [
    ...
    "//javascript/webdriver/atoms:execute_script",
    ...
  ]
```
The first part (`js_deps(name = "deps"...`) declares what are the dependency of this `build.desc`: with that CrazyFunBuild knows
what to build before fulfilling our build.

The second part (`js_fragment(...`) defines an Atom: the `get_element_from_cache` is going to be the name of
an Atom to build; it can be found in the module `bot.inject.cache` and is realised by the function named
`bot.inject.cache.getElement`.

The third part (`js_deps(name = "build_atoms"...`) is a list of the Atoms (either defined by something like the second
part or in one of the files we declared as dependency) that we want to build.

If you reached this stage in understanding the Atoms, you are ready to go further by yourself.

## Presentation and Slides (old)

* _April 2012_ - Presented GhostDriver at [Selenium Conference 2012](http://www.seleniumconf.org/speakers/#IDM):
[slides](http://cdn.ivandemarino.me/slides/speed_up_selenium_with_phantomjs/index.html)
and [video](http://blog.ivandemarino.me/2012/05/01/Me-the-Selenium-Conference-2012).
* _March 2013_ - Updates about GhostDriver at [Selenium Camp 2013](http://seleniumcamp.com/materials/ghost-driver/):
[slides](https://speakerdeck.com/detronizator/getting-started-with-ghostdriver)
and [personal blog post](http://blog.ivandemarino.me/2013/03/03/Me-Selenium-Camp-2013).

## Contributions and/or Bug Report

You can contribute by testing GhostDriver, reporting bugs and issues, or submitting Pull Requests.
Any **help is welcome**, but bear in mind the following base principles:

* Issue reporting requires a reproducible example, otherwise will most probably be **closed withouth warning**
* Squash your commits by theme: I prefer a clean, readable log
* Maintain consistency with the code-style you are surrounded by
* If you are going to make a big, substantial change, let's discuss it first
* I **HATE** CoffeeScript: assume I'm going to laugh off any "contribution" that contains such _aberrating crap_!
* Open Source is NOT a democracy (and I mean it!)

## License
GhostDriver is distributed under [BSD License](http://www.opensource.org/licenses/BSD-2-Clause).

## Release names
See [here](http://en.wikipedia.org/wiki/List_of_ghosts).
