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
  name: onlineboutique
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "onlineboutique.example.com"

```
To create Virtual Service

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: onlineboutique
spec:
  hosts:
  - "onlineboutique.example.com"
  gateways:
  - onlineboutique
  http:
  - route:
    - destination:
        host: frontend
        port:
          number: 80

```
Test:

```
sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -H "HOST: onlineboutique.example.com" http://172.18.255.200 -v
*   Trying 172.18.255.200:80...
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=e6515d8e-a835-4a8e-8ff6-3c9401cbeeab; Max-Age=172800
< date: Thu, 20 Jan 2022 09:39:09 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11061
< server: istio-envoy
< transfer-encoding: chunked
< 

<!DOCTYPE html>
<html lang="en">

<head>

```

2: Traffic routing:
Split the traffic between the frontend and frontend-v2 service by 50%.
The way to verify that this works is when 50% of the requests would show the landing page banner as "Free shipping with $100 purchase!" vs "Free shipping with $75 purchase!"

DestinationRule

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-frontend
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
  name: onlineboutique
spec:
  hosts:
  - "onlineboutique.example.com"
  gateways:
  - onlineboutique
  http:
  - route:
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
sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "HOST: onlineboutique.example.com" http://172.18.255.200 -v | grep -i 'free shipping'
*   Trying 172.18.255.200:80...
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=3b3bfa72-b450-4e9a-9c92-a688f43c9163; Max-Age=172800
< date: Thu, 20 Jan 2022 11:00:16 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 36
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [7968 bytes data]
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host 172.18.255.200 left intact
sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "HOST: onlineboutique.example.com" http://172.18.255.200 -v | grep -i 'free shipping'
*   Trying 172.18.255.200:80...
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=cf585c91-6a4d-48d7-90f2-31e834b1366c; Max-Age=172800
< date: Thu, 20 Jan 2022 11:00:17 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 36
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [7968 bytes data]
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host 172.18.255.200 left intact

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
  name: onlineboutique
spec:
  hosts:
  - "onlineboutique.example.com"
  gateways:
  - onlineboutique
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Firefox/.*"
    route:
    - destination:
        host: frontend
        subset: v1
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome/.*"
    route:      
    - destination:
        host: frontend
        subset: v2
```
Test:

```
sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" HOST: "onlineboutique.example.com" http://172.18.255.200 -v | grep -i 'free shipping'
* Closing connection -1
*   Trying 172.18.255.200:80...
* Connected to onlineboutique.example.com (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=9867100d-9e7a-474f-b102-da480870f4fc; Max-Age=172800
< date: Thu, 20 Jan 2022 11:13:13 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11061
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [7968 bytes data]
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host onlineboutique.example.com left intact


sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" HOST: "onlineboutique.example.com" http://172.18.255.200 -v | grep -i 'free shipping'
* Closing connection -1
*   Trying 172.18.255.200:80...
* Connected to onlineboutique.example.com (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=32587ae3-280c-49b7-89a2-50b554e52672; Max-Age=172800
< date: Thu, 20 Jan 2022 11:13:16 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 29
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [10877 bytes data]
* Connection #0 to host onlineboutique.example.com left intact
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
*   Trying 172.18.255.200:80...

sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i" HOST: "onlineboutique.example.com" http://172.18.255.200 -v | grep -i 'free shipping'
* Closing connection -1
*   Trying 172.18.255.200:80...
* Connected to onlineboutique.example.com (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=c05df26e-5dee-4f07-9ec9-49c27a1cf15c; Max-Age=172800
< date: Thu, 20 Jan 2022 11:13:27 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 50
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6978 bytes data]
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host onlineboutique.example.com left intact

sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i" HOST: "onlineboutique.example.com" http://172.18.255.200 -v | grep -i 'free shipping'
* Closing connection -1
*   Trying 172.18.255.200:80...
* Connected to onlineboutique.example.com (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=abb42ca7-449f-44b3-9681-ff2de2a807f6; Max-Age=172800
< date: Thu, 20 Jan 2022 11:13:30 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 32
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [3885 bytes data]
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host onlineboutique.example.com left intact

```

4. Timeout:
This is a slightly different lab. You need to tighten the boundaries of acceptable latency in this lab.
Delete the productcatalogservice. There is a lot of latency between the frontend and the productcatalogv2 service. add a timeout of 3s. (You need to produce a 504 Gateway timeout error).

Created product-svc virtual service for productcatalogservice and added fault injection

```

apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: product-svc
spec:
  hosts:
  - productcatalogservice
  http:
  - route:
    - destination:
        host: productcatalogservice
    fault:
      delay:
        fixedDelay: 5s
        percent: 100
```

Added timeout in frontend Vistual service

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: onlineboutique
spec:
  hosts:
  - "onlineboutique.example.com"
  gateways:
  - onlineboutique
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Firefox/.*"
    route:
    - destination:
        host: frontend
        subset: v1
    timeout: 3s    
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome/.*"
    route:      
    - destination:
        host: frontend
        subset: v2
    timeout: 3s

```

Test:

```
sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i" HOST: "onlineboutique.example.com" http://172.18.255.200 -v
* Closing connection -1
*   Trying 172.18.255.200:80...
* Connected to onlineboutique.example.com (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 504 Gateway Timeout
< content-length: 24
< content-type: text/plain
< date: Thu, 20 Jan 2022 11:57:42 GMT
< server: istio-envoy

sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/assignment$ curl -s -H "User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0" HOST: "onlineboutique.example.com" http://172.18.255.200 -v
* Closing connection -1
*   Trying 172.18.255.200:80...
* Connected to onlineboutique.example.com (172.18.255.200) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 504 Gateway Timeout
< content-length: 24
< content-type: text/plain
< date: Thu, 20 Jan 2022 12:04:59 GMT
< server: istio-envoy

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
kubectl create -n istio-system secret tls onlineboutique-credential --key=onlineboutique.example.com.key --cert=onlineboutique.example.com.crt

```

Modfied gateway to use ssl

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: onlineboutique
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - "onlineboutique.example.com"
    tls:
      mode: SIMPLE
      credentialName: onlineboutique-credential
    
```
Modfied Virtual service for curl test

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: onlineboutique
spec:
  hosts:
  - "onlineboutique.example.com"
  gateways:
  - onlineboutique
  http:
  - match:
    - headers:
        user-agent:
          regex: ".*Firefox/.*"
    route:
    - destination:
        host: frontend
        subset: v1
    timeout: 3s    
  - match:
    - headers:
        user-agent:
          regex: ".*Chrome/.*"
    route:      
    - destination:
        host: frontend
        subset: v2
    timeout: 3s
  - match:
    - headers:
        user-agent:
          regex: ".*curl/.*"
    route:      
    - destination:
        host: frontend
        subset: v1

```
Test:

```
sunil@SUNIL-PM:~/Documents/local-k8s/online-boutique/release/certs$ curl -vs --cacert example.com.crt  https://onlineboutique.example.com | grep -i 'free shipping'
*   Trying 172.18.255.200:443...
* Connected to onlineboutique.example.com (172.18.255.200) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: example.com.crt
*  CApath: /etc/ssl/certs
} [5 bytes data]
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
} [512 bytes data]
* TLSv1.3 (IN), TLS handshake, Server hello (2):
{ [122 bytes data]
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
{ [15 bytes data]
* TLSv1.3 (IN), TLS handshake, Certificate (11):
{ [758 bytes data]
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
*  subject: CN=onlineboutique.example.com; O=httpbin organization
*  start date: Jan 20 07:42:51 2022 GMT
*  expire date: Jan 20 07:42:51 2023 GMT
*  common name: onlineboutique.example.com (matched)
*  issuer: O=example Inc.; CN=example.com
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
} [5 bytes data]
* Using Stream ID: 1 (easy handle 0x5601b9811580)
} [5 bytes data]
> GET / HTTP/2
> Host: onlineboutique.example.com
> user-agent: curl/7.74.0
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
< set-cookie: shop_session-id=53b69541-d8cb-47f4-b641-7cc4333e6371; Max-Age=172800
< date: Thu, 20 Jan 2022 12:37:17 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 5058
< server: istio-envoy
< 
{ [7960 bytes data]
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host onlineboutique.example.com left intact
```
