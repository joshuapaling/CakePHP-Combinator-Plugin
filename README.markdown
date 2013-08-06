# CakePHP Combinator Plugin #

A Combinator plugin for CakePHP 2.1 - combine, minify and cache Javascript and CSS files for faster load times.

## Deprecated Project ##

As of now (June 2013) this plugin no longer fits in with my workflow, and I'm no longer maintaining it. The javascript compression uses [JSMin.php](https://code.google.com/p/minify/source/browse/min/lib/JSMin.php), which is old and unmaintained. I found the more javascript I wrote, the more chance I had of coming across an error with the compressed version of my code - so that wasn't cool. There's better JS compression solutions available these days.

In my new (Combinator-plugin-free) workflow, I'm using [CodeKit](http://incident57.com/codekit/). It's awesome. You should try it. I'm writing SASS rather than CSS (also awesome, try it!), and among other things, CodeKit compiles, concatenates and minifies my SASS/CSS and Javascript on my local dev machine, each time I save. So, I upload it already compressed, thus removing the need for this Combinator plugin.

## Introduction ##


This plugin is based on [Cake 1.3 Combinator Article from the Bakery](http://bakery.cakephp.org/articles/st3ph/2010/09/10/combinator-compress-and-combine-your-js-and-css-files). I've made it compatible with CakePHP 2.1, and packaged it as a plugin. I've also upgraded the CSS compression from CSSTidy to the more recent and better maintained [CSS Min](http://code.google.com/p/cssmin/).

The plugin is quick and easy to install. The installation instructions are somewhat long - but that's just to provide clarity.

NOTE - [Mark Story's AssetCompress Plugin](https://github.com/markstory/asset_compress) is far more mature and feature rich that this plugin. This plugin is simpler and requires less configuration.

## Features ##

* Combine Multiple CSS or Javascript files into one
* Minify CSS and Javascript files
* Caches the combined/minified file, so it's only recreated if the files included in it have changed

## Requirements ##

* CakePHP 2.1+ (probably also works with CakePHP 2.0 - I haven't tested)

## Installation ##

### 1. Copy the plugin into your app/Plugin/Combinator directory ###

    git submodule add git@github.com:joshuapaling/CakePHP-Combinator-Plugin.git app/Plugin/Combinator

or download from [https://github.com/joshuapaling/CakePHP-Combinator-Plugin](https://github.com/joshuapaling/CakePHP-Combinator-Plugin)
	
### 2. Load the Plugin ###

In app/Config/bootstrap.php, at the bottom, add a line to load the plugin - either:
	
	CakePlugin::load('Combinator'); // Loads only the combinator plugin

or
	
	CakePlugin::loadAll(); // Loads all plugins at once
	
### 3. Add the Combinator to your Helpers array ###

Add 'Combinator.Combinator' to your Helpers array in app/Controller/AppController.php (the first 'Combinator' refers to the name of the plugin, the second to the name of the helper itself)
	
Your AppController.php might start something like this:
	
	class AppController extends Controller {
		var $helpers = array('Cache','Html','Session','Form','Combinator.Combinator');
		
### 4. Set write permissions ###

Ensure that the directories holding your Javascript and CSS files are writable, so the combinator plugin can write cached files.

### 5. Start Combining your CSS and Javascript files! ###

Add code in your layout file (eg. app/View/Layouts/default.ctp).

A minimal use might look something like this:

	$this->Combinator->add_libs('js', array('main','jquery.min','jquery.cookie')); // include main.js, jquery.min.js, jquery.cookie.js
	$this->Combinator->add_libs('css', array('default','contact','blog')); // include default.css, contact.css, blog.css
	
	echo $this->Combinator->scripts('js'); // Output Javascript files
	echo $this->Combinator->scripts('css'); // Output CSS files

You have the option to set the scripts to load asynchronously by setting the $async param to true

	echo $this->Combinator->scripts('js', true); // Output Javascript files with the async attribute
	
Now here if you want to append any js then you can do that by setting the $toEnd param to true

	$this->Combinator->add_libs('js',array('js1','js2'),true);// append Javascript files at the end of the minified js

same thing possible with css 

	$this->Combinator->add_libs('css',array('css1','css2'),true);// append css files at the end of the minified css

Now if you want to add any inline js or css to your minified js or minified css you can do that :
	
	$inlinejs =" /* your js code here */ ";
	$this->Combinator->add_inline_code('js', $inlinejs);

same with css

	$inlinecss =" /* your css code here */ ";
	$this->Combinator->add_inline_code('css', $inlinecss);

However, I like to set it up as follows, so that my CSS and Javascript files are only minified and cached when I'm not in debug mode:

	$cssFiles = array('default','contact','blog');
	$jsFiles = array('main','jquery.min','jquery.cookie');
	$asyncJsFiles = array('post-init');

	if(Configure::read('debug') == 2){ 
		// Don't compress/cache css/js when we are in debug mode
		echo $this->Html->css($cssFiles);
		echo $this->Html->script($jsFiles);
		echo $this->Html->script($asyncJsFiles,array('async'));
	} else {
		$this->Combinator->add_libs('js', $jsFiles);
		$this->Combinator->add_libs('css', $cssFiles);
		echo $this->Combinator->scripts('js');
		echo $this->Combinator->scripts('css');
		$this->Combinator->reset_lib_list('js');
		$this->Combinator->add_libs('js', $asyncJsFiles);
		echo $this->Combinator->scripts('js',true);
	}
	
## Tricks, Tips and Issues ##

* By default the files are compressed, you can change that by setting via the options of the helper.
* By default the cached files are written to /app/webroot/js and /app/webroot/css. You can change that by setting via the options of the helper. The helper removes the / at the beginning and the end of the path specified.
* If you get a JavaScript error with a packed version of a file it's most likely missing a semi-colon somewhere.

## License ##

GPLv3 - http://www.gnu.org/licenses/gpl.html
