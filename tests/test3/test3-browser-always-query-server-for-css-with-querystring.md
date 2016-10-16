Test3: updating an asset source which expires in 1 year, with query string
===================
2016-10-16



The setup for this test is explained in the main document
above in the hierarchy.



The goal of this test is to store an asset for 1 year in the browser, and observe that changing
the asset on the server doesn't make the browser download the new asset (we would need something like the query string trick for that, 
which was tested in test 2), or not.




We have two request-response exchanges.

When the first request is executed, the styles.css?v=10 resource is already in the browser cache.
It was cached for 1 year.

Then, I update the css source and reload the page, without telling it to the browser.


The following nginx directives apply to the index.html:

```nginx
add_header Cache-Control public;
add_header Cache-Control no-cache;
```

And the ones pertaining to the css file are the following:

```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires 1y;
        access_log off;
        add_header Cache-Control public;
}
```



I focus on styles.css (not index.html) in this test.







Conclusion notes
----------------------------

Fetching a resource with a query string, the tested browsers check with the server whether or not the resource is still fresh.

I personally would have expect that the browsers "serve" the resource to the user without contacting the browsers,
but maybe it's because of the query string? -> test 4 will answer that.







Browsers Comparison
-------------------------

First Browser's Request


header  						|		browsers
------------------------------  |  ------------------------------
GET /styles.css?v=10 HTTP/1.1	|	all
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


Note: actually the If-Modified-Since value correspond to the last "Last-Modified" value returned by the server.
Since I tested multiple browsers one after the other instead of in parallell, there are some differences between the browsers
for this value, but had I tested browsers in parallell it shouldn't.



First Server's Response


header  						|		browsers
------------------------------  |  ------------------------------
HTTP/1.1 304 Not Modified					|	all
Server: nginx/1.11.3			| 	all
Date: $someValue				|	all
Last-Modified: $someValue		| all
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
Cache-Control: max-age=0		| 	all
If-Modified-Since: $someValue			| all
If-None-Match: "5803175c-6614"			| all


Second Server's Response


header  						|		browsers
------------------------------  |  ------------------------------
HTTP/1.1 200 OK					|	all
Server: nginx/1.11.3			| 	all
Date: $someValue				|	all
Content-Type: text/css			|	all
Content-Length: 26133			|	all
Last-Modified: $someValue		| all
Connection: keep-alive								| all
ETag: "$someValue-6615"					| all
Expires: Mon, 16 Oct 2017 $someValue	| all			
Cache-Control: max-age=31536000			| all
Cache-Control: public					| all
Accept-Ranges: bytes					| all









Firefox 49
----------------

```http
GET /styles.css?v=10 HTTP/1.1
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
Date: Sun, 16 Oct 2016 11:50:42 GMT
Last-Modified: Sun, 16 Oct 2016 05:59:56 GMT
Connection: keep-alive
ETag: "5803175c-6614"
Expires: Mon, 16 Oct 2017 11:50:42 GMT
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
If-Modified-Since: Sun, 16 Oct 2016 05:59:56 GMT
If-None-Match: "5803175c-6614"
Cache-Control: max-age=0

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:50:47 GMT
Content-Type: text/css
Content-Length: 26133
Last-Modified: Sun, 16 Oct 2016 11:50:47 GMT
Connection: keep-alive
ETag: "58036997-6615"
Expires: Mon, 16 Oct 2017 11:50:47 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes



```




Chrome 53
----------------

```http
GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58036b67-6614"
If-Modified-Since: Sun, 16 Oct 2016 11:58:31 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:58:51 GMT
Last-Modified: Sun, 16 Oct 2016 11:58:31 GMT
Connection: keep-alive
ETag: "58036b67-6614"
Expires: Mon, 16 Oct 2017 11:58:51 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58036b67-6614"
If-Modified-Since: Sun, 16 Oct 2016 11:58:31 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.59 Safari/537.36
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 11:58:56 GMT
Content-Type: text/css
Content-Length: 26133
Last-Modified: Sun, 16 Oct 2016 11:58:55 GMT
Connection: keep-alive
ETag: "58036b7f-6615"
Expires: Mon, 16 Oct 2017 11:58:56 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes


```





Safari 10
----------------

```http
GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Accept-Encoding: gzip, deflate
Connection: keep-alive
If-None-Match: "58036c3c-6614"
Accept: text/css,*/*;q=0.1
If-Modified-Since: Sun, 16 Oct 2016 12:02:04 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Referer: http://cachesite/
Cache-Control: max-age=0
Accept-Language: en-us

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 12:02:14 GMT
Last-Modified: Sun, 16 Oct 2016 12:02:04 GMT
Connection: keep-alive
ETag: "58036c3c-6614"
Expires: Mon, 16 Oct 2017 12:02:14 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Accept-Encoding: gzip, deflate
Connection: keep-alive
If-None-Match: "58036c3c-6614"
Accept: text/css,*/*;q=0.1
If-Modified-Since: Sun, 16 Oct 2016 12:02:04 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/602.1.50 (KHTML, like Gecko) Version/10.0 Safari/602.1.50
Referer: http://cachesite/
Cache-Control: max-age=0
Accept-Language: en-us

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 12:02:17 GMT
Content-Type: text/css
Content-Length: 26133
Last-Modified: Sun, 16 Oct 2016 12:02:16 GMT
Connection: keep-alive
ETag: "58036c48-6615"
Expires: Mon, 16 Oct 2017 12:02:17 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes



```




Opera 40
--------------

```http

GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58036cb5-6614"
If-Modified-Since: Sun, 16 Oct 2016 12:04:05 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, lzma, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 304 Not Modified
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 12:04:37 GMT
Last-Modified: Sun, 16 Oct 2016 12:04:05 GMT
Connection: keep-alive
ETag: "58036cb5-6614"
Expires: Mon, 16 Oct 2017 12:04:37 GMT
Cache-Control: max-age=31536000
Cache-Control: public


GET /styles.css?v=10 HTTP/1.1
Host: cachesite
Connection: keep-alive
Cache-Control: max-age=0
If-None-Match: "58036cb5-6614"
If-Modified-Since: Sun, 16 Oct 2016 12:04:05 GMT
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36 OPR/40.0.2308.81
Accept: text/css,*/*;q=0.1
Referer: http://cachesite/
Accept-Encoding: gzip, deflate, lzma, sdch
Accept-Language: en-US,en;q=0.8

HTTP/1.1 200 OK
Server: nginx/1.11.3
Date: Sun, 16 Oct 2016 12:04:41 GMT
Content-Type: text/css
Content-Length: 26133
Last-Modified: Sun, 16 Oct 2016 12:04:40 GMT
Connection: keep-alive
ETag: "58036cd8-6615"
Expires: Mon, 16 Oct 2017 12:04:41 GMT
Cache-Control: max-age=31536000
Cache-Control: public
Accept-Ranges: bytes
```

































