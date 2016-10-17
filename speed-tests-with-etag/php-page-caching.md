Http Cache test: php page caching
=============================
2016-10-17

ETag is always on in those tests.


I want to see whether or not a php page is cached by the following browsers:
ff49, chrome54, safari10, opera40.





Test results
-----------------

All tested browsers have the same behaviour: they do not cache a php page no matter what.






Test setup
---------------
Setting those two directives for a php page

```nginx
expires 10m;
add_header Cache-Control public;
```


Then loading the php page in the browser.

