**Assignment**

1: Gateways:
Expose the frontend service of the application using the Istio-ingress gateway.
The host to be used is "onlineboutique.example.com" for the Ingress gateway. Any other host's requests should be rejected by the gateway.

Applied below manifests to the cluster

To create Gateway

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
     name: public-gateway
spec:
   selector:
       istio: ingressgateway
   servers:
       - hosts:
           - onlineboutique.example.com
         port:
              name: http2
              number: 80
              protocol: HTTP

```
To create Virtual Service

```


apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
     name: frontend-virtual-service
spec:
  gateways:
      - public-gateway
  hosts:
      - onlineboutique.example.com
  http:
  - match:
        - uri: 
            prefix: /  
    route:
    - destination:
        host: frontend
        port:
          number: 80

```
Test:

```
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s -o /dev/null -v
*   Trying 172.19.255.200:80...
* TCP_NODELAY set
* Connected to onlineboutique.example.com (172.19.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=db51f72b-0529-4220-8852-0ab00b20bc6b; Max-Age=172800
< date: Wed, 12 Oct 2022 05:57:19 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 69
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6978 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$

```

2: Traffic routing:
Split the traffic between the frontend and frontend-v2 service by 50%.
The way to verify that this works is when 50% of the requests would show the landing page banner as "Free shipping with $100 purchase!" vs "Free shipping with $75 purchase!"

DestinationRule

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
      name: frontend-destination-rule
spec:
  host: frontend
  subsets:
      - name: v1
        labels:
           version: v1
      - name: v2
        labels:
            version: v2
```
Split traffic VirtualService modification:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
     name: frontend-virtual-service
spec:
  gateways:
      - public-gateway
  hosts:
      - onlineboutique.example.com
  http:
    - match:
        - uri: 
            prefix: /
      route:
         - destination:
               host: frontend
               subset: v1
           weight: 50
         - destination:
               host: frontend
               subset: v2
           weight: 50
```
Test:

```
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ 

```
3: Traffic Routing:
Route traffic to the based on the browser being used.
When you use Firefox the Gateway routes to the frontend service whereas it routes to the frontend-v2 pods if it is accessed via Chrome.
Hint: use the user-agent HTTP header added by the browser.

Modified virtual service to route based on browser

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
     name: frontend-virtual-service
spec:
  gateways:
      - public-gateway
  hosts:
      - onlineboutique.example.com
  http:
    - match:
         - headers:
              user-agent:
                    regex: '.*Chrome.*$'
      route:
          - destination:
                host: frontend
                subset: v1
    - match:
         - headers:
             user-agent:
                   regex: '.*Firefox.*$'
      route:
          - destination:
              host: frontend
              subset: v2
    - match:
        - uri: 
            prefix: /
      route:
         - destination:
               host: frontend
               subset: v1
           weight: 50
         - destination:
               host: frontend
               subset: v2
           weight: 50
```
Test:

FireFox:
```
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
```
Chrome:

```      
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i" http://onlineboutique.example.com -s |grep -i "free shipping"
                <div class="h-free-shipping">Free shipping with $75 purchase!</div>

```

4. Timeout:
This is a slightly different lab. You need to tighten the boundaries of acceptable latency in this lab.
Delete the productcatalogservice. There is a lot of latency between the frontend and the productcatalogv2 service. add a timeout of 3s. (You need to produce a 504 Gateway timeout error).

Created product-svc virtual service for productcatalogservice and added fault injection

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
     name: productcatalog-virtual-service
spec:
   hosts:
     - productcatalogservice
   http:
     - fault:
           delay:
             fixedDelay: 5s
             percentage:
                     value: 50
       route:
          - destination:
                host: productcatalogservice
                port:
                  number: 3550

```

Added timeout in frontend Vistual service

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
     name: frontend-virtual-service
spec:
  gateways:
      - public-gateway
  hosts:
      - onlineboutique.example.com
  http:
    - match:
         - headers:
              user-agent:
                    regex: '.*Chrome.*$'
      route:
          - destination:
                host: frontend
                subset: v1
      timeout: 3s
    - match:
         - headers:
             user-agent:
                   regex: '.*Firefox.*$'
      route:
          - destination:
              host: frontend
              subset: v2
      timeout: 3s
    - match:
        - uri: 
            prefix: /
      route:
         - destination:
               host: frontend
               subset: v1
           weight: 50
         - destination:
               host: frontend
               subset: v2
           weight: 50
      timeout: 3s
```

Test:

```
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s -o /dev/null -v
*   Trying 172.19.255.200:80...
* TCP_NODELAY set
* Connected to onlineboutique.example.com (172.19.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 504 Gateway Timeout
< content-length: 24
< content-type: text/plain
< date: Wed, 12 Oct 2022 06:10:05 GMT
< server: istio-envoy
< 
{ [24 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s -o /dev/null -v
*   Trying 172.19.255.200:80...
* TCP_NODELAY set
* Connected to onlineboutique.example.com (172.19.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 504 Gateway Timeout
< content-length: 24
< content-type: text/plain
< date: Wed, 12 Oct 2022 06:10:14 GMT
< server: istio-envoy
< 
{ [24 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s -o /dev/null -v
*   Trying 172.19.255.200:80...
* TCP_NODELAY set
* Connected to onlineboutique.example.com (172.19.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=c5268502-d960-49b3-8ce6-df7c049dd0c7; Max-Age=172800
< date: Wed, 12 Oct 2022 06:10:16 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 62
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [3885 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl http://onlineboutique.example.com -s -o /dev/null -v
*   Trying 172.19.255.200:80...
* TCP_NODELAY set
* Connected to onlineboutique.example.com (172.19.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=648984e2-ef42-4303-a875-fe7b06680112; Max-Age=172800
< date: Wed, 12 Oct 2022 06:10:19 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 78
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6978 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$

```

5. TLS:
Setup a TLS ingress gateway for the frontend service. Generate self signed certificates and add them to the Ingress Gateway for TLS communication.

Generated ca, csr, cert, key using below openssl commands

```

openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt

openssl req -out onlineboutique.example.com.csr -newkey rsa:2048 -nodes -keyout onlineboutique.example.com.key -subj "/CN=onlineboutique.example.com/O=httpbin organization"

openssl x509 -req -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in onlineboutique.example.com.csr -out onlineboutique.example.com.crt

```
Created kubernetes secrets object for the cert and key

```
kubectl create -n istio-system secret tls gateway-tls-creds --key=onlineboutique.example.com.key --cert=onlineboutique.example.com.crt

```

Modfied gateway to use ssl

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
     name: public-gateway
spec:
   selector:
       istio: ingressgateway
   servers:
       - hosts:
           - onlineboutique.example.com
         port:
              name: https
              number: 443
              protocol: HTTPS
         tls:
           mode: SIMPLE
           credentialName: gateway-tls-creds
    
```
Test:

```
shehbaz@PF33M0G2:~/work/online-boutique/release/assignment$ curl --insecure https://onlineboutique.example.com -s -o /dev/null -v
*   Trying 172.19.255.200:443...
* TCP_NODELAY set
* Connected to onlineboutique.example.com (172.19.255.200) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
} [5 bytes data]
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
} [512 bytes data]
* TLSv1.3 (IN), TLS handshake, Server hello (2):
{ [122 bytes data]
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
{ [15 bytes data]
* TLSv1.3 (IN), TLS handshake, Certificate (11):
{ [762 bytes data]
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
{ [264 bytes data]
* TLSv1.3 (IN), TLS handshake, Finished (20):
{ [52 bytes data]
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.3 (OUT), TLS handshake, Finished (20):
} [52 bytes data]
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=onlineboutique.example.com; O=example.com
*  start date: Oct 11 12:18:13 2022 GMT
*  expire date: Oct 11 12:18:13 2023 GMT
*  issuer: CN=example.com; O=selfCA
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
} [5 bytes data]
* Using Stream ID: 1 (easy handle 0x5567db6372f0)
} [5 bytes data]
> GET / HTTP/2
> Host: onlineboutique.example.com
> user-agent: curl/7.68.0
> accept: */*
> 
{ [5 bytes data]
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
{ [230 bytes data]
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
{ [230 bytes data]
* old SSL session ID is stale, removing
{ [5 bytes data]
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
} [5 bytes data]
< HTTP/2 200 
< set-cookie: shop_session-id=ae2607ff-e30b-431c-91c6-eaa4a437b59e; Max-Age=172800
< date: Wed, 12 Oct 2022 06:15:18 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 67
< server: istio-envoy
< 
{ [7960 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
```
