Http Cache test: Cache-Control no-store
=============================
2016-10-16

ETag is always on in those tests.


I want to see the effect of "Cache-Control no-store" on the following browsers:
ff49, chrome53, safari10, opera40.


The expected behaviour is the following (quote from https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching): 

```
By contrast, "no-store" is much simpler. It simply disallows the browser and all intermediate caches from storing any version of the returned response—for example, one containing private personal or banking data. Every time the user requests this asset, a request is sent to the server and a full response is downloaded.
```




Conclusion
--------------
The "Cache-Control: no store" is effective to tell browsers NOT TO cache a resource (at least html or css resource).






Test1
==========

First, let's apply the "Cache-Control: no-store" header to the html page.

(nginx)
```nginx
Cache-control no-store;
```

Results on an html page
----------------------------

ff49, chrome53, safari10, opera40:
The index.html page is never cached. The browser never provides the "If-Modified-Since" or the "If-None-Match" headers.


Note: all browsers provided a "Cache-Control: max-age=0" header.






Results on a css stylesheet
--------------------------------

Let's apply the following nginx conf to the css stylesheet:

```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        add_header Cache-Control no-store;
}
```

ff49, chrome53, safari10, opera40:
The css resource is never cached. The browser never provides the "If-Modified-Since" or the "If-None-Match" headers.






