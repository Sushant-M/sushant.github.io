---
layout: post
title: "Mystic CSP Errors"
date: 2026-05-29 11:35:53 -0700
categories: blog
---

## What is the Form-Action directive?

According to the Mozilla documentation, the `form-action` directive limits the URLs that can be the target of a form submission.

So if I send back a CSP header to the client with a `form-action` set to a certain pattern of URLs, the client will not be able to submit a form to any URL that does not match that pattern. For example:

```
Content-Security-Policy: form-action https://my-new-website.com;
```

These will only let the client send form data to any URL matching the `form-action` directive. In this case, the client will not be able to submit forms to any URL that does not match `https://my-new-website.com`.

## What is the use of this restriction?

Well in case an attacker manages to maliciously inject a form or a script that can submit a form (also known as a cross site scripting attack), it can be blocked if the `form-action` directive is set to a certain pattern of URLs. So if they somehow manipulated the website to send the form data to anything other than `my-new-website`, the browser will stop this request. When you look at the console, you would see the following error:

```
Content-Security-Policy: form-action: Blocked form submission to 'https://attackers-website.com/' because it violates the following Content Security Policy directive: "form-action 'self'".
```

## So why is this confusing in any way?

If you clicked on the Mozilla link above, you'd see a giant banner that says

> **Warning:** Whether form-action should block redirects after a form submission is debated and browser implementations of this aspect are inconsistent (e.g., Firefox 57 doesn't block the redirects whereas Chrome 63 does).

What is this alluding to? There are many cases where you'd like to submit a form and _then_ be redirected to a different page. For example, if you submit a form to `https://my-new-website.com/login`, the credentials will be validated and then the page might redirect you to /home, /dashboard or anything else you could have configured.


## What about redirects to websites that are not on the same domain?

There are cases where you want the redirect to be a 3rd party website outside of your domain. Imagine if the user embeds your application into their website. They sign in through your application and then want to be redirected to their own application. So the flow would be:

`User -> Embedding website -> your application -> login -> Embedding website`

## The main CRUX of this: The misleading error message on Chromium browsers when a redirect is blocked.

Let's demonstrate this through code, it'll be extremely clear that way.

Let's build a simple few pages that do the following:

Page 1: Contains a form with a simple input that asks for your name.
Page 2: This is the page that the form from page 1 is submitted to. This page does nothing but accept the form and then redirect you to a 3rd page.
Page 3: This will be a 3rd party page. The place where you want the user to be redirected. In this example, we'll use `google.com`.

On page 1, we'll add a CSP header with the `form-action` directive set to `http://localhost:8000`. This will ensure that you can only submit the form to the same domain `http://localhost:8000`.

Code so far:

### Page 1:

```
<?php
header("Content-Security-Policy: form-action http://localhost:8000");
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Secure Page</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <h1>CSP Demo</h1>
    <p>Input your name!</p>
    <form action="page2.php" method="POST">
        <input type="text" id="name" name="name"><br>
        <button type="submit">Submit!</button>
    </form>
</body>
</html>

```

### Page 2:

```
<?php
  // This is a server-side redirect
  header("Location: https://www.google.com");
  exit;
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Response Page</title>
</head>
<body>
    <p>You should not see this if the redirect works.</p>
</body>
</html>
```

### Let's run this in Chrome and see what happens:

![Blocked Redirect on Chrome Screenshot](</assets/images/page1_chrome.png>)

Notice the error message here, we'll get back to it after we see what Firefox does:

```
Sending form data to 'http://localhost:8000/page2.php' violates the following Content Security Policy directive: "form-action http://localhost:8000". The request has been blocked.
```

Now let's run the same code on Firefox:

![Page 1 on firefox](</assets/images/firefox_page1.png>)

When we hit the form submission:

![Successfull redirect on firefox](</assets/images/firefox_redirect.png>)

## So why in the world is the error message so misleading on Chrome?

If you got this far I'm sure you're thinking the same thing. Why does Chrome say tht the URIs don't match? On first glance it seems obvious that they do.

It's to obfuscate the redirect URL and prevent information leaks. It is misleading by design. They could potentially improve the error messaging to say that it is due to the redirect being blocked but that's upto the Chromium team.

Here is a [link](https://source.chromium.org/chromium/chromium/src/+/main:third_party/blink/web_tests/http/tests/security/contentSecurityPolicy/1.1/form-action-src-redirect-blocked-in-new-window-expected.txt;l=1?q=%22Refused%20to%20send%20form%20data%22&ss=chromium) to the unit test on Chromium that tests for this behavior.