## Minimize security-related feedback to untrusted users
e.g. "Login failed" Don't say why!  
More detail would help attackers to get more information.  

## HTML taget & window.open()
```html
<h3>Table of Contents</h3>
<ul>
  <li><a href="/example/html/pref.html" target="view_window">Preface0</a></li>
  <li><a href="/example/html/pref.html" target="_top">Preface</a></li>
  <li><a href="/example/html/chap1.html" target="_self">Chapter 1</a></li>
  <li><a href="/example/html/chap2.html" target="_parent">Chapter 2</a></li>
  <li><a href="/example/html/chap3.html" target="_blank">Chapter 3</a></li>
</ul>
```  
Target can used to control the behaviors of clicking action.  
If not using `_self` in target, there would be potential vulnerability: **Tabnabbing**.  
What people always don't know is that: The page we linked to by taget, can gain partial control of the source page with `window.opener`! `window.opener.location` isn't limited to CORS!  
The new opened tab can modify `window.opener.location` to redirect source page to a malicious page such like phishing page.  
People must find that their source page has been redirected? No, attacker could make their malicious page similar to the original source page and make a query of credentials!  
The most famous example is facebook:  
Assume that you see a social link on the page and click it.  
The click would open up a new tab page. In the page, there is a script:  
```js
if(window.opener){
  window.opener.location = "phishing" + document.referer;
}
```  
When you go back to the original tab page, you would find that your page has been redirected!  
[DEV Community](https://www.facebook.com/pg/thepracticaldev/about/)  
[dev.to](https://github.com/thepracticaldev/dev.to)  

Mitigation:  
To prevent from utilization of `window.opener`, please use `rel=noopener`, this can make sure `window.opener=null` in later version of Chrome and Opera.  
For the browsers who still not support `rel=noopener`, please use `rel=noreferrer`, this would stop the referrer in http header.  
In addition, you can also use the following code:  
```js
var otherwindow = window.open();
otherwindow.opener = null;
otherwindow.location = url;
```  
Use the javascript to force the opener value to be null.  

## Metacharacter
Escape any characters you send back that might be interpreted as metacharacters!  
`java.net.URLEncoder.encode` (url-encode):  
translate string into application/x-www-form-urlencoded, space converted to "+", alphanumeric remain the same, some other would be represented by %xy(hex representation).  
Some web application framework would also work on escape the output. So you need to know when escape happen, to prevent escape double times!  
e.g. Ruby on Rails: ERB template

## HTTP Cookie
Server would reply with fields to set cookies: `Set-Cookie: name=value`  
If no expiration time set, cookie expires when browser exists.  
Cookie attributes:  
cookie domain: server domain when cookie sent  
path: path when cookie sent  
expriation  
secure flag: Only transmit on encrypted channel  
HttpOnly flag: Only http/https can access the cookies

## Session management
Http itself is **stateless**, it won't record the state or relationship between current request and previous requests.  
But it is not convenient for interactive applciations such like shopping chars.  
Therefore, we need session management, by passing a "session id".  
Good session ids are not guessable  
Good session management should prevent from session hijacking and CSRF.  
Session id should have at least 128 bits of random data, cryptographically securely pseudo-random number generator instead of rolling down by developers themselves, encrypting session ids over untrusted channels.  

**Session fixation attack**  
1. attacker visits the website and website return a session id to him  
2. attacker use the session id to create a link and send it to victim  
3. victim click the link and login into the website, session is successfully built up!  
4. website doesn't change the session id, attacker use the same session id to hijack the session of victim's  
Mitigation:  
**generate a new session id after login!!**  
also use httponly flag to prevent accessing cookie with javascript(session info would stored in cookie)  
limit time of session  
**Make sure that logging off disables cookie**: otherwise, cookie would continue session, and session can be captured.  

## XSS
Stored based XSS: the most persistent and can be hidden from the user because the it stored in the database and triggered when shown on webpage.  
DOM based and reflected XSS: DOM based can be part of reflected XSS. They both need user interaction(URL input) to trigger the attack.  

## CSRF(XSRF)
CSRF doesn't help attacker gain control of victim, but can deceive the browser that his behavior is victim's behavior!  
CSRF exploits the browser's trust on user, XSS exploits the user's trust on browser.  
Mitigation:  
Check HTTP Referer: referer should be in same domain with "vulnerable url"! This method is great and easy, but attacker also can modify http header value.  
Check with token: The "vulnerable url" should works with token which is not stored in cookies.The token should be pseudo-random number and cannot be be guessed by attackers. Token can be attached in the form.  
Use POST  

## Hardening web application with http headers
* CSP  
* HSTS(Use https only)  
* X-Content-Type-Options  
* X-Frame-Options  
* X-XSS-Protection  

## Cross-Origin resource sharing (CORS)
* Access-Control-Allow-Origin  
* Access-Control-Allow-Methods  
* Access-Control-Allow-Credentials  
Be careful, if access-control-allow-credentials is true, it means that you completely trust the site and it's absolutely necessary!
