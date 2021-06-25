**Exercise 1 :**

Que:
Expose the frontend service of the application using the Istio-ingress gateway
The host to be used is "onlineboutique.example.com" for the Ingress gateway. Any other host's requests should be rejected by the gateway. 

Solution:
Apply manifests files 
istio-virtual-service.yaml &  istio-gateway.yaml 

Verification :

```yaml
# Succesful connection with host  onlineboutique.example.com 
 
 
infracloud@Infracloud:~$ curl -H "Host: onlineboutique.example.com" http://172.18.240.2 -v

 Trying 172.18.240.2:80...
* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=4179c8c4-96fc-450b-8cef-ea1e1654ae98; Max-Age=172800
< date: Wed, 23 Jun 2021 05:59:42 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 61
< server: istio-envoy
< transfer-encoding: chunked
< 


# Connection being rejected for hosts other than onlineboutique.example.com


infracloud@Infracloud:~$ curl -H "Host: onlineboutique.example.com1" http://172.18.240.2 -v
*   Trying 172.18.240.2:80...
* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com1
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Wed, 23 Jun 2021 06:01:06 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #0 to host 172.18.240.2 left intact
infracloud@Infracloud:~$ curl -H "Host: example.com1" http://172.18.240.2 -v
*   Trying 172.18.240.2:80...
* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: example.com1
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 404 Not Found
< date: Wed, 23 Jun 2021 06:01:13 GMT
< server: istio-envoy
< content-length: 0
< 
* Connection #0 to host 172.18.240.2 left intact
```



Exercise 2 :

Que:  Split the traffic between the frontend and frontend-v2 service by 50%. 
	    The way to verify that this works is when 50% of the requests would show the landing page banner as "Free shipping with $100 purchase!" vs "Free shipping with $75 purchase!" 

Solution: Apply manifests files  istio-virtual-service-weighted.yaml, istio-gateway.yaml ,  istio-destination-rule.yaml 

Verification :

```yaml

infracloud@Infracloud:~$ for i in {1..4}; do curl -H "Host: onlineboutique.example.com" http://172.18.240.2 -v|grep Free;done > curl_output
*   Trying 172.18.240.2:80...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=f8e630eb-fd5e-458e-8016-3c7d712f88f2; Max-Age=172800
< date: Wed, 23 Jun 2021 07:28:39 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11040
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6975 bytes data]
100 10857    0 10857    0     0    983      0 --:--:--  0:00:11 --:--:--  2245
* Connection #0 to host 172.18.240.2 left intact
*   Trying 172.18.240.2:80...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=5544af1f-579c-48c6-843e-b7780efd7469; Max-Age=172800
< date: Wed, 23 Jun 2021 07:28:50 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11053
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6975 bytes data]
100 10857    0 10857    0     0    982      0 --:--:--  0:00:11 --:--:--  2821
* Connection #0 to host 172.18.240.2 left intact
*   Trying 172.18.240.2:80...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=f06ef4ba-bca5-4273-8483-b65a1e1243ce; Max-Age=172800
< date: Wed, 23 Jun 2021 07:28:50 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 47
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6978 bytes data]
100 10861    0 10861    0     0   216k      0 --:--:-- --:--:-- --:--:--  216k
* Connection #0 to host 172.18.240.2 left intact
*   Trying 172.18.240.2:80...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0* Connected to 172.18.240.2 (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=f2bc318c-d13e-47dc-bbaa-b48f1f01f3d7; Max-Age=172800
< date: Wed, 23 Jun 2021 07:29:01 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11039
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6975 bytes data]
100 10861    0 10861    0     0    983      0 --:--:--  0:00:11 --:--:--  2832
* Connection #0 to host 172.18.240.2 left intact



infracloud@Infracloud:~$ cat curl_output 
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
                <div class="h-free-shipping">Free shipping with $75 purchase! &nbsp;&nbsp;</div>
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
                <div class="h-free-shipping">Free shipping with $100 purchase! &nbsp;&nbsp;</div>
 ```
 
 
Exercise 3 :

Que: 	Route traffic to the based on the browser being used. 
	    When you use firefox the Gateway routes to the frontend service whereas it routes to the frontend-v2 pods if it is accessed via Chrome 
    	(Hint: use the user-agent HTTP header added by the browser)

Solution: Apply manifests files  istio-vsvc-useragent-based-route.yaml , istio-gateway.yaml 

Verification :
```yaml
infracloud@Infracloud:~/online-boutique$ curl -H "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/53790" http://onlineboutique.example.com/ -v|grep Free
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 172.18.240.2:80...
* Connected to onlineboutique.example.com (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/53790
> 
  0     0    0     0    0     0      0      0 --:--:--  0:00:10 --:--:--     0* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=30b27225-913f-462d-b03e-640c54648fcf; Max-Age=172800
< date: Wed, 23 Jun 2021 11:17:02 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11050
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6975 bytes data]
100  6963    0  6963    0     0                 <div class="h-free-shipping">**Free shipping with $100 purchase! **&nbsp;&nbsp;</div>
100 10861    0 10861    0     0    982      0 --:--:--  0:00:11 --:--:--  2823
* Connection #0 to host onlineboutique.example.com left intact



infracloud@Infracloud:~/online-boutique$ curl -H "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Firefox/88.0.4324.182 Safari/53790" http://onlineboutique.example.com/ -v|grep Free
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 172.18.240.2:80...
* Connected to onlineboutique.example.com (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> Accept: */*
> user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Firefox/88.0.4324.182 Safari/53790
> 
  0     0    0     0    0     0      0      0 --:--:--  0:00:11 --:--:--     0* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< set-cookie: shop_session-id=bc78b9d9-5dfe-40b3-bd40-aaf612adcdfa; Max-Age=172800
< date: Wed, 23 Jun 2021 11:17:59 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 11679
< server: istio-envoy
< transfer-encoding: chunked
< 
{ [6975 bytes data]
                <div class="h-free-shipping">**Free shipping with $75 purchase!** &nbsp;&nbsp;</div>
100 10857    0 10857    0     0    929      0 --:--:--  0:00:11 --:--:--  2427
* Connection #0 to host onlineboutique.example.com left intact
```


Exercise 4 :

Que: This is a slightly different lab. You need to tighten the boundaries of acceptable latency in this lab.  
     Delete the `productcatalogservice`. There is a lot of latency between the frontend and the productcatalogv2 service. add a timeout of 3s. (You need to produce a 504 Gateway timeout error)

Solution: Apply manifests files  istio-vsvc-useragent-based-route.yaml , istio-gateway.yaml

Verification :

```yaml

infracloud@Infracloud:~/online-boutique$ curl   http://onlineboutique.example.com/ -v
*   Trying 172.18.240.2:80...
* Connected to onlineboutique.example.com (172.18.240.2) port 80 (#0)
> GET / HTTP/1.1
> Host: onlineboutique.example.com
> User-Agent: curl/7.74.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 504 Gateway Timeout
< content-length: 24
< content-type: text/plain
< date: Wed, 23 Jun 2021 11:22:08 GMT
< server: istio-envoy
< 
```

Exercise 5 :

Que : Setup a TLS ingress gateway for the frontend service. Generate self signed certificates and add them to the Ingress Gateway for TLS communication.

Solution : 
         1) Generate Self signed root ca and certs file using below commands 
            (Reference : https://istio.io/latest/docs/tasks/traffic-management/ingress/secure-ingress/)
            i) openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt
          
            ii) openssl req -out onlineboutique.example.com.csr -newkey rsa:2048 -nodes -keyout onlineboutique.example.com.key -subj "/CN=onlineboutique.example.com/O=Infracloud"
                   
          iii) openssl x509 -req -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in onlineboutique.example.com.csr -out onlineboutique.example.com.crt
	  
	 2) Apply Manifests istio-secure-virtual-service.yaml & istio-secure-gateway.yaml
          
Verification :
```yaml
infracloud@Infracloud:~/online-boutique$ curl -HHost:onlineboutique.example.com --resolve "onlineboutique.example.com:443:172.18.240.2" --cacert ../example.com.crt "https://onlineboutique.example.com:443" -v
* Added onlineboutique.example.com:443:172.18.240.2 to DNS cache
* Hostname onlineboutique.example.com was found in DNS cache
*   Trying 172.18.240.2:443...
* Connected to onlineboutique.example.com (172.18.240.2) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: ../example.com.crt
*  CApath: /etc/ssl/certs
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
*  subject: CN=onlineboutique.example.com; O=Infracloud
*  start date: Jun 24 08:53:30 2021 GMT
*  expire date: Jun 24 08:53:30 2022 GMT
*  common name: onlineboutique.example.com (matched)
*  issuer: O=example Inc.; CN=example.com
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x561f07ac2580)
> GET / HTTP/2
> Host:onlineboutique.example.com
> user-agent: curl/7.74.0
> accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
< HTTP/2 200 
< set-cookie: shop_session-id=897b1699-9fb8-4684-bf8f-6403d2646e46; Max-Age=172800
< date: Thu, 24 Jun 2021 09:55:51 GMT
< content-type: text/html; charset=utf-8
< x-envoy-upstream-service-time: 39
< server: istio-envoy
< 
```
