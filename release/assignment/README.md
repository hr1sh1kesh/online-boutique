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
