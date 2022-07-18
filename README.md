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

### Serving static content

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

if the URI in a request is `/path/`, `/data/path/index.html` will be responded

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

## Caching

Enabling caching for proxy server using directive `proxy_cache_path`. Parameters include
path, keyszone (name of chache and size).

```
http {
	# ...
	proxy_cache_path /data/nginx/cache keys_zone=mycache:10m;
	server {
		proxy_cache mycache;
		location / {
			proxy_pass http://localhost:8000;
		}
	}
}
```

## Rate Limiting

- Limiting the no of connections
  Done using `limit_conn_zone` direction

```
limit_conn_zone <key> zone=<name>:<size>;
limit_conn_zone $binary_remote_addr zone=addr:10m;
```

use the `limit_conn` to apply the limit in ` location{}``server{}``http{} ` contexts

```
location /download {
	limit_conn addr 1;
}
```

- limiting total no of users on the server

```
http {
	limit_conn_zone $server_name zone=servers:10m;

	server {
		limit_conn servers 1000;
	}
}
```

- Limiting the request rate
  Used to prevent DDoS attaks. The method is based on the `leaky bucket` algorithm

```
http {
	# ...
	limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s
}
```

where, - limit_req_zone : directive to setup request limiting - $binary_remote_addr : key for limiting, ip address of client - zone : argument with <name>:<size> - rate : argument with <count>r/[s | m]

```
server {
	location /search {
		limit_req zone=one;
	}
}
```

- handling excessive requests
  excessive requests can be buffered using `burst`

```
location /search {
	limit_req zone=one burst=5; # size of queue is set to 5
	limit_req zone=one burst=5 nodelay; # if delay is not needed
	limit_req zone=one burst=5 delay=3; # after first 3, other reqs will be delayed
}
```

- Limiting the bandwidth
  Useful when the server wants the downloads a files to be of some maximum rate.

```
location /download/ {
	limit_rate 50k; # 50 kilobytes per seconds
}
```

## Whitelisting/Blacklisting

- Using `allow` and `deny` with specified ipaddress/ subnet will either allow or deny
  the requests

```
server {
	allow 192.168.10.2;
	allow 102.168.10.1/24;
	deny all;
}
```

- Limiting a list of ips

```
geo $whitelisted_ip {
	default 	 0;
	192.168.10.11/32 1;
	192.168.15.11/32 1;
}


server {
	if ( $whitelisted_ip = 0 ) {
		return 403;
	}
}
```

## References

[Doc](https://docs.nginx.com/nginx/)
