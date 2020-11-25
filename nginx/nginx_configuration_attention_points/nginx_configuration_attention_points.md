This article focuses on some attention points about using `upstream`, `backup`, and `proxy_next_stream`.  
Below conclusions are from nginx document about [upstream](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails) and [proxy_next_upstream](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream) and practical test.
# :zap: 1. upstream
1.1 `upstream` use `round_robin` defaulty.   
```
upstream test_upstreams{
        server localhost:8090; # work
        server localhost:8091; # work
    }
    
upstream test_upstreams{
        round_robin;
        server localhost:8090; # work
        server localhost:8091; # work
    }
```
1.2 `round_robin` will skip the services which are down.  
```
upstream test_upstreams{
        server localhost:8090; # down, Attention point: totally skip this services when it's down.
        server localhost:8091; # work
    }
```
1.3 `ip_hash` will not skip the services which are down.  
```
upstream test_upstreams{
        ip_hash;
        server localhost:8090; # down, Attention point: may request to this service if hash matched.
        server localhost:8091; # work
    }
```

# :zap: 2. backup
2.1 In the `upstream`, if a service is down or too busy, the request will transfer to `backup`, so `backup` service have extremely low pressure in usual occasion even if configuring `round_robin`.  
```
upstream test_upstreams{
        server localhost:8090; #down
        server localhost:8091 backup; # work and backup make all request to this.
    }
```

```
upstream test_upstreams{
        server localhost:8090; # work, Attention point: all requests to this service in usual occasion.
        server localhost:8091 backup; # work
    }
```

2.2 `backup` will skip the `weight` because `backup` just check the pre service is down or too busy.  
```
upstream test_upstreams{
        server localhost:8090 weight=1; # work, Attention point: all requests to this service in usual occasion.
        server localhost:8091 weight=5 backup; # work
    }
```

2.3 `backup` cannot be used along with `ip_hash`.  
```
# Attention point: configuration valid error
upstream test_upstreams{
        ip_hash;
        server localhost:8090;
        server localhost:8091 backup;
    }
```

# :zap: 3. proxy_next_upstream
3.1 Must configure `error` behind `proxy_next_upstream` like `proxy_next_upstream error`, or if there is a service down, request may receive 502 according to `max_fails` and `fail_timeout`.  
```
server{
        proxy_next_upstream http_404; # not 'proxy_next_upstream error'
        location / {
                # proxy_next_upstream http_404; # not 'proxy_next_upstream error'
                proxy_pass http://test_upstream;
        }
}

test_upstream{
    server localhost:8090; #down, Attention point: sometimes reponse 502
    server localhost:8091; #work
}
```

3.2 In the `upstream`, the `max_fails` default value is 1.  
```
test_upstream{
    server localhost:8090 max_fails=1; # default
    server localhost:8091 max_fails=5; # configure 5
}
```

3.3 And the `fail_timeout` default value is `10` which represents 10 seconds.  
```
test_upstream{
    server localhost:8090 fail_timeout=10; # default
    server localhost:8091 fail_timeout=20; # configure 20 seconds
}
```

3.4 If configure `http_5xx` or `non_idempotent` behind `proxy_next_upstream`, it may produce repeat data causing by repeat requests when the APIs are not transactional and non idempotency.  
`non_idempotent` make `proxy_next_upstream` support post, patch which are non-indepotent.

```
server{
        proxy_next_upstream http_502 non_idempotent;
        location / {
                # proxy_next_upstream http_502 non_idempotent;
                proxy_pass http://test_upstream;
        }
}

test_upstream{
    server localhost:8090; # return 502
    server localhost:8091; # Attention point: repeat to deal with original request even if post because of configuring non_idempotent.
}
```
