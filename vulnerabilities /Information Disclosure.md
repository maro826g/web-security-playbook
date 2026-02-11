information disclosure is when a webiste leaks a sensitive information it could include :

- Data about other users, such as usernames or financial information
- Sensitive commercial or business data
- Technical details about the website and its infrastructure

how information disclosure happen :

**Failure to remove internal content from public content: for example comments in the code
 
Insecure configuration of the website and related technologies: for example  failing to disable debugging, default configurations by displaying a long error message**
 

**Flawed design and behavior of the application: when a website send a different response error message for each diffrent error (like saying wrong username while logging in, this way u can guess all usernames in the website)**

**Common sources of information disclosure:

1)Files for web crawlers: many websites use** 
 `/robots.txt` and `/sitemap.xml`
to help browsers to avoid some pages and directories
so always check for those two pages manually

**2) Directory listings**
if a directory doesn't have an index page you will be able to see the list of the contents inside them this can be dangerous in some cases

3) **Developer comments**

some developers forget to remove their comments from the code, you should always check the page source and dev/tools

4)**Error messages**

sometimes error messages are so detailed and has sensitive informations so you should make sure to check this to

solving first lab :
**Information disclosure in error messages**

1)navigate to one of the products page 
2)change the product id to a non expected value (characters)
3)whatch urself getting directed to an erorr page the has the apache server version 

5)**Debugging data**

during the debugging phase many websites generate custom error messages and logs that contain large amounts of information about the application's behavior
it can include 

- Values for key session variables that can be manipulated via user input
- Hostnames and credentials for back-end components
- File and directory names on the server
- Keys used to encrypt data transmitted via the client

Debugging information may sometimes be logged in a separate file. If an attacker is able to gain access to this file, it can serve as a useful reference for understanding the application's runtime state

solving second lab :
 **Information disclosure on debug page**

1)you can use burp by enabling live passive crawl 
2) go to target page 
3)in urlveiw right click on the the webiste url

4)engement tools, find comments
///
1) or you can simple right click veiw page sorce  and look for the comment manually
2) use the url in the comment and go to a PHP info page 
3)get the secret_key and paste it in the submit

6)**User account pages**
this is literally an IDOR vulnerability by changing the username in the url you get to see other user's information 
GET /user/personal-info?user=attacker→GET /user/personal-info?user=victim

7)**Source code disclosure via backup files** 

Access to a website’s source code makes it much easier for an attacker to understand how the site works and create serious attacks.

Sensitive data like API keys or backend login details are sometimes hard-coded in the source code itself.

Websites might accidentally expose their own code through backup files created by text editors (for example

```
file.php~
```

or

```
.bak
```

files).

Because servers usually run files like

```
.php
```

instead of showing them, attackers can trick the server by requesting these backup versions to read the raw code.

solving the third lab 
**Source code disclosure via backup files
1)go to /robots.txt
2)you will find /backup in it
3)navigate to /backup
4)scroll down to see the database connection code and get the password from it and submit it 

or go to burp sitemap and map all the pages of the webiste to get the /backup contenets**

having the sorce code can lead to huge vulnerabilities such as **Insecure deserialization**

8)**Information disclosure due to insecure configuration**

some website can have improper configuration duo to widespread use of third-party technologies so this can lead to a vulnerability for example having the `HTTP TRACE` enabled

solving the fourth lab 
**Authentication bypass via information disclosure

1)try to fuzz directories and you will find /admin but it says not authorized
2))get the request in burp and try to but X_Forwarded-For: 127.0.0.1 to trick the website that you are from the local host (wont work)

3) now try to send it with diffrent methods 
heres a link for all request methods types**
https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods

4)you will find out that TRACE request method works

5)you will find out that it uses X-Custom-IP-Authorization 

6)now add it in ur request with the local host ip so it will be like 

```python

X-Custom-IP-Authorization: 127.0.0.1
```

7)add it to ur request and send it on burp and you will be able to see the url that redirects you to delete carlos user
7`) or you can go to proxy→ match and replace
8) click add request header
9) turn off interceptor and navigate to your page find urself has admin panel and admin features to delete
 

9)**Version control history**

Access to a website’s version control data can expose sensitive information about the site.

- Websites often use **Git** for version control, which stores data in a hidden **.git** folder.
- Sometimes **.git** is accidentally exposed in production (`/.git`).
- Attackers can **download the .git folder** to see commit logs, diffs, and code snippets.
- Even partial code can reveal **hard-coded secrets** like API keys or backend credentials.
- Comparing diffs may also leak **application logic** or other valuable details.

solve fifth and last lab 
 **Information disclosure in version control history
1) first check if theres /.git directory
2) download it on ur machine with this command**
- wget -r https://YOUR-LAB-ID.web-security-academy.net/.git/

-r here important cuz ure now gripping all the git structure to analyze it 
3)do git log to get all commits happened before 
4) you will find a commit says Remove admin password from config (get the id or the commit hash for this commit we will need it)
5) git show <COMMIT_HASH> // to get all the change happened before and you will find the **hard-coded admin password** in red (removed) and green (added) lines
6)get in the admin account using administrator as username and that password you have got 

**Preventing information disclosure vulnerabilities**

- **Raise awareness** – Make sure all team members know what data is sensitive; even harmless-looking info can help attackers.
- **Audit code** – Include checks for information leaks in QA/build processes (e.g., strip developer comments automatically).
- **Use generic error messages** – Don’t reveal details about application behavior.
- **Disable debugging in production** – Turn off diagnostic or debugging features on live sites.
- **Secure third-party tools** – Understand their configs and security implications; disable unused features or settings.
