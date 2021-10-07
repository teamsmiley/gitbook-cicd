# ingress-nginx

svc를 ingress로 외부에 오픈

## regex

<https://kubernetes.github.io/ingress-nginx/user-guide/ingress-path-matching/>

- priority
  In NGINX, regular expressions follow a first match policy, 그러므로 ingress nginx가 정규식의 길이를 기준으로 역순으로 정렬을 한후 controller에 업데이트합니다.

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress-1
spec:
  rules:
    - host: test.com
      http:
        paths:
          - path: /foo/bar
            backend:
              serviceName: service1
              servicePort: 80
          - path: /foo/bar/
            backend:
              serviceName: service2
              servicePort: 80
```

```yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress-2
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: test.com
      http:
        paths:
          - path: /foo/bar/(.+)
            backend:
              serviceName: service3
              servicePort: 80
```

이 같은 파일이 두개면 아래와 같이 정렬

```conf
location ~* ^/foo/bar/.+ {
  ...
}

location ~* "^/foo/bar/" {
  ...
}

location ~* "^/foo/bar" {
  ...
}
```

- test.com/foo/bar/1 matches ~\* ^/foo/bar/.+ and will go to service 3.
- test.com/foo/bar/ matches ~\* ^/foo/bar/ and will go to service 2.
- test.com/foo/bar matches ~\* ^/foo/bar and will go to service 1.

테스트를 위해 다음과같은 yml을 만들어서 적용하엿다.

```yml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress-1
  namespace: default
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: c4.xgridcolo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: front
                port:
                  name: http
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress-2
  namespace: default
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: c4.xgridcolo.com
      http:
        paths:
          - path: /api(/|$)(.*)/i
            pathType: Prefix
            backend:
              service:
                name: api
                port:
                  name: http
```

결과 생성된 파일이다. 결론부터 이야기하면 ingress에 rewrite를 쓰면 그 인그레스에만 적용이 된다. 아래보면 /api부분에만 rewrite $1 가 적용되어 있다.

```conf
## start server c4.xgridcolo.com
server {
  server_name c4.xgridcolo.com ;

  listen 80  ;
  listen [::]:80  ;
  listen 443  ssl http2 ;
  listen [::]:443  ssl http2 ;

  set $proxy_upstream_name "-";

  ssl_certificate_by_lua_block {
    certificate.call()
  }

  location ~* "^/api(/|$)(.*)" {

    set $namespace      "default";
    set $ingress_name   "test-ingress-2";
    set $service_name   "api";
    set $service_port   "http";
    set $location_path  "/api(/|${literal_dollar})(.*)";
    set $global_rate_limit_exceeding n;

    rewrite_by_lua_block {
      lua_ingress.rewrite({
        force_ssl_redirect = false,
        ssl_redirect = true,
        force_no_ssl_redirect = false,
        preserve_trailing_slash = false,
        use_port_in_redirects = false,
        global_throttle = { namespace = "", limit = 0, window_size = 0, key = { }, ignored_cidrs = { } }
      })
      balancer.rewrite()
      plugins.run()
    }

    ...

    port_in_redirect off;

    set $balancer_ewma_score -1;
    set $proxy_upstream_name "default-api-http";
    set $proxy_host          $proxy_upstream_name;
    set $pass_access_scheme  $scheme;

    set $pass_server_port    $server_port;

    set $best_http_host      $http_host;
    set $pass_port           $pass_server_port;

    set $proxy_alternative_upstream_name "";

    client_max_body_size                    1m;

    proxy_set_header Host                   $best_http_host;

    # Pass the extracted client certificate to the backend

    # Allow websocket connections
    proxy_set_header                        Upgrade           $http_upgrade;

    proxy_set_header                        Connection        $connection_upgrade;

    proxy_set_header X-Request-ID           $req_id;
    proxy_set_header X-Real-IP              $remote_addr;

    proxy_set_header X-Forwarded-For        $remote_addr;

    proxy_set_header X-Forwarded-Host       $best_http_host;
    proxy_set_header X-Forwarded-Port       $pass_port;
    proxy_set_header X-Forwarded-Proto      $pass_access_scheme;
    proxy_set_header X-Forwarded-Scheme     $pass_access_scheme;

    proxy_set_header X-Scheme               $pass_access_scheme;

    # Pass the original X-Forwarded-For
    proxy_set_header X-Original-Forwarded-For $http_x_forwarded_for;

    # mitigate HTTPoxy Vulnerability
    # https://www.nginx.com/blog/mitigating-the-httpoxy-vulnerability-with-nginx/
    proxy_set_header Proxy                  "";
    # Custom headers to proxied server

    proxy_connect_timeout                   5s;
    proxy_send_timeout                      60s;
    proxy_read_timeout                      60s;

    proxy_buffering                         off;
    proxy_buffer_size                       4k;
    proxy_buffers                           4 4k;

    proxy_max_temp_file_size                1024m;

    proxy_request_buffering                 on;
    proxy_http_version                      1.1;

    proxy_cookie_domain                     off;
    proxy_cookie_path                       off;

    # In case of errors try the next upstream server before returning an error
    proxy_next_upstream                     error timeout;
    proxy_next_upstream_timeout             0;
    proxy_next_upstream_tries               3;

    rewrite "(?i)/api(/|$)(.*)" /$1 break;  # <------------여기
    proxy_pass http://upstream_balancer;

    proxy_redirect                          off;

  }

  location ~* "^/" {

    set $namespace      "default";
    set $ingress_name   "test-ingress-1";
    set $service_name   "front";
    set $service_port   "http";
    set $location_path  "/";
    set $global_rate_limit_exceeding n;

    rewrite_by_lua_block {
      lua_ingress.rewrite({
        force_ssl_redirect = false,
        ssl_redirect = true,
        force_no_ssl_redirect = false,
        preserve_trailing_slash = false,
        use_port_in_redirects = false,
        global_throttle = { namespace = "", limit = 0, window_size = 0, key = { }, ignored_cidrs = { } }
      })
      balancer.rewrite()
      plugins.run()
    }

    port_in_redirect off;

    set $balancer_ewma_score -1;
    set $proxy_upstream_name "default-front-http";
    set $proxy_host          $proxy_upstream_name;
    set $pass_access_scheme  $scheme;

    set $pass_server_port    $server_port;

    set $best_http_host      $http_host;
    set $pass_port           $pass_server_port;

    set $proxy_alternative_upstream_name "";

    client_max_body_size                    1m;

    proxy_set_header Host                   $best_http_host;

    # Pass the extracted client certificate to the backend

    # Allow websocket connections
    proxy_set_header                        Upgrade           $http_upgrade;

    proxy_set_header                        Connection        $connection_upgrade;

    proxy_set_header X-Request-ID           $req_id;
    proxy_set_header X-Real-IP              $remote_addr;

    proxy_set_header X-Forwarded-For        $remote_addr;

    proxy_set_header X-Forwarded-Host       $best_http_host;
    proxy_set_header X-Forwarded-Port       $pass_port;
    proxy_set_header X-Forwarded-Proto      $pass_access_scheme;
    proxy_set_header X-Forwarded-Scheme     $pass_access_scheme;

    proxy_set_header X-Scheme               $pass_access_scheme;

    # Pass the original X-Forwarded-For
    proxy_set_header X-Original-Forwarded-For $http_x_forwarded_for;

    # mitigate HTTPoxy Vulnerability
    # https://www.nginx.com/blog/mitigating-the-httpoxy-vulnerability-with-nginx/
    proxy_set_header Proxy                  "";

    # Custom headers to proxied server

    proxy_connect_timeout                   5s;
    proxy_send_timeout                      60s;
    proxy_read_timeout                      60s;

    proxy_buffering                         off;
    proxy_buffer_size                       4k;
    proxy_buffers                           4 4k;

    proxy_max_temp_file_size                1024m;

    proxy_request_buffering                 on;
    proxy_http_version                      1.1;

    proxy_cookie_domain                     off;
    proxy_cookie_path                       off;

    # In case of errors try the next upstream server before returning an error
    proxy_next_upstream                     error timeout;
    proxy_next_upstream_timeout             0;
    proxy_next_upstream_tries               3;

    proxy_pass http://upstream_balancer;

    proxy_redirect                          off;

  }

}
## end server c4.xgridcolo.com
```
