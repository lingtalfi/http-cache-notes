Cache headers
===================
2016-10-15





Conclusion
-------------------

One should read this document to understand the basic theory about caching: https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching.


In practice however (see my tests), it seems that the cache theory is just a suggestion that browser can implement
if they want, but they are not forced to.

I tested only for html and css resources, but that gave me an idea of how the caching theory diverges from the reality.


The one directive that works fine is Cache-Control: no-store, which prevents all tested browsers to cache a resource.

Apart from that, I observed that all tested browsers will cache the resource anyway, and they will always check
whether or not it's valid (meaning sending an http request with the If-None-Match and If-Modified-Since headers).

The exception is safari which will never cache an html page, no matter what your directives are.

Of course this is just my own experience, and I encourage you to do your own tests, should you be interested in that.


Anyway, I still believe one should implement http caching following the theory guidelines, as they make a lot of sense,
and let browser do their things (we are already used to it anyway).














The theory
-----------------

In order to understand how http cache works, one should read this document first:

https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching


Basically, a server can attach "cache" information to any resource, and this information is used by
browsers to act appropriately.

The "cache" information provided by the server include the following:

- private/public (cached for a single user only, or can the cache be shared by all users)
- freshness time (how long is the cache valid)
- identifier of the resource (used to confirm whether or not a given resource is still fresh)
- "directives" that control whether or not the browser is allowed to cache the resource and how 


The basic synopsis is the following:

1. The browser fetches a resource from the server.

2. The server provides the resource and indicates that it could be cached by any user for 10 minutes.
	It also says that the resource's fingerprint is AAA.

3. For the next 10 minutes, the browser doesn't need to communicate again with the server should the AAA resource
be asked by the user: it is "served" directly from the browser's cache (aka store).

4. After 10 minutes, the user makes another request for resource AAA.
The browser now is not sure whether or not the resource is still the same as before or if it has been updated.
Therefore, it asks the resource's fingerprint to the server.

The server might respond with AAA, in which case the browser concludes that the resource hasn't been updated, and
then the browser "serves" the resource to the user from its store.

The other case is when the server responds with BBB. In this case, the browser sees that the resource has been updated,
and therefore it needs to re-request the whole resource from the server.



Important tip: there is no mechanism in http that allow you to invalidate a cache once its freshness time has been set
in the browser's cache. However, you can work around this by changing the url of the resource (styles.v1.css -> styles.v2.css), thus forcing the browser to redownload the new resource.



Question: does this trick work for invalidating resources: style.css -> style.css?v=2?

We will answer this question later by testing it, continue reading...



### Nuance between cache and store

To cache and store is not exactly the same thing.

The browser can either store or cache a resource fetched from the server.

Cache is stronger, because the browser can then "serve" it directly to the user without asking the server (while its fresh).

Store is more an intermediate step: when the browser stores a resource, it can "serve" it to the user, provided that
it has confirmed with the server that the resource has not changed (using ETag method).

This comes useful when you specify the Cache-Control header server-side: no-cache means that the browser
is allowed to store the resource nonetheless (meaning it can keep a copy of the content, but must check
that the resource has not changed before "serving" it to the user, everytime).

no-store means that the browser is even not allowed to store the resource.
In other words, it must fetch the whole content everytime with the server.








Let's test it
-----------------------------

Our goal is to verify that the theory works for the major browsers.
We will test chrome53, firefox49, safari 10, opera40, ie11 and edge 13 (both via virtualbox).


In order to test http caching, I will create a fake website called cachesite.
The website contains a php page, and I will use nginx as the web server.



### The site structure and the battle plan


```
- mysite
----- index.html 
----- styles.css 
```




### First, turn off open_file_cache

The nginx open_file_cache module caches (amongst other things) open file descriptors, their sizes and modification times.

This directly affects the value of the ETag, which is based on both the last modified time (first part before the dash) and
the content-length (second part after the dash).

So with open_file_cache on and the following directive for instance: 

```nginx
open_file_cache_valid  60s;
```

this would mean that nginx would not update the ETag until 60 seconds have passed.
While this may be good in production, I will just drop it during this test phase, to have fresh ETags every
time I make a request.







### First exchange test

Let's see how browsers handle a basic cache scenario of the styles.css file.

Here is our nginx conf playground, that we will modify on a per test basis:

```nginx
events {}

http {

	charset utf-8;
    include mime.types;

	server{
		listen 80;
		server_name cachesite;
		root "/Volumes/Macintosh HD 2/it/server/http-cache-notes/cachesite";
		index index.html;


	    open_file_cache off;
	    etag on; 
	    add_header Cache-Control public;
	    add_header Cache-Control no-cache;


	    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
	            expires 1y;
	            access_log off;
	            add_header Cache-Control public;
	    }


	    # pass all php files to php-fpm/php-fcgi server 
	    location ~ \.php {
	        include fastcgi_params;
	        include fastcgi.conf;
	        
	        fastcgi_pass 127.0.0.1:9000;
	    }   		
	}
}
```



I made some tests, they are available in the "tests" directory and the "speed-tests-with-etag" directory of this repository.











Sources:
best: https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching

http://nsscreencast.com/episodes/15-http-caching
https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control



