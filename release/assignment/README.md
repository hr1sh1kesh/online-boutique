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

