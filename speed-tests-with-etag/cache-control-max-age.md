Http Cache test: Cache-Control max-age
=============================
2016-10-16

ETag is always on in those tests.


I want to see the effect of "Cache-Control max-age" on the following browsers:
ff49, chrome53, safari10, opera40.


The expected behaviour is the following (quote from https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching): 

```
This directive specifies the maximum time in seconds that the fetched response is allowed to be reused from the time of the request. For example, "max-age=60" indicates that the response can be cached and reused for the next 60 seconds.
```




Conclusion
--------------
The "Cache-Control: max-age" tell the browsers how long a resource will be fresh.

However, browsers do use this information as they want.

And from my tests, it doesn't make any difference if you use max-age or not, at least for html and css resources,
because the browser will check for its validity anyway and will invalidate them if necessary.

Also, safari10 will not cache an html page (no matter which value of max-age you have, at least in the current test  conditions which includes ETag).






Test1
==========

First, let's apply the "Cache-Control: max-age" header to the html page.

(nginx)
```nginx
Cache-control max-age=10;
```

Results on an html page
----------------------------

ff49, chrome53, opera40:
The index.html page is cached until it's modified. I would personally wonder: what's the point of max-age then, if it behaves exactly the same with or without, anyway...


safari10:
will not cache the html page.



Results on a css stylesheet
--------------------------------

Let's apply the following nginx conf to the css stylesheet:

```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        add_header Cache-Control max-age=10;
}
```

ff49, chrome53, safari10, opera40:
The css resource is cached until it's modified (exactly the same behaviour as without the "Cache-Control: max-age" header).






