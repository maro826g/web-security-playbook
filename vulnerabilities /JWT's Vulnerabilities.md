JSON Web Tokens (JWT)

- used for authentication and authorization and information exchange

consists of three parts (separated by a dot)

- header: metadata about the token (signing algorithm, token type)
- payload: claims (statements) about user or session (user ID, role, expiration)
- signature: ensure token’s integrity by cryptographically  singing the header and payload (is make by hashing the header and payload together)
- JWTs were created to preform a lightweight, stateless and scalable method for authentication and session data
- unlike traditional session-based authentication methods jwt doesn't need server-side storage to manage the session state by embedding session-related data directly into the session
- signature is what stops a hacker from modify the token and sent it by a diffrent role or session ID because if you change them in payload you can't modify them in the signature

```python
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwI
iwibmFtZSI6IkpvZSIsImlhdCI6MTUxNjIzOTAyMn0.SflKxwRJSMeKKF2QT4f
wpMeJf36POk6yJV_adQssw5c
```

header, payload, signature

if you have the secret key and decode them they will look like

header (base64)

{

“alg”: “HS256”,

“typ”: “JWT”

}

payload (base64)

{

“sub”: 122345”,

“name”: “john”,

“admin”: true,

“exp”: 1715

}

signature (symmetric or asymmetric algorithm for encoded header ,payload)

example

HMACSHA256

base64urlEncode(header+ “.” + base64UrlEncode(payload), secret)

)

![image.png](attachment:e068a916-34f0-4ff5-86ad-a36b334f7a2f:image.png)

### The None Algorithm Vulnerability

happens when the signature is not validated efficiently 

what causes the vulnerability?

Misconfiguration in the implementation of JWT libraries 

- some JWT libraries support none algo for testing purposes where token doesn't require a signature
- if improperly configured it may accept tokens it might accept tokens with non algo as valid without verifying

failure to validate signature 

- application fails to enforce the need for a valid cryptographic signature

improper trust in the JWT header

- if the application blindly trusts the value in the header
the attacker can modify the algorithm (HS256,RS256) to none and bypass the signature validation

### **JWT vs JWS vs JWE**

JSON Web Token(JWT): It only defines a format for representing information ("claims") as a JSON object that can be transferred between two parties

JSON Web Signature (JWS): Guarantees **Integrity**. "I promise this data hasn't been changed." (The data is readable to anyone).

JSON Web Encryption (JWE) Guarantees **Confidentiality**. "I promise only the recipient can read this data." (The data is scrambled).

## **Exploiting flawed JWT signature verification

Accepting arbitrary signatures**

first lab
1) go to /admin page it says unauthorized 

2)checked the jwt token decoded the header and the payload (first two partitions)  found the header has the ID and the hashing algorithm (rs256)

 and the payload looks like 

```python
{"iss":"portswigger","exp":1769210945,"sub":"wiener"}
```

3)simply changed the sub from wiener → administrator

encode it one more time to base64 (btw if you found = signs after encoding base64 ignore em they are just for **padding purposes** cuz base64 is fixed size encoding so don't include them in ur token)

4)modify ur token to the new one and sent the request

### **Accepting tokens with no signature**

sometimes the signature is validated properly but you can choose to remove it by modifying the header and if the application lets u able to do this and choose the alg ureslef you can set it to none 
example:

instead of 

```python
{
"alg": "HS256",
"typ": "JWT"
}
```

```python
you make it into 

{
"alg": "none",
"typ": "JWT"
}
```

but then you will have to remove the signature from the token since it has no use as u modified it into none but dont forget to leave the final dot that ends the payload part

second lab

1)hit /admin take the token from the get request 

2)decode the header as base64 it will look like

```python
{"kid":"a7969602-9380-4438-b200-40a62f9a4531","alg":"RS256"}
```

modify it to

```python
{"kid":"a7969602-9380-4438-b200-40a62f9a4531","alg":"none"}
```

3)remove the signature part from the request and navigate to /admin page and delete carlos

### **Brute-forcing secret keys**

some algorithms like `HS256`  use a placeholder string as a secret key, just like a password and it can be an easy word and guessable for testing purposes cuz sometimes developers forget to this secret, and once the hacker has this key he can generate as much valid signatures as he wants with different claims and different headers

here's a wordlist repo has most common placeholders 

https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list

or u can also use hashcat to fuzz it out

you just need a valid JWT token from the target and the wordlist and you can use this command

```python
hashcat -a 0 -m 16500 <jwt> <wordlist>
```

Hashcat signs the header and payload from the JWT using each secret in the wordlist, then compares the resulting signature with the original one from the server. If any of the signatures match, hashcat outputs the identified secret in the following format, along with various other details:

`<jwt>:<identified-secret>`

lab 3: 

1)login as wiener get a valid token

2)open virtual machine and store your wordlist into a txt file secrets.txt

3)use this command to fuzz the secret 

```python
hashcat -a 0 -m 16500 <paste your token here> secrets.txt
```

`-a 0` 

is to specify the attack mode 0 is straight dictionary attack

`-m 16500` is to use jwt attack where it uses those wordlist secrets to generate a token and see if one of the matches

4)hit enter and the key is : secret1

5)go to burp and install jwt editor then click new symmetric key and assign the key specify secret → secret1  then click ok

6)go send a /get request to admin page send the request to the repeater

7)click json web token on the request  and modify the sub to administrator then click sign 

8)it will sign the header with the claims using the secret and make a valid token with valid signature 

9)now hit send or copy the forged token and put it into the browser using cookie editor or inspect

### **JWT header parameter injections**

A JWK is simply a JSON object that represents a cryptographic key. instead of sharing a key in a raw format (like a PEM file with `-----BEGIN PUBLIC KEY-----`), you share it as a structured JSON object.

example:

```python
{
  "kid": "a",
  "typ": "JWT",
  "alg": "RS256",
  "jwk": {
    "kty": "RSA",
    "e": "AQAB",
    "kid": "a",
    "n": "p-pdDNS5fZR3LpcqAeREeRZaFgS6LSnTBpCnzuajomo5amlRZHM48epHgc1zwLgpO-a9aMntfXX6OtTv_Rzo8Zou-QUOnCAM0d9HI79_PMNLnrOZDysx2TticH6yK9qcuuLGcyKQSAs_byheI848LbLr0UBkwxptYKoWmj5eHpKby5Dt_eemSzOi_i77I_uGShZTEWJQwdUxJ6I9Nv3l3xd_G_gTYU6en-hfjg5jj7Afgk6Ms_tQsADVnihZZJMmFWynsYyeIOa8gGLWxlXLdL-2XNGthhgcMIK7PWS2LRbBsSAKTXdOrjcCl7mX5jcmuzYpLn-TbkvkONGolONhmQ"
  }
}
```

now lets solve lab4:
in this lab you will try to inject that jwk in the header means ure telling the server to use this key to generate the signature and ignore the server key

so 

1)login as wiener:peter to get valid token

2)send a GET req to /admin page 

3)capture that request and send it to the repeater

4)now let's generate that key to use in our attack, go to JWT editor extension, then click new rsa key, keep the key size as `2048` cuz it's the standard size of the rsa key if lower the server may ignore it cuz it would be too weak 

5)get back to the request in repeater, click JSON Web Token then modify the sub to `administrator` 

6)click attack then choose embedded jwk attack and choose the key you just generated 

7)now it will embedded that jwk in the token header so that the server uses it to generate the signature with and now u have valid signature and valid token

8)now copy that token use it to delete carlos account

### **Injecting self-signed JWTs via the jku parameter**

`jku` (JWK Set URL): simply instead of making the server use the key from the header u make the server use the key from a exploit server that you use

now lets solve 

lab5:
1)go to JWT editor and generate a new RSA key

2)then go to the exploit server to use that key as the public key that the server uses 

the exploit server page will look like

```python
{
    "keys": [
{
    "p": "yGQsFj1NTMUMRRSkprv5PGHVLrP4syThTuHlNmn2GGxK4AXpjs_gfTMvEKrOuKirL-iDICNf32P3NAcD4ho2m6DiwHp-pW5hnfMhX5kCiy99imhYle1kSa3Oe6hSs7EGX5ztbire0Julz7fKQJE7YAqc4E9OEhWbjFIU_YkMwGM",
    "kty": "RSA",
    "q": "wewd5zz5OTbvp_SQBu7-gaA78Bcg9DwtPG4KJYktTkgJ9vYmRJuQtrLXrC4HYdRWABy9oPvpoKyBJELAN0StsJ_pDas5HEURK3kPEES-AlJltyPenKH0cfK_Wmd5TOkBN1c1VnYDrPZro94MAmSZhmTvCoQT6JKYSF7gastqvsk",
    "d": "GOwsED3-xCn5Ef7rz7sLO1vz1G9QXFO7k95s18QdAlwxW6xg21FQEdfRbVNJIEpWR9ujaHr_aMv1nbSzEEBgknZx85tGWqNKYpmVYKlz2P_mJxS6QwsmBBb1PJPNT3nIRYRjXRTwLKUZOvprcFnZ17RcoqGDmYDbuUCV44o-qCdgYdgfiHfX-d-Z_jS9XEzGR92450kfSE2AsuPg9-D8XRVR1Bq33y67lG3nBCXNE6URdT9pJme_YXTV5pRlaCE0pAorpCQl8sntbDAxlrTKKHUvolMOZ_slLsIJntch5aEP44CKHcCQZTkTtRN6WWaRlJ6W-PXFtQo0v0AT9X7kgQ",
    "e": "AQAB",
    "kid": "b0b6ef19-fa6b-4460-b063-44f51578af36",
    "qi": "Lw2_G12kzunmbndwj3pADf6gq_SK228cx7n8KKSKOCbDoLKJBJH3vrMFgLq31Ekv0HLto2w8aOyMnBkHVN3JXrwWmN2DHBqkG9Q5vjrFajyRLSWrF8hhHvm2HO-E2fHJTuQOt4VW8hQtgrU--9NpYppT5t6pw7yN4XLOksDnLQQ",
    "dp": "hQGQDZbUts7XLQbdnlmHvR9Ga0BDI0yoSz4-cBZ2pJFERVtHQWYSn6cYZxyoJwK01RCj7_Hq0ZA9ZQf--NTjR_rKZm0noFAadMcKcLRTbuSvk-1cVu8BMLIvKf54HhKyo0W6hfPoflfA_5UCpkZ_PWjt5SheLjyvSLy2d-2-S_c",
    "dq": "JSKYH133w9MXVaxpoEpIBn6uu61SLDMR3o6b9tAsEt-MuPQuI9k-fx4EWv59f2hwB5l5Xsie1pvyJwV5VZwbPsWAlZOCXj2DqjWGgvEHCd7Jh6agzJHfA6sepatG-UltaDGVDzeOQKL3veuZlSO6mpfdhsAAJ_tamhFBHHJTwLE",
    "n": "l8xY_cPUp7PM5Pjny6F5ZT0T_o1Xo_m3eRm2TCgVeLNiNZ7YyKoMox0y8RNgla90WL3XXcwJOfEdu1gkqXrJD5WEhYyDeLYUUMvZCOr661e5XJI1eoOL4LaP-h_UroPlViRf9q5H-_Am1IAOuMAZirYQJ4up2aiSCW3NeBqy-Ln8u9cPktBWD5QYYUzvjF_qqCJEygGjOcV6wzJCJ1YhkSjYu0QSdzWxmDb5bBRfO8VahS2uzFKVaRNzxw-RAip0WSFyvx9ZLpeGwrwW9CxNfU3YeKzXdkJlc20bjgGxueLLyPCnRrrbWlXNwovTuwpDxoiTpErqvGA4psucO8qHuw"
}
    ]
}
```

3) now go send a get request the /admin page after logging in as wiener

4)send that request to repeater and click JSON Web Token, edit the kid in this token to match the one marked red in your generated key

5)modify the sub as administrator

6)and now time to inject that jku in the header it will look like 

`“jku”: “<exploit server url>”` 

the final header will look like

```python
{
    "kid": "b0b6ef19-fa6b-4460-b063-44f51578af36",
    "alg": "RS256",
    "jku": "https://exploit-0ae70022035f8b1081fa2e9001fc00f3.exploit-server.net/exploit"
}
```

7)sign this token using the key you just made copy it and use it to login as an admin

### **Injecting self-signed JWTs via the kid parameter**

the Key id (kid) is a parameter tells the server which key is used to verify and generate this signature 

this kid parameter can be tested against sqli and path traversal as  any other parameters 

so if this key i vulnerable to path traversal you can use it to assign the key to a known key in the server if u can see the files in the server or you can redirect it to an empty file so that the key is NULL (a known empty file is `dev/null`in linux systems 

lets solve lab 6:

1)assume that the kid parameter is vuln to path traversal or you can test it in the wild by doing `“kid": "nonexistent_key_12345”` and see if you get an error like `500 Internal Server Error` or `Key file` `/var/www/app/keys/nonexistent_key_12345 not found.`

2)now since it vuln to path traversal we will try to redirect it to the dev/null file by using `“kid”:”../../../../../../dev/null ”`

3)now you told the server to use this key to verify your token now you need to assign your token to that null key

4)go to jwt editor, click on new symmetric key, generate, ok

5)now it made a new random key,  but you need to make it null so modify the k parameter in it to `AA==`   this is null in base64 encoding

so the key will look like 

```python
{
    "kty": "oct",
    "kid": "1d18f1f1-fa93-4476-ab80-c8769b6f6616",
    "k": "AA=="
}
```

6)send a request to /admin page, modify the kid to `../../../../../../dev/null` and the sub to administrator assign that token using your null key and hit send

7)take that token to login as an admin and delete carlos
