Http Cache test: Cache-Control public
=============================
2016-10-16

ETag is always on in those tests.


I want to see the effect of "Cache-Control public" on the following browsers:
ff49, chrome53, safari10, opera40.


Conclusion
--------------
By default, browsers cache css and html resources, except for safari10 which doesn't cache html resource by default.






Test1
==========

First, let's apply the Cache-Control: public header to the html page.

(nginx)
```nginx
Cache-control public;
```

Results on an html page
----------------------------

ff49, chrome53, opera40:
The index.html page is cached until it's modified.

Note: a simple save in sublime suffices to change the file's last modified time.


Safari10: no cache (Http status=200 every time no matter what)



Let's see what the headers are.

Firefox 49, just pressing the refresh page button many times, and here is the result of the last stroke:

```http
GET / HTTP/1.1
Host: cachesite
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.8,fr;q=0.5,fr-FR;q=0.3
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
If-Modified-Since: Sun, 16 Oct 2016 17:29:29 GMT
If-None-Match: "5803b8f9-662"
Cache-Control: max-age=0

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 17:32:41 GMT
Last-Modified: Sun, 16 Oct 2016 17:29:29 GMT
Connection: keep-alive
ETag: "5803b8f9-662"
Cache-Control: public



```

Safari10: same setup, here is the result of the last stroke:

```http
GET / HTTP/1.1
Host: cachesite
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Accept-Language: en-us
Cache-Control: max-age=0
Accept-Encoding: gzip, deflate

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 17:31:50 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 1634
Last-Modified: Sun, 16 Oct 2016 17:29:29 GMT
Connection: keep-alive
ETag: "5803b8f9-662"
Cache-Control: public
Accept-Ranges: bytes

```


As we can see, safari does not send the If-Modified-Since or the If-None-Match header for html content.

My theory is that it's just browser default.





Results on a css stylesheet
--------------------------------

Let's apply the following nginx conf to the css stylesheet:

```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        add_header Cache-Control public;
}
```

ff49, chrome53, safari10, opera40:
Again, the css stylesheet is cached until it's modified.

Note that this time, safari also behaves as the other browsers.






