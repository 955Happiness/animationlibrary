#Nginx 学习指南

>##Nginx directives
- block directives
  ---
  块级指令
- simple directives
  ---
  
- directives nest
  ---
  ```$xslt
  [emerg] "map" directive is not allowed here in ...
  ```
  every configure directive does have **A Pre-defined Set Of Use Contexts** in the configuration file
  
  ```$xslt
  server {
    location / {
      proxy_pass http://localhost:8080/;
    }
    
    location ~ \.(gif|jpg|png)$ {
      root /data/images;
    }
  }
  ```
>##Nginx variables
- built-in variables
- user-defined variable
- readable variables
- writeable variables

>##Nginx directive execution order
```$xslt
We start with a confused example:
? location /test {
? set $a 32;
? echo $a;
?
? set $a 56;
? echo $a;
? }

```
the result is:
```$xslt
$ curl 'http://localhost:8080/test
56
56
```
so,what the fuck?Not expected.

>##Nginx模块
- ngx_http_core
  ---
  $http_XXX、$cookie_XXX、$arg_XXX、$request_method、
  $sent_http_XXX variable group etc.
  To summarize, the assignment to $args also successfully **Influences** the behavior of the ngx_proxy module
- ngx_set_misc
  ---
  set_unescape_uri
  ```$xslt
  location /test {
    set_unescape_uri $name $arg_name;
    set_unescape_uri $class $arg_class;
    echo "name: $name";
    echo "class: $class";
  }
  ```
- ngx_proxy
  ---
  by default the ngx_proxy module automatically forwards the current URL **Query String** to the remote HTTP service
  ```$xslt
  server {
    listen 8080;
    location /test {
      set $args "foo=1&bar=2";
      proxy_pass http://127.0.0.1:8081/args;
    }
  }
  server {
    listen 8081;
    location /args {
      echo "args: $args";
    }
  }
  ```
- ngx_map
- ngx_geo
- ngx_echo
  ---
  Variables in Subrequests.
  
  let's check out an example using subrequests:
  ```$xslt
  location /main {
    echo_location /foo;
    echo_location /bar;
  }
  location /foo {
    echo foo;
  }
  location /bar {
    echo bar;
  }
  ```
  the response body of these two subrequests get concatenated together according to
  their running order, to form the final response body of their parent request (for /main):
  ```$xslt
  $ curl 'http://localhost:8080/main'
  foo
  bar
  ```
- ngx_auth_request
  ---
  ```$xslt
  location /main {
    set $var main;
    auth_request /sub;
    echo "main: $var";
  }
  location /sub {
    set $var sub;
    echo "sub: $var";
  }
  ```
  the result is:
  ```$xslt
  $ curl 'http://localhost:8080/main'
  main: sub
  ```
  for the previous example, some readers might ask: "why doesn't the response body of the subrequest appear in the final output?" The answer is simple: it is just
  because the auth_request directive discards the response body of the subrequest it manages, and **Only Checks The Response Status Code** of the subrequest. When
  the status code looks good, like 200, auth_request will just allow Nginx continue processing the main request; otherwise it will immediately abort the main
  request by returning a 403 error page, for example. In our example, the subrequest to /sub just return a 200 response implicitly created by the echo directive in
  location /sub.
  
  Such bad side effects
  make many 3rd-party modules like ngx_echo, ngx_lua and ngx_srcache choose to **Disable** the variable sharing behavior for subrequests by default

