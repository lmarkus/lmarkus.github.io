---
layout: post
title: Grunt sanity for large projects.
excerpt: You don't design monolithic apps. Why should your build process be any different?
---

##Typical Structure
Your project might looks something like this:
<pre>
+-Gruntfile.js
|
+-module_1
|
+-module_2
.
.
.
+-module_N
</pre>

As you increase in complexity, your Gruntfile starts getting larger and harder to read. Let's face it; it might've started out simple
(maybe just a linter), but after you've added template compilation, CSS pre-processing, custom scripts and more, it's getting
hard to manage and troubleshoot.

In particular, your `initConfig()` section might be a sprawling mess.

##Shared configuration
Starting with Grunt@0.4.6 you can now use `config.merge()` ([Documentation](http://gruntjs.com/api/grunt.config#grunt.config.merge)) inside a gruntfile, to incrementally set-up the configuration for your build.

To illustrate with an overly simple example, let's take this project as a starting point:
<pre>
+-Gruntfile.js
|
+-client+
|       +-home.js
|
+-server+
        +-app.js
</pre>


It's got two modules, and let's say that our build process only cares about linting the files.

The gruntfile might look something like this:

{% highlight javascript %}
module.exports = function(grunt){
    grunt.initConfig({
        jshint:{
            files:['client/home.js','server/app.js']
        }
    });
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.registerTask('lint', ['jshint']);
    grunt.registerTask('build', ['lint']);
}
{% endhighlight %}


In order to break down this build, we're going to add a new Gruntfile to each module

<pre>
+-Gruntfile.js
|
+-client+
|       +-home.js
|       +-Gruntfile.js
|
+-server+
        +-app.js
        +-Gruntfile.js
</pre>

Each of these gruntfiles is going to add only the parts it cares about to the config by using `config.merge()`, in addition to
using targets (eg: `jshint:client`) to further segregate what gets built.

This would be one of the sub-gruntfiles:

{% highlight javascript %}
/* client/Gruntfile.js */

module.exports = function(grunt){
    grunt.config.merge({
        jshint:{
            client:{
                src:[__dirname+'/home.js']
            }
        }
    });

    /*optional*/
    grunt.registerTask('lint:client', ['jshint:client']);
    grunt.registerTask('build:client', ['lint:client']);
    /* optional */
}
{% endhighlight %}

And this would be the new main gruntfile

{% highlight javascript %}
/* /Gruntfile.js */
module.exports = function(grunt){

    //Load downstream Gruntfiles
    require('./client/Gruntfile')(grunt);
    require('./server/Gruntfile')(grunt);

    //Set up global tasks
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.registerTask('lint', ['jshint']);
    grunt.registerTask('build', ['lint']);
}
{% endhighlight %}


As you can see, the main gruntfile acts as an index for all the submodules, passing the `grunt` object to each one so the
build can be incrementally configured. (Although it may very well be set up to run some tasks globally for the project).


To invoke it, one would simply do
{% highlight bash %}
$ grunt build
{% endhighlight %}

This will invoke all targets for `build`


##Building submodules
Remember the optional block up there?  If you want to build individual components, you would simply pass that
target as a parameter to grunt:
{% highlight bash %}
$ grunt build:client
{% endhighlight %}

This will only invoke the build target `client`, which is defined in one of the downstream files.

##Live example

[https://github.com/lmarkus/Example_ModularGrunt](https://github.com/lmarkus/Example_ModularGrunt)



