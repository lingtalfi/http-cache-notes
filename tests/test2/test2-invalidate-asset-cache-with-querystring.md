Test1: invalidate asset cache with a query string
===================
2016-10-16



The setup for this test is explained in the main document
above in the hierarchy.



The goal of this test is to see if we can invalidate the styles.css's browser cached resource 
with a query string.


When the first request is executed, the styles.css?v=4 resource is already in the browser cache.
Then I change the source code of index.html and make it call styles.css?v=5 instead of styles.css?v=4.

In order to make a decent test, we need to force the browser to download index.html every time.
The following nginx directives apply to the index.html:

```nginx
add_header Cache-Control public;
add_header Cache-Control no-cache;
```


I focus on styles.css (not index.html) in this test.







Conclusion notes
----------------------------

The query string trick is effective.

I also tested (in all browsers) that even if newest versions arrive, they don't discard older versions.
This means that if I request styles.css?v=5, then styles.css?v=6, then styles.css?v=7,
then all those versions are in cache, and I can just recall them (styles.css?v=5, 6 or 7) directly
from the cache (assuming with respect of their own expiration date).






Browsers Comparison
-------------------------

First Browser's Request


header  						|		browsers
------------------------------  |  ------------------------------
GET /styles.css?v=4 HTTP/1.1	|	all
Host: cachesite					|   all						
User-Agent: $someValue			|	all
Accept: text/css,*/*;q=0.1		|   all
Accept-Language: $someValue		| 	all
Accept-Encoding: $someValue		| 	all
Referer: http://cachesite/		|   all
Connection: keep-alive			|   all
Cache-Control: max-age=0		| 	all
If-Modified-Since: Sun, 16 Oct 2016 05:59:56 GMT			| all
If-None-Match: "5803175c-6614"								| all




First Server's Response


header  						|		browsers
------------------------------  |  ------------------------------
HTTP/1.1 304 Not Modified					|	all
Server: nginx/1.11.3			| 	all
Date: $someValue				|	all
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT		| all
Connection: keep-alive								| all
ETag: "5803175c-6614"								| all
Expires: Mon, 16 Oct 2017 $someValue	| all			
Cache-Control: max-age=31536000			| all
Cache-Control: public					| all



Second Browser's Request


header  						|		browsers
------------------------------  |  ------------------------------
GET /styles.css?v=10 HTTP/1.1				| all
Host: cachesite								| all
User-Agent: $someValue						| all
Accept: text/css,*/*;q=0.1					| all
Accept-Language: $someValue					| all
Accept-Encoding: $someValue					| all
Referer: http://cachesite/					| all
Connection: keep-alive						| all
Cache-Control: max-age=0					| chrome53, safari10, opera40



Second Server's Response


header  						|		browsers
------------------------------  |  ------------------------------
HTTP/1.1 200 OK					|	all
Server: nginx/1.11.3			| 	all
Date: $someValue				|	all
Content-Type: text/css			|	all
Content-Length: 26132			|	all
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT		| all
Connection: keep-alive								| all
ETag: "5803175c-6614"								| all
Expires: Mon, 16 Oct 2017 $someValue	| all			
Cache-Control: max-age=31536000			| all
Cache-Control: public					| all
Accept-Ranges: bytes					| all









Firefox 49
----------------

```http

GET /styles.css?v=4 HTTP/1.1
Host: cachesite
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0
Accept: text/css,*/*;q=0.1
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://cachesite/
Connection: keep-alive
If-Modified-Since: Sun, 16 Oct 2016 05:59:56 GMT
If-None-Match: "5803175c-6614"
Cache-Control: max-age=0


HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:06:57 GMT
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:06:57 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0
Accept: text/css,*/*;q=0.1
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://cachesite/
Connection: keep-alive

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:12:37 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:12:37 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes


```




Chrome 53
----------------

```http
GET /styles.css?v=4 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8
If-None-Match: "5803175c-6614"
If-Modified-Since: Sun, 16 Oct 2016 05:59:56 GMT

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:14:14 GMT
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:14:14 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8


HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:14:21 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:14:21 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes


```





Safari 10
----------------

```http
GET /styles.css?v=4 HTTP/1.1
Host: cachesite
Accept-Encoding: gzip, deflate
Connection: keep-alive
If-None-Match: "5803175c-6614"
Accept: text/css,*/*;q=0.1
If-Modified-Since: Sun, 16 Oct 2016 05:59:56 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Referer: http://cachesite/
Cache-Control: max-age=0
Accept-Language: en-us

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:19:31 GMT
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:19:31 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Accept-Encoding: gzip, deflate
Connection: keep-alive
Accept: text/css,*/*;q=0.1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Accept-Language: en-us
Referer: http://cachesite/
Cache-Control: max-age=0

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:19:37 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:19:37 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes

```





Opera 40
--------------

```http
GET /styles.css?v=4 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "5803175c-6614"
If-Modified-Since: Sun, 16 Oct 2016 05:59:56 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, lzma, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:22:21 GMT
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:22:21 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, lzma, sdch
Accept-Language: en-US,en;q=0.8


HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:22:24 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:22:24 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes

```

































