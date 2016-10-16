Test4: updating an asset source which expires in 1 year, without query string
===================
2016-10-16



The setup for this test is explained in the main document
above in the hierarchy.



The goal of this test is to store an asset for 1 year in the browser, and observe that changing the asset on the server doesn't make the browser download the new asset (we would need something like the query string trick for that, which was tested in test 2), or not.

This is basically the same setup as test3, except that in our html, 
we call styles.css instead of styles.css?v=10, essentially removing
any doubt that test3 introduced.


I focus on styles.css (not index.html) in this test.







Conclusion notes
----------------------------

As far as css is concerned, browsers seems to always check ETags with the server, even though the server specified that the resource
was valid for 1 year, at least in the conditions of our testing.







Browsers Comparison
-------------------------

First Browser's Request


header  						|		browsers
------------------------------  |  ------------------------------
GET /styles.css HTTP/1.1	|	all
Host: cachesite					|   all						
User-Agent: $someValue			|	all
Accept: text/css,*/*;q=0.1		|   all
Accept-Language: $someValue		| 	all
Accept-Encoding: $someValue		| 	all
Referer: http://cachesite/		|   all
Connection: keep-alive			|   all
Cache-Control: max-age=0		| 	all
If-Modified-Since: $someValue			| all
If-None-Match: "5803175c-6614"			| all






First Server's Response


header  						|		browsers
------------------------------  |  ------------------------------
HTTP/1.1 304 Not Modified					|	all
Server: nginx/1.11.3			| 	all
Date: $someValue				|	all
Last-Modified: $someValue		| all
Connection: keep-alive								| all
ETag: "$someValue-6614"								| all
Expires: Mon, 16 Oct 2017 $someValue	| all			
Cache-Control: max-age=31536000			| all
Cache-Control: public					| all



Second Browser's Request


header  						|		browsers
------------------------------  |  ------------------------------
GET /styles.css HTTP/1.1				| all
Host: cachesite								| all
User-Agent: $someValue						| all
Accept: text/css,*/*;q=0.1					| all
Accept-Language: $someValue					| all
Accept-Encoding: $someValue					| all
Referer: http://cachesite/					| all
Connection: keep-alive						| all
Cache-Control: max-age=0		| 	all
If-Modified-Since: $someValue			| all
If-None-Match: "$someValue-6614"			| all


Second Server's Response


header  						|		browsers
------------------------------  |  ------------------------------
HTTP/1.1 200 OK					|	all
Server: nginx/1.11.3			| 	all
Date: $someValue				|	all
Content-Type: text/css			|	all
Content-Length: 26132			|	all
Last-Modified: $someValueIncremented		| all
Connection: keep-alive								| all
ETag: "$someValue-6614"					| all
Expires: Mon, 16 Oct 2017 $someValue	| all			
Cache-Control: max-age=31536000			| all
Cache-Control: public					| all
Accept-Ranges: bytes					| all









Firefox 49
----------------

```http
GET /styles.css HTTP/1.1
Host: cachesite
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0
Accept: text/css,*/*;q=0.1
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://cachesite/
Connection: keep-alive
If-Modified-Since: Sun, 16 Oct 2016 13:06:20 GMT
If-None-Match: "58037b4c-6614"
Cache-Control: max-age=0

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:08:29 GMT
Last-Modified: Sun, 16 Oct 2016 13:06:20 GMT
Connection: keep-alive
ETag: "58037b4c-6614"
Expires: Mon, 16 Oct 2017 13:08:29 GMT
Cache-Control: max-age=31536000
Cache-Control: public

GET /styles.css HTTP/1.1
Host: cachesite
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:49.0) Gecko/20100101 Firefox/49.0
Accept: text/css,*/*;q=0.1
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://cachesite/
Connection: keep-alive
If-Modified-Since: Sun, 16 Oct 2016 13:06:20 GMT
If-None-Match: "58037b4c-6614"
Cache-Control: max-age=0

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:08:32 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 13:08:31 GMT
Connection: keep-alive
ETag: "58037bcf-6614"
Expires: Mon, 16 Oct 2017 13:08:32 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes


```




Chrome 53
----------------

```http

GET /styles.css HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58037bcf-6614"
If-Modified-Since: Sun, 16 Oct 2016 13:08:31 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:10:18 GMT
Last-Modified: Sun, 16 Oct 2016 13:08:31 GMT
Connection: keep-alive
ETag: "58037bcf-6614"
Expires: Mon, 16 Oct 2017 13:10:18 GMT
Cache-Control: max-age=31536000
Cache-Control: public



GET /styles.css HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58037bcf-6614"
If-Modified-Since: Sun, 16 Oct 2016 13:08:31 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:10:23 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 13:10:21 GMT
Connection: keep-alive
ETag: "58037c3d-6614"
Expires: Mon, 16 Oct 2017 13:10:23 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes

```





Safari 10
----------------

```http

GET /styles.css HTTP/1.1
Host: cachesite
Accept-Encoding: gzip, deflate
Connection: keep-alive
If-None-Match: "58037c3d-6614"
Accept: text/css,*/*;q=0.1
If-Modified-Since: Sun, 16 Oct 2016 13:10:21 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Referer: http://cachesite/
Cache-Control: max-age=0
Accept-Language: en-us

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:12:13 GMT
Last-Modified: Sun, 16 Oct 2016 13:10:21 GMT
Connection: keep-alive
ETag: "58037c3d-6614"
Expires: Mon, 16 Oct 2017 13:12:13 GMT
Cache-Control: max-age=31536000
Cache-Control: public



GET /styles.css HTTP/1.1
Host: cachesite
Accept-Encoding: gzip, deflate
Connection: keep-alive
If-None-Match: "58037c3d-6614"
Accept: text/css,*/*;q=0.1
If-Modified-Since: Sun, 16 Oct 2016 13:10:21 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Referer: http://cachesite/
Cache-Control: max-age=0
Accept-Language: en-us

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:12:16 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 13:12:15 GMT
Connection: keep-alive
ETag: "58037caf-6614"
Expires: Mon, 16 Oct 2017 13:12:16 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes

```




Opera 40
--------------

```http
GET /styles.css HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58037caf-6614"
If-Modified-Since: Sun, 16 Oct 2016 13:12:15 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, lzma, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:13:25 GMT
Last-Modified: Sun, 16 Oct 2016 13:12:15 GMT
Connection: keep-alive
ETag: "58037caf-6614"
Expires: Mon, 16 Oct 2017 13:13:25 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58037caf-6614"
If-Modified-Since: Sun, 16 Oct 2016 13:12:15 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, lzma, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 13:13:28 GMT
Content-Type: text/css
Content-Length: 26132
Last-Modified: Sun, 16 Oct 2016 13:13:27 GMT
Connection: keep-alive
ETag: "58037cf7-6614"
Expires: Mon, 16 Oct 2017 13:13:28 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes
```

































