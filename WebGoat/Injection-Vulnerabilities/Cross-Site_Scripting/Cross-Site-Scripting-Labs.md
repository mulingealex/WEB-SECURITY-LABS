Cross-Site Scripting (XSS) is a web application vulnerability that occurs when untrusted user input is rendered by the browser without proper sanitization or output encoding.

This allows attacker-controlled JavaScript to execute within the context of a trusted web application.

Basic proof-of-concept payload:

```
<script>alert(1)</script>
```

If reflected or rendered unsafely by the application, the browser interprets the payload as executable JavaScript instead of plain text.

# 1. REFLECTED XSS
Reflected xss is a vulnerability where:
    User input is immediately reflected by the server
    and executed as JavaScript in the victim's browser.
## Overview
This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability in the `CrossSiteScripting` lesson of WebGoat

The objective was to identify a vulnerable input field, inject a malicious JavaScript payload, intercept and manipulate requests using burpsuit and confirm successful client-side script execution.
# Objective

Identify which user-controlled parameter reflects unsanitized input back into the server response and execute arbitrary JavaScript in the victim's browser.
# Initial Reconnaissance
The vulnerable page contained:
- Shopping cart items
- Credit card input field
- Access code input field
- Quantity parameters
The application hinted that:
- `alert()`
- `console.log()`
could be used to identify XSS vulnerabilities.

![[Pasted image 20260527123702.png]]
# Vulnerability Discovery

The following payload was injected into the vulnerable parameter:

```
<script>alert(1)</script>
```

![[Pasted image 20260527123847.png]]

The request was intercepted and analyzed in Burp Suite.

![[Pasted image 20260527123922.png]]

# Request Manipulation with Repeater

The captured request was sent to the Burp Repeater module for controlled testing.

![[Pasted image 20260527124103.png]]

A successful response confirmed:
- payload reflection
- vulnerable rendering behavior
- JavaScript execution
# Successful Exploitation
The browser executed the payload and displayed:

```
alert(1)
```

![[Pasted image 20260527124541.png]]

This confirmed the presence of a reflected XSS vulnerability.
# Technical Analysis
The vulnerability existed because:
1. User-controlled input was accepted by the server
2. Input was reflected into the response
3. No output encoding or sanitization was applied
4. The frontend rendered the response using unsafe HTML rendering logic

# 2. DOM BASED XSS
## Overview

This lab demonstrates how a **DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability can occur when client-side JavaScript processes attacker-controlled input from the URL and renders it into the DOM without proper sanitization.

Unlike traditional reflected XSS, the malicious payload never needs to be reflected by the server. The vulnerability exists entirely in the browser.
# Objective
The objective was to:
- Identify a hidden client-side route
- Analyze Backbone.js routing logic
- Trace attacker-controlled input through JavaScript handlers
- Exploit unsafe DOM rendering
- Trigger JavaScript execution through a crafted URL payload
# Understanding DOM-Based XSS

DOM XSS occurs when:

```
Attacker-controlled input → Client-side JavaScript → Dangerous DOM sink
```

The browser itself becomes responsible for the vulnerability.
# Initial Reconnaissance
The lesson hinted that the vulnerability could be discovered by inspecting:

```
client-side route configurations
```

Using Firefox Developer Tools (`F12`) and the Debugger search functionality (`Ctrl + Shift + F`), the JavaScript source code was searched for route definitions.

Search keyword:

```
route
```

![[Pasted image 20260527150353.png]]
# Discovery of Hidden Route

The following Backbone route was identified:

```
'test/:param': 'testRoute'
```

This indicated that any URL matching:

```
#test/<value>
```

would invoke the `testRoute()` function.
# Identifying the Vulnerable Sink

The application reflected attacker-controlled input into the page without proper encoding.

The vulnerable route format became:

```
start.mvc#test/
```

which allowed arbitrary attacker-controlled payloads to be appended.

![[Pasted image 20260527150604.png]]
# Initial Payload Attempt

The first payload tested was:

```
<script>webgoat.customjs.phoneHome()</script>
```

Full URL:

```
http://127.0.0.1:8080/WebGoat/start.mvc#test/<script>webgoat.customjs.phoneHome()</script>
```

However, the payload did not execute.
# Why the Script Payload Failed

Modern browsers often do not execute dynamically injected `<script>` tags when inserted through DOM manipulation methods such as:

```
innerHTML
```

This is an important real-world DOM XSS nuance.
# Successful Payload

A more reliable event-handler-based payload was used instead:

```
<img src=x onerror=webgoat.customjs.phoneHome()>
```

Full exploit URL:

```
http://127.0.0.1:8080/WebGoat/start.mvc#test/<img src=x onerror=webgoat.customjs.phoneHome()>
```

![[Pasted image 20260527150845.png]]
# Successful Execution
The browser console displayed:

![[Pasted image 20260527151119.png]]

```
phoneHome invoked
```

and returned a success response containing a random number:

```
phoneHome Response is 1265537972
```

This confirmed successful DOM-based XSS exploitation.

# 3. STORED XSS

Stored XSS occurs when malicious JavaScript is **saved on the server** (database, comments section, message board, profile field, etc.).
## Lab Scenario

The application contained a comment feature that allowed users to submit and view messages. Since submitted comments were stored and displayed to all visitors, the feature presented a potential attack surface for Stored XSS.

The challenge required posting a comment that would execute the following JavaScript function:

```
webgoat.customjs.phoneHome()
```

Successful execution would generate a unique response value that could be used to complete the lesson.
## Reconnaissance

Before attempting exploitation, I reviewed the application's functionality and identified the comment field as the primary input vector. Existing comments were displayed to users, indicating that user-supplied content was being rendered within the page.
## Exploitation

To test for Stored XSS, the following payload was submitted through the comment form:

```
<script>webgoat.customjs.phoneHome()</script>
```

![[Pasted image 20260605132139.png]]

The application accepted the input and stored it successfully.

After reloading the page, the stored content was rendered and the embedded JavaScript executed automatically within the browser.
## Verification
To confirm execution, browser Developer Tools were used to inspect application requests and responses.

![[Pasted image 20260605132307.png]]

Analysis of the generated network traffic revealed the application's response:

```
phoneHome Response is 1243505505
```

The returned value confirmed that the injected JavaScript had executed successfully within the application's context.
### Attack Flow
1. Attacker submits malicious JavaScript.
2. Application stores the payload in a database.
3. Victim visits the affected page.
4. Browser renders and executes the stored script.
5. Attacker achieves their intended objective, such as session theft, account takeover, or data exfiltration.
