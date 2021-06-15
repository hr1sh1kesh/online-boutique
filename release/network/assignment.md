**exercise - 1 :** 
```
Istio exercises
1) Gateways:
    Expose the frontend service of the application using the Istio-ingress gateway
	The host to be used is "onlineboutique.example.com" for the Ingress gateway. Any other host's requests should be rejected by the gateway.
```

Apply the configs of `onlineboutique-gateway.yaml` , `frontend.yaml` and `destination-frontend.yaml`

```bash
gateway.networking.istio.io/onlineboutique-gateway created
virtualservice.networking.istio.io/online-boutique-route created
destinationrule.networking.istio.io/frontend-rule created
```

Created an Ingress gateway to allow only "onlineboutique.example.com" hosts on port 80 for HTTP traffic.

Created a virtual service "online-boutique-route" to allow "onlineboutique.example.com" host to forward to frontend hosts. Apply this VS on the gateway "onlineboutique-gateway"

Created a Destination Rule to include the subsets v1 and v2 of frontend services. Currently, the traffic is forwarded to v1,v2 frontend services in round robin.

Verify : 

```yaml
curl -H "Host: onlineboutique.example.com" http://172.18.0.10/ -v
```

Check the working of gateway and forwarding to frontend services by specifying the Host using the -H flag in curl command

Working: `HTTP/1.1 200 OK`

```bash
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $ curl -H "Host: onlineboutique.example.com" http://172.18.0.10/ -v
*   Trying 172.18.0.10:80...
* TCP_NODELAY set
* Connected to 172.18.0.10 (172.18.0.10) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=2aedf6c2-222e-47a2-afa4-fce4d8609771; Max-Age=172800
< date: Tue, 15 Jun 2021 09:23:08 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 5167
< server: istio-envoy
< transfer-encoding: chunked
```

Changed the host. Error : `HTTP/1.1 404 Not Found`

```bash
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $ curl -H "Host: myonlineboutique" http://172.18.0.10/ -v                                     
*   Trying 172.18.0.10:80...
* TCP_NODELAY set
* Connected to 172.18.0.10 (172.18.0.10) port 80 (#0)
> GET / HTTP/1.1
> Host: myonlineboutique
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Tue, 15 Jun 2021 09:24:39 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #0 to host 172.18.0.10 left intact
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $ curl http://172.18.0.10/ -v       
*   Trying 172.18.0.10:80...
* TCP_NODELAY set
* Connected to 172.18.0.10 (172.18.0.10) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.18.0.10
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Tue, 15 Jun 2021 09:25:03 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #0 to host 172.18.0.10 left intact
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $
```

**exercise - 2:**

```bash
2) Traffic routing 
    Split the traffic between the frontend and frontend-v2 service by 50%. 
	The way to verify that this works is when 50% of the requests would show the landing page banner as "Free shipping with $100 purchase!" vs "Free shipping with $75 purchase!"
```

Using the configs of `frontend.yaml` , the traffic is forwarded to v1,v2 frontend services in round robin.

We can also set weight-based routing by creating another virtual service using `frontend-weighted.yaml` : Delete the configs of `frontend.yaml` and apply `frontend-weighted.yaml`

```bash
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: frontend-weighted
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
for ((i=0; i<2; i++)); do curl -s -H "Host: onlineboutique.example.com" http://172.18.0.10/ | grep -A 2 "free-shipping"; done
```

```bash
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $ k get virtualservice
NAME                GATEWAYS                     HOSTS                            AGE
frontend-weighted   ["onlineboutique-gateway"]   ["onlineboutique.example.com"]   8m59s
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $
```

```bash
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $ for ((i=0; i<2; i++)); do curl -s -H "Host: onlineboutique.example.com" http://172.18.0.10/ | grep -A 2 "free-shipping"; done

                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>

                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>

amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $
```

Full output:

```bash
amodi@infracloud ~/Projects/istio-assignment/online-boutique/release (master*?) $ for ((i=0; i<2; i++)); do curl -s -H "Host: onlineboutique.example.com" http://172.18.0.10/ -v | grep -A 2 "free-shipping"; done 
*   Trying 172.18.0.10:80...
* TCP_NODELAY set
* Connected to 172.18.0.10 (172.18.0.10) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=6dfd45a4-05e1-47f9-bf88-cdfda13ce7ed; Max-Age=172800
< date: Tue, 15 Jun 2021 09:54:46 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 232
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6977 bytes data]
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>

* Connection #0 to host 172.18.0.10 left intact
*   Trying 172.18.0.10:80...
* TCP_NODELAY set
* Connected to 172.18.0.10 (172.18.0.10) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=6173b14b-2f6c-42f1-90c8-9028173a7cc7; Max-Age=172800
< date: Tue, 15 Jun 2021 09:54:51 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 5035
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6976 bytes data]
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>

* Connection #0 to host 172.18.0.10 left intact
```

**exercise - 3:**

```bash
3) Traffic Routing
	Route traffic to the based on the browser being used. 
	When you use firefox the Gateway routes to the frontend service whereas it routes to the frontend-v2 pods if it is accessed via Chrome 
	Hint: use the user-agent HTTP header added by the browser
```

Apply the configs of `browser.yaml` which creates a Virtual Service for routing traffic based on user-agent. Firefox routes to v1 frontend service and Chrome routes to v2 frontend service.
