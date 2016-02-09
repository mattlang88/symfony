# Cache Service

New challenge! Eventually we'll be rendering a lot of markdown. Maybe even really
big pieces of markdown. Rendering this takes time, so we don't want to do it on
every single request in production. Instead, we need to cache the markdown processing.

Caching is just another tool that we need. Some service that is really good at
caching a string and letting us fetch that string later. Fortunately, Symfony comes
with a bundle called `DoctrineCacheBundle`. 

Double check that you have it in your project's composer.json file. If for some reason
you don't have it just use composer to grab it. 

By default it isn't instatiated. In the `AppKernel` file add that to the project since
we want to use the services it provides. `new DoctrineCacheBundle()`. That added a
use statement on top of the class. Just for consistency, add the full namespace down
at the bottom like the rest of the bundles. Awesome!

Of course this bundle has its own documenation so you can read how to install and 
configure it. I want to go the hard way because we can figure out how to use it
entirely with the tools infront of us. 

We just learned that every bundle can be configured. Back in the terminal, run
`./bin/console config:dump-reference` and now that we've plugged in this bundle
we have a new entry called `doctrine_cache`. Rerun our command with `doctrine_cache`
at the end. 

Nice! This prints out a huge configuration file that shows us exactly how we can
configure different cache objects. We have a `providers` key that you can give a name,
a type which is probably something like `file_system`, `apc`, or `memcache`. And then, based on
on which type you chose there is some additional configuration below it. I don't expect
you to just look at this and know how to use the bundle. You should read its documentation.
But once you get used to how bundles are configured you'll be able to skip that
process, just look at this configuration dump to figure out how things should work.

Let's configure a cache object! Open up `config.yml` and let's start with 
`doctrine_cache` and then below that there is a `providers` key and below that
there is a `name` key. 

This `Prototype` thing here in Symfony actually means that you can make this name
anything. Let's make it `my_markdown_cache`. You'll see how that's going to work
in a second. Below that, we need a type which we'll set to `file_system`. This is one
of the built in types of caches that you can use with this bundle, just read the
bundle's docs to figure that out. 

Perfect, that's all we need to do! Now since it's the bundle's job to provide us with
services what we've just done is told that bundle to give us a new service that caches
things on our local file system. 

Back to our terminal with `./bin/console debug:container` and I'll search for 
`markdown_cache` since that's the name that I gave to it. 

And voila! We have one new service called `doctrine_cache.providers.my_markdown_cache`.
You'll notice that the service name kind of matches `doctrine_cache` `providers`
`my_markdown_cache`, it doesn't need to that's just how this bundle works. 

The point is, we've configured that bundle to give us yet another tool in our
tool kit. 

Let's use this in our controller. In `GenusController` first, let's get that cache
out. `$cache = $this->get('')` and just start typing `markdown` and there's our service
right there! Perfect! 

Now, we'll need the cache key. We just want to make sure that the same string doesn't
get rendered twice through markdown as it gets cached. So, we'll just add `$key = md5('$funFact');`
Now use that object by saying `if ($cache->contains($key))` if we've already cached
that key, then simply `$funFact = $cache->fetch($key);`. Else, we're going to need
to run through the markdown. 

To make this a little more interesting, put a `sleep(1);` to pretend like our markdown
transformation is taking a really long time, this will make it easier to test this. 
Then we'll make the fun fact and finish with `$cache->save($key)` and then save the 
new `$funFact` cached version. There we go! 

Back to the browser, and watch closely. Refresh, and things are moving a bit slow.
The important thing is that in the web debug toolbar you can see 1048 ms. Refresh again
and now it's super fast! 41 ms because the second time it comes from the cache directory.
Uhh... so where is this being cached?

Out of the box this bundle caches into the `var/cache` directory. Over in the terminal
run `ls var/cache/` and then `ls var/cache/dev` and inside of there is a `doctrine`
directory and a `cache/file_system` directory. Ah ha, and there is our cached markdown.
The point here is that one of the assumptions that the bundle made is where you want
it to actually cache things on your file system. Clearly, there needs to be a way
to control that as well. 

Rerun the `config:dump-reference doctrine_cache` and we'll see that there is a directory
key we can use to specify things further. Back in our IDE add some new configuration for
`directory`. Let's put things in `/tmp/doctrine_cache` for the time being. Don't worry
we'll change that later. 

Back to the browser and refresh! Poseidon's beard, check out that HUGE error! 
"Unrecognized option 'directory'". One of the really cool things about configuration
in Symfony is that it's validated. If you mess something up, which I totally just did,
it's not going to fail silently, it's going to tell you. 

Going back to the terminal, we'll see that the problem here is that I actually needed
to have `providers:`, my provider `name:` and then a `file_system` key before the
`directory` key. In the IDE let's plug in `file_system` just above directory and fix
the indentation. Now that's fixed, back to the browser to refresh!

It should be slow the first time, just like that. And then it's super fast the second
time. In the terminal we can even see the `/tmp/doctrine_cache/` directory. 

This is the whole thing here, we installed a bundle, and then we used its configuration
to update one specific service. The way you configure a bundle is going to be different
for every single one. 

I used `config:dump-reference` to reverse engineer things which is great and I want
you to know that is there. But, don't skip on reading the bundle's documentation. 

