**Assignment**

1: Gateways:
Expose the frontend service of the application using the Istio-ingress gateway.
The host to be used is "onlineboutique.example.com" for the Ingress gateway. Any other host's requests should be rejected by the gateway.

Applied below manifests to the cluster

```
To create Gateway

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

To create Virtual Service

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

Test:

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

```
DestinationRule

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

Split traffic VirtualService modification:

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
     

Test:

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

```
Modified virtual service to route based on browser

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

Test:

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
*   Trying 172.18.255.200:80...
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#1)
> GET / HTTP/1.1
> Host: 172.18.255.200
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Thu, 20 Jan 2022 11:13:13 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #1 to host 172.18.255.200 left intact

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
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#1)
> GET / HTTP/1.1
> Host: 172.18.255.200
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:96.0) Gecko/20100101 Firefox/96.0
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Thu, 20 Jan 2022 11:13:15 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #1 to host 172.18.255.200 left intact

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
*   Trying 172.18.255.200:80...
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#1)
> GET / HTTP/1.1
> Host: 172.18.255.200
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Thu, 20 Jan 2022 11:13:26 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #1 to host 172.18.255.200 left intact

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
*   Trying 172.18.255.200:80...
* Connected to 172.18.255.200 (172.18.255.200) port 80 (#1)
> GET / HTTP/1.1
> Host: 172.18.255.200
> Accept: */*
> User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36i
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Thu, 20 Jan 2022 11:13:30 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #1 to host 172.18.255.200 left intact

