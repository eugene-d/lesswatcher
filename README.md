# Lesswatcher

A simple command-line utility to watch multiple directories for changes to LESS files (e.g. in both /less and /css), and trigger lessc compilation in a target directory (e.g. in just /css).  


## Features
  - Cross-platform (OSX and Windows 7, at least)
  - Watches filesystem for changes to .less files 
  - Triggers lessc compilation of only those .css files your app actually needs


## Opinionated Rationale

### Offline lessc makes sense

There's a pretty good argument for doing lessc compilation offline. Doing it in the browser is a non-starter for performance reasons. Doing it on the server is an improvement, but still consumes resources and adds to overhead. (Also, support for things like detecting changes to @imported LESS files is spotty in some app frameworks' plugins, making development a PITA.) Fortunately, there's a third way: compiling LESS to CSS offline, and deploying the pre-generated CSS. This simplifies the server environment and eliminates any lessc-related performance overhead. 

### Why not just use LiveReload or SimpLESS or... ?

There are many other tools for doing offline lessc, some with nice GUIs and CLIs and nice features... but they tend to be OS-specific, and/or to not lend themselves to versioned or centrally-managed configuration. Whereas thanks to Node (and the excellent Hound module), Lesswatcher works across OSX and Windows, and can leverage a central configuration. So, if your team has heterogeneous dev environments, and you want to standardize on web dev tooling for LESS usage, a solution like this using node + lessc makes sense.

### What about other similar NPM modules?

There are other npm modules that do lessc with directory watching, and some of them even support Windows, but at the time I wrote Lesswatcher, none of them seemed to support a distinction between .less files to watch for changes, versus .less files to compile directly to corresponding .css. So using such tools results in compiling every watched .less file into .css, even for .less files used only in an @import to another .less file. Generating such unused .css artifacts bothered me, enough to motivate writing a new tool.


## Lesswatcher Conventions

Lesswatcher supports a convention of managing LESS files in two places:
  
  - **LESS_DIR**
  for .less files designed for @import only (e.g. vendor-provided LESS like twitter-bootstrap)
  
  - **CSS_DIR**
  for "primary" .less files that compile directly to .css

Lesswatcher watches the filesystem for changes to .less files in both places, then compiles just the CSS_DIR's .less files. This way you avoid generating unused .css files, which would otherwise litter your project and add to bloat and potential confusion. 

The generated .css files should be managed in source control, and referenced directly in markup or app config just like any other .css file. The generated .css files should never be edited by hand. (Managing compiled assets in scm is usually a bad idea, but in this case it's justified given the benefits of offline lessc.)


## Installation

###Prerequisites:
1. Install latest **node** (e.g. 0.8.11) and **npm** (e.g. 1.1.62)
  (If on Windows, use the official nodejs.org/download/ MSI installer)

2. Edit your $PATH env var:
  **Add the npm/bin directory to your $PATH**
  (Its location depends on your system, might be e.g. /usr/local/share/npm/bin)

3. Add a new $NODE_PATH env var:
  **Add the npm/lib/node_modules directory to $NODE_PATH**
  (npm/lib is a peer of npm/bin, it's where node's global modules are installed)

###Install globally:
4. Install lesswatcher globally:

    **$ npm install -g lesswatcher**

###Optional Configuration:

5. Optionally create a **lesswatcher-config.json** file to override the default settings. (See Configuration section for details.)

6. See if it worked! Just cd to your project's web-app directory (i.e. the parent of /less and /css) and type:

    **$ lesswatcher**

That's it! Now, editing .less files will trigger lessc on all less files under your CSS_DIR's directory (with console logging in the terminal window).


## Configuration

In Lesswatcher, the intent is to provide generic defaults that should work out of the box for most projects, i.e.:
Install lesswatcher globally, cd to your project's web-app dir (the parent of less and css subdirectories), and run it. 

Default configuration:

    "LESS_DIR": "<directory-where-lesswatcher-was-invoked>/less",
    "CSS_DIR": "<directory-where-lesswatcher-was-invoked>/css",
    "LESSC_COMPILER": <lessc-version-bundled-with-lesswatcher>, 
    "LESSC_OPTS": {
      "compress": false,
      "yui": false
    }

But customization is supported, via CLI args and/or custom conf files:

####1. Arg overrides: 
  Invoke lesswatcher with CLI args to override corresponding conf settings:
  
    --less_dir (string: fully-qualified path to your LESS files)
    --css_dir (string: fully-qualified path to your CSS files)
    --compiler (string: path to lessc compiler already on your system, e.g. to use a specific version vs what lesswatcher provides)
    --lessc_compress (pass "--compress" option to lessc, see less docs)
    --lessc_yui (pass "--yui-compress" option to lessc, see less docs)
 
  Example:
  
    $ lesswatcher --less_dir=/special/place/for/less --css_dir=/my/very/own/css --compiler=lessc --lessc_yui

Any CLI arg provided will trump corresponding values from the conf files described below.

####2. Custom conf file (custom location) 
  For maximum control, you can specify the location of a custom conf file, by invoking lesswatcher with a --conf argument, e.g.
  
    $ lesswatcher --conf=/fully-qualified-path-to/custom-lesswatcher-conf.json

This configuration file will trump any settings in the default custom conf file location described next.

####3. Custom conf file (default location) 
  For convenience, instead of specifying the location of your custom conf, you can simply put it in a file named **"lesswatcher-conf.json"** in the same web-app dir from which you invoke lesswatcher. Note it must use valid JSON syntax. Any values not provided will fall back to the defaults.
  Example:

    "LESS_DIR": "/my/custom/dir/for/lessfiles",
    "CSS_DIR": "/my/custom/dir/for/css",
    "LESSC_COMPILER": "/usr/local/bin/lessc-v1.3.1/lessc"

Finally, any values not defined via CLI arg or conf file will fall back to the defaults. 


## TODO - Possible future features

* support watching for newly created files without restart
* show lessc errors instead of exiting


## LICENSE

See the LICENSE file. 

