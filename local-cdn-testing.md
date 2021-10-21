# Local CDN for app compatibility testing
Atlassian introduced support for serving web resources through a CDN for some of their DC apps (at least for [Confluence](https://confluence.atlassian.com/doc/configure-your-cdn-for-confluence-data-center-976771362.html) and [Jira](https://confluence.atlassian.com/adminjiraserver/configure-your-cdn-for-jira-data-center-974378841.html)).
Usually apps don't need to do anything specifically to support this, but it's still a good idea to test compatibility.

Testing can be done with a 'local CDN' provided using ngrok and docker.

With this setup requests from the browser are routed like this:
```
Browser -> ngrok -> dockerized nginx -> Confluence
```
* ngrok is used to provide SSL, since the CDN feature requires https URLs.
* nginx is used for caching resources, downloading them from Confluence if not already in the cache.


## Establish the ngrok Tunnel
```
ngrok http -subdomain=my-cdn-url -inspect=false 8888
```

## Start Nginx in Docker
You need this nginx config file. Adapt the `server_name` property to match your ngrok tunnel domain and `proxy_pass` to match host and port of your Confluence / Jira instance.

```
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '[$time_local] $status "$request"';
    access_log /dev/stdout main;
    error_log /dev/stdout error;

    sendfile        on;
    keepalive_timeout  65;

    proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=STATIC:10m inactive=24h max_size=1g;

    server {
        server_name my-cdn-url.eu.ngrok.io;

        location / {
            add_header X-Cache $upstream_cache_status;
            proxy_set_header x-forwarded-host $host;
            proxy_ignore_headers Set-Cookie;
            proxy_cache STATIC;
            proxy_cache_valid 200 1h;
            proxy_pass http://docker.for.mac.host.internal:1990;
        }
    }
}
```

Save that file somewhere, adapt server name, then run nginx:
```
docker run -it --rm -p8888:80 -v /path/to/cdn-nginx.conf:/etc/nginx/nginx.conf:ro nginx:1.17-alpine
```
Test if the connection works by accessing `https://<your-ngrok-tunnel-domain/some-thing`. You should see the access logged in the container's output.

## Start Confluence with CDN feature enabled
This is a DC feature, so you need to install a DC Confluence/Jira license to access the configuration UI.

Enable the CDN feature at: `<baseUrl>/plugins/servlet/content-distribution` by inserting your full ngrok tunnel URL, e.g. `https://my-cdn-url.eu.ngrok.io`

Once the feature has been enabled you should see web resources being loaded from your ngrok tunnel domain. Happy testing.
