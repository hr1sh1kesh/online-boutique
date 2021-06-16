**exercise - 1 :** 
```
Istio exercises
1) Gateways:
    Expose the frontend service of the application using the Istio-ingress gateway
	The host to be used is "onlineboutique.example.com" for the Ingress gateway. Any other host's requests should be rejected by the gateway.
```

Apply the configs of `istio-manifest.yaml` 


```bash
gateway.networking.istio.io/onlineboutique-gateway created
virtualservice.networking.istio.io/frontend-vs created
destinationrule.networking.istio.io/frontend-rule created
```


This manifest will create VirtualService, Gateway and DestinationRule

Verify : 

```yaml
curl -H "Host: onlineboutique.example.com" http://172.18.0.40/ -v
```

```
**exercise - 2:**
```bash
2) Traffic routing 
    Split the traffic between the frontend and frontend-v2 service by 50%. 
	The way to verify that this works is when 50% of the requests would show the landing page banner as "Free shipping with $100 purchase!" vs "Free shipping with $75 purchase!"
```
Using the same manifest file "instio-manifest.yaml" created the VS, DR and gateway. The code for VirtualService is as given below:

```bash
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: frontend-vs
spec:
  hosts:
  - "onlineboutique.example.com"
  gateways:
  - onlineboutique-gateway
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
Verified : 
```bash
for ((i=0; i<2; i++)); do curl -s -H "Host: onlineboutique.example.com" http://172.18.0.40/ | grep -A 2 "free-shipping"; done
```
```bash
NAME                GATEWAYS                     HOSTS                            AGE
frontend-vs   ["onlineboutique-gateway"]   ["onlineboutique.example.com"]   8m59s

```
```bash
infracloud@Sumedh:~/online-boutique/release$ for ((i=0; i<2; i++)); do curl -s -H "Host: onlineboutique.example.com" http://172.18.0.40/ | grep -A 2 "free-shipping"; done
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
```
Full output:
```bash
infracloud@Sumedh:~/online-boutique/release$ for ((i=0; i<2; i++)); do curl -s -H "Host: onlineboutique.example.com" http://172.18.0.40/ -v | grep -A 2 "free-shipping"; done 
*   Trying 172.18.0.40:80...
* TCP_NODELAY set
* Connected to 172.18.0.40 (172.18.0.40) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=6dfd45a4-05e1-47f9-bf88-cdfda13ce7ed; Max-Age=172800
< date: Tue, 16 Jun 2021 12:17:46 IST
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 232
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6977 bytes data]
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host 172.18.0.40 left intact
*   Trying 172.18.0.40:80...
* TCP_NODELAY set
* Connected to 172.18.0.40 (172.18.0.40) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=6173b14b-2f6c-42f1-90c8-9028173a7cc7; Max-Age=172800
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 5035
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6976 bytes data]
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
* Connection #0 to host 172.18.0.40 left intact
```
**exercise - 3:**
```bash
3) Traffic Routing
	Route traffic to the based on the browser being used. 
	When you use firefox the Gateway routes to the frontend service whereas it routes to the frontend-v2 pods if it is accessed via Chrome 
	Hint: use the user-agent HTTP header added by the browser
```
Apply The manifest file `routing-header.yaml` which create the VS which routes the traffic as per the browser used.
Version 1 service is routed to Mozilla firefox while version 2 service routed to chrome browser.
**exercise - 4:**
```bash
4) Timeout
    This is a slightly different lab. You need to tighten the boundaries of acceptable latency in this lab.  
    Delete the `productcatalogservice`. There is a lot of latency between the frontend and the productcatalogv2 service. add a timeout of 3s. (You need to produce a 504 Gateway timeout error)
```
Allpy the manifest `timeout_routing.yaml`. We deleted the service productcatalougeservice v1 and as a result all the traffic routed to the productcatalogservice:v2 (OR we can diver all the traffic from v1 to v2 using Kiali dashboard) and added the timeout of 3 sec. Received the following error: 
Received error:
```bash
HTTP Status: 500 Internal Server Error
rpc error: code = Unavailable desc = upstream request timeout
could not retrieve products
main.(*frontendServer).homeHandler
	/src/handlers.go:63
net/http.HandlerFunc.ServeHTTP
	/usr/local/go/src/net/http/server.go:2042
github.com/gorilla/mux.(*Router).ServeHTTP
	/go/pkg/mod/github.com/gorilla/mux@v1.7.3/mux.go:212
main.(*logHandler).ServeHTTP
	/src/middleware.go:81
main.ensureSessionID.func1
	/src/middleware.go:103
net/http.HandlerFunc.ServeHTTP
	/usr/local/go/src/net/http/server.go:2042
go.opencensus.io/plugin/ochttp.(*Handler).ServeHTTP
	/go/pkg/mod/go.opencensus.io@v0.21.0/plugin/ochttp/server.go:86
net/http.serverHandler.ServeHTTP
	/usr/local/go/src/net/http/server.go:2843
net/http.(*conn).serve
	/usr/local/go/src/net/http/server.go:1925
runtime.goexit
	/usr/local/go/src/runtime/asm_amd64.s:1374
```
**exercise - 5**
```bash
5) TLS 
	Setup a TLS ingress gateway for the frontend service. Generate self signed certificates and add them to the Ingress Gateway for TLS communication.
```
Create a secret for Ingress-Gateway before applying the `istio_routing.yaml` manifest. Apply the manifest `tls_routing.yaml` to create the VS, DR and Gateway
Follow the steps from the documentation [https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/#configure-a-tls-ingress-gateway-for-a-single-host](https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/#configure-a-tls-ingress-gateway-for-a-single-host) to Generate client and server certificates. 
```bash
-----
Signature ok
subject=CN = frontend.example.com, O = frontend organization
Getting CA Private Key
amodi@infracloud ~/Desktop/network/certs $ ll
total 20K
-rw-rw-r-- 1 amodi amodi 1.1K Jun 15 17:06 frontend.example.com.crt
-rw-rw-r-- 1 amodi amodi  948 Jun 15 17:06 frontend.example.com.csr
-rw------- 1 amodi amodi 1.7K Jun 15 17:06 frontend.example.com.key
-rw-rw-r-- 1 amodi amodi 1.2K Jun 15 17:06 onlineboutique.com.crt
-rw------- 1 amodi amodi 1.7K Jun 15 17:06 onlineboutique.com.key
amodi@infracloud ~/Desktop/network/certs $ kubectl create -n istio-system secret tls frontend-credential --key=frontend.example.com.key --cert=frontend.example.com.crt
secret/frontend-credential created
amodi@infracloud ~/Desktop/network/certs $
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release/network (istio-assignment*?) $ k apply -f secure-gateway.yaml
gateway.networking.istio.io/onlineboutique-gateway configured
```
Verified connection : 
```bash
infracloud@Sumedh:~/online-boutique/release$ curl -HHost:onlineboutique.example.com --resolve "frontend.example.com:443:172.18.0.40" --cacert ~/certs/onlineboutique.com.crt "https://frontend.example.com:443" -v -s
* Added frontend.example.com:443:172.18.0.40 to DNS cache
* Hostname frontend.example.com was found in DNS cache
*   Trying 172.18.0.40:443...
* TCP_NODELAY set
* Connected to frontend.example.com (172.18.0.40) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: certs/onlineboutique.com.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=frontend.example.com; O=frontend organization
*  start date: Jun 15 11:36:40 2021 GMT
*  expire date: Jun 15 11:36:40 2022 GMT
*  common name: frontend.example.com (matched)
*  issuer: O=onlineboutique Inc.; CN=onlineboutique.com
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55613ca9ae10)
> GET / HTTP/2
> Host:onlineboutique.example.com
> user-agent: curl/7.68.0
> accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
< HTTP/2 404 
< date: Tue, 15 Jun 2021 13:27:44 GMT
< server: istio-envoy
< 
* Connection #0 to host frontend.example.com left intact
```
