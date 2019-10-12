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
