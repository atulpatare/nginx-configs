## NGINX Configs
Basic configurations for nginx reverseproxy, caching, rate limiting, request rewrites, etc.


### Proxy
`proxy_pass` directive is used to redirect requests to other server, nginx config or internal
application server with supported protocol which includes FastCGI, uwsgi, SCGI, and memcached.

Example
```
location /redirect_on {
	proxy_pass http://somelink;
}
```

```
location /app {
	proxy_pass http://localhost:4000;
}
```

- Update or add custom headers
```
location /some/path {
	proxy_set_header Host $host;
	proxy_pass http://locahost:4000;
}
```

### Service static content
Serving static content can we helpful when you have an static app or website (React app), 
images, files, etc.

`root` directive is used to specify the location for the files, which can be used in
`http{}` , `server{}` , `location{}` contexts. 

```
server {
	root /www/data;

	location / {

	}

	location /images/ {

	}
}

```
In the above example, the server will try to find matching files in `/www/data` directory.


```
location / {
	root /data;
	index index.html 
}
```
if the URI in a request is `/path/`,  `/data/path/index.html` will be responded

`try_files` directive is used to try alternatives for requested resources
```
server {
	root /www/data;

	location /images/ {
		try_files $uri /images/default.gif =404; # if all options fail, return 404
	}
}
```


### Url rewriting
Useful when you want to redirect, or just change the uri as per choice

- redirecting to a domain

```
server {
	listen 80;
	hostname insider.com;
	rewrite ^ https://newinsider.com$request_uri? permanent;
}
```

- redirect from http to https

```
server {
	listen 80 default_server;
	server_name _;
	return 301 https://$host$request_uri;
}
```

- rewriting the url for reverse proxy
```
location /api {
	rewrite ^/api/(.*) /$1 break;
	proxy_set_header Host $host;
	proxy_pass http://backend_server;
}
```


## References
[Doc](https://docs.nginx.com/nginx/)
