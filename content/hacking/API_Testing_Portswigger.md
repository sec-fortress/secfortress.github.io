---
title: API Testing || Attacking REST, Swagger APIs {Portswigger}
image: https://i.imgur.com/mStB3iJ.jpeg
description: Started researching on attacking APIs few days ago, which has been fun....In the below walkthrough i have solved all API testing labs on portswigger.
categories: [API]
tags: [Web, Burpsuite, Fuzzing, API]
weight: 2
author: SecFortress
---


**APIs (Application Programming Interfaces) enable software systems and applications to communicate and share data. API testing is important as vulnerabilities in APIs may undermine core aspects of a website's confidentiality, integrity, and availability. Let dive into this lab dummy walkthrough :)**


## **Exploiting an API endpoint using documentation**


**Objective:** To solve the lab, find the exposed API documentation and delete user `carlos`. You can log in to your own account using the following credentials: `wiener:peter`.


Login in as user wiener and sending a request to burp we have this :


![](https://i.imgur.com/z9B4syr.png)


We can then start to look for common API endpoints like the following. You can also try to fuzz for the API endpoints using the following wordlist [here](https://gist.github.com/yassineaboukir/8e12adefbd505ef704674ad6ad48743d).

- `/api`
- `/swagger/index.html`
- `/openapi.json`


Fortunately for me i was able to discover the API endpoint by just making a GET request to `/api` without fuzzing which returned the API documentation.


![](https://i.imgur.com/8AjRUgt.png)

I then made a GET request to retrieve series of account information trying out usernames we have at hand



![](https://i.imgur.com/5UVtpzp.png)
[User => Wiener]


![](https://i.imgur.com/AbSV5My.png)
[User => Carlos]

![](https://i.imgur.com/YtzUYaD.png)
[User => Administrator ]



We can also change the email of a particular user take for example user `wiener` by using the **PATCH** request method together with an provided data in JSON format.

- Change request method to **POST** using burp functionality then manually change **POST** header to **PATCH** method.
- Change `content-type` to `application/json`
- Then use the JSON format to send the data.


![](https://i.imgur.com/I18zhbt.png)


We can then finally delete the user `carlos`  as requested by changing request method to **GET** and then changing **GET** to **DELETE** manually


![](https://i.imgur.com/PdtIJf4.png)



## **Finding and exploiting an unused API endpoint**


**Objective:** To solve the lab, exploit a hidden API endpoint to buy a "**Lightweight l33t Leather Jacket**". You can log in to your own account using the following credentials: `wiener:peter`.


As said by the objective we need to first of all add the item to cart 


![](https://i.imgur.com/R6NMtSp.png)



After moving this item to cart and login in with the given credentials we cannot purchase this item cos' we have insufficient funds


![](https://i.imgur.com/0c8k7Ox.png)


Since i am using burp pro we need to setup [jython](https://repo1.maven.org/maven2/org/python/jython-slim/2.7.4/) in other to be able to download some extensions that need this. Download the first file from the given link and import it to burpsuite as shown below in the BApp store.

![](https://i.imgur.com/rYcJ59n.png)


We then need to download two burp extensions by navigating to the burp extender tab :

1. We can use the [JS Link Finder](https://portswigger.net/bappstore/0e61c786db0c4ac787a08c4516d52ccf) BApp to crawl endpoints for sensitive files containing API information.
2. We can use the [Content type converter](https://portswigger.net/bappstore/db57ecbe2cb7446292a94aa6181c9278) BApp to automatically convert data submitted within requests between XML and JSON.

Once downloaded we are good to go, Navigate to **Targets** > **Site map**, then click on the targets and select **Engagements tools** > **Find scripts**


![](https://i.imgur.com/3RtAl4L.png)


This would pull up all scripts and we can try to see if there are any scripts literally related/can serve as a documentation for the API endpoint.


![](https://i.imgur.com/XSvv85M.png)



Sending this to the repeater tab and checking the response we finally found an API endpoint in the following format : `${url.host}/api/products/${encodeURIComponent(productId)}/price`  



![](https://i.imgur.com/E5YtCiF.png)


We can also attempt to find this hidden API endpoints by crawling the site using burp suite pro feature which should give you the API endpoints


![](https://i.imgur.com/idNuSdo.png)


![](https://i.imgur.com/la4ea0n.png)
[API Endpoint Discovered]

Sending this requests to my repeater tab i noticed that the items are judged by their product ID which returns the price for that particular product.

![](https://i.imgur.com/0ysIwkc.png)



Changing product ID to 1 we can see the price for the "Lightweight "l33t" Leather Jacket" item


![](https://i.imgur.com/lk0THN2.png)


![](https://i.imgur.com/c5UaZ2r.png)


Since there is typically no way to alter our own amount in our wallet we can actually alter the amount of the product changing it to `$0.00` 🤣.

Go ahead and right click then select **Extensions** > **Content Type Converter** > **Convert to JSON**, this should change the request method to **POST** and also add a `content-type` value to the request


![](https://i.imgur.com/gYEN9ZW.png)


As shown below we get the error "**Method not allowed**", well we can use the **PATCH** request method also as this is accepted by most APIs


![](https://i.imgur.com/n9qhRs4.png)

This successfully changes the price to `$0.00`

![](https://i.imgur.com/02yvlII.png)

Then sending a GET request we can see that the product price has been changed to `$0.00`, sweet. Note that if you get the **"Unauthorized"** message then login as user `wiener` first before doing all of this (if possible you just need the new session key).

![](https://i.imgur.com/dbJcOV3.png)


Navigating back to the website we can see that we have successfully reduced the price of this product and can add it to cart now.

![](https://i.imgur.com/hMvapxe.png)

You should be able to solve the lab now by purchasing the item.

![](https://i.imgur.com/NfMkBcB.png)



## **Exploiting a mass assignment vulnerability**



**Objective:** To solve the lab, find and exploit a mass assignment vulnerability to buy a **Lightweight l33t Leather Jacket**. You can log in to your own account using the following credentials: `wiener:peter`.


This vulnerability exists do to the fact that -:

- the application supporting parameters that were never intended to be processed by the developer.
- For example, if a `PATCH` request is made to `/api/users/` 
- that allows users to update their data with the following result;


```json
{
    "username": "wiener",
    "email": "wiener@example.com",
}
```

A concurrent `GET /api/users/123` request returns the following JSON data:


```json
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```


> This shows that the hidden `id` and `isAdmin` parameters are bound to the internal user object, alongside the updated username and email parameters.


An attacker can then send a `PATCH` request with the `isAdmin` parameter value set to `true` ;

```json
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": true,
}
```



For the lab we need to download the [Param miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943) BApp which guesses up to 65,536 param names per request, based on information taken from the scope. I won't be using this though. but it is worth downloading. You can go ahead and the item to cart as requested. (Don't forget to login as user `wiener`)



![](https://i.imgur.com/UaPkbEw.png)


Then intercept the request with burpsuite 


![](https://i.imgur.com/82f6tCp.png)

I noticed that after adding the item to cart an API request is made to `/api/checkout` as shown in the above screenshot.


Forwarding this request to the repeater tab, the response is a JSON data which contains the information about the present item we have in our cart


![](https://i.imgur.com/0gqyoL9.png)

Decided to check other request method accepted by this endpoint and as shown below it only accepts **POST** and **GET** request methods.


![](https://i.imgur.com/UC8HhzT.png)


So i decided to alter the request first of, by changing the request method to **POST** (`Change request method`), Then i used the `Content type converter` extension we used in the last challenge to change request body to JSON, then copied the whole JSON data we got from the response to our request tab.

Altering the `item_price` value to 0, nothing changes because we do not have that certain permission to do so as we can see below we are still getting the **INSUFFICIENT_FUNDS** error. 


![](https://i.imgur.com/VRoi8Mi.png)


Then i decided to alter the `percentage` discount value giving our self an 100% discount


$$
\text{Discount Percentage} = \left( \frac{1337 - 0}{1337} \right) \times 100 = 100\%
$$


![](https://i.imgur.com/H9jQxi9.png)



We should now have the lab solved.


![](https://i.imgur.com/KuGvQqS.png)



## **Exploiting server-side parameter pollution in a query string**



- Query syntax characters like `#`, `&`, and `=` can be used to test for this vulnerability
- An attacker can then observe how the application responds.
- It is also crucial to URL-encode this characters. 
	- Otherwise the front-end application will interpret it as a fragment identifier and it won't be passed to the internal API.


**Objective:** To solve the lab, log in as the `administrator` and delete `carlos`.

First of all we start by intercepting the login page cos' we have not been given a **username** neither a **password**.

![](https://i.imgur.com/j1UAid6.png)

 We can then provide wrong credentials and intercept, sending the request to the repeater tab


![](https://i.imgur.com/5lZbdys.png)

After several enumeration figured out there is probably no way in the previous endpoint and started to focus on the `/forgot-password` endpoint

![](https://i.imgur.com/OMnk6qw.png)


adding a parameter with the URL encoded character of `&` as `%26`, i figured out that the extra username parameter is not been read at all so specifying wrong input in the first parameter leads to an "**Invalid Username**" error


![](https://i.imgur.com/jhETqA5.png)

Mean while reversing the case as shown below we have a valid response


![](https://i.imgur.com/62qlHUM.png)

Then i decided to move a step further by adding the `#` encoded character `%23` to the end of the query and we have the error "**Field not specified**"

![](https://i.imgur.com/L57Ubaa.png)

Then decided to bruteforce this parameter by sending this to the intruder tab and selecting the second parameter as this position point for this attack. The major reason for doing this is because there is an hidden parameter which could be altered if we know it.

![](https://i.imgur.com/qS1uUJM.png)

Then navigate to **Payloads** > **Payload Options [Simple list]** > **Add from list** and select "Server-side variable names" as the wordlist.

![](https://i.imgur.com/hxnSim2.png)

Start the attack and as shown below the length of the response from the "`field`" parameter is SUS

![](https://i.imgur.com/gsUa9Ls.png)

Changing the `field` value to `username` we can see that there is a change in the output.

![](https://i.imgur.com/fP5QoHL.png)

Using burp pro crawling feature i was able to find the `/static/js/forgotPassword.js` endpoint with the following hidden `reset_token` parameter

```js
forgotPwdReady(() => {
    const queryString = window.location.search;
    const urlParams = new URLSearchParams(queryString);
    const resetToken = urlParams.get('reset-token');
    if (resetToken)
    {
        window.location.href = `/forgot-password?reset_token=${resetToken}`;
    }
    else
    {
        const forgotPasswordBtn = document.getElementById("forgot-password-btn");
        forgotPasswordBtn.addEventListener("click", displayMsg);
    }
```

Parsing the `resest_token` parameter as a value to the `field` variable, we have a reset token.

![](https://i.imgur.com/GuoFSk4.png)


Copied this reset token and went over to the URL specified in the JS file `/forgot-password?reset_token=${resetToken}` specifying the reset token given which referred me to a page to reset the administrator password "`admin:admin`"


![](https://i.imgur.com/BP8wtoN.png)

Successfully logged in as administrator and deleted the user `carlos` to complete the challenge 

![](https://i.imgur.com/E7EsztQ.png)


## **Exploiting server-side parameter pollution in a REST URL**

**Objective:** To solve the lab, log in as the `administrator` and delete `carlos`.


- REST APIs may place parameter names and values in URL path, rather than the query string.
- For example requests are made to the following endpoint as:

	```
	GET /edit_profile.php?name=peter
	```

- This results in the following server-side request:

```
GET /api/private/users/peter
```

Just as mentioned in the [academy](https://portswigger.net/web-security/api-testing/server-side-parameter-pollution#testing-for-server-side-parameter-pollution-in-rest-paths) section of this challenge we have to combine [path traversal](https://portswigger.net/web-security/file-path-traversal) sequences to modify parameters and observe how the application responds.

First of we start by traversing twice and seeing if this application is truly vulnerable as shown below. 

![](https://i.imgur.com/M902zMN.png)


Again i tried to guess the users path by traversing 3 times which worked as shown below meaning there is correct path like `../../../users/carlos`

![](https://i.imgur.com/fXSqCsL.png)


Took it a step further again by guessing the `field` variable just as used in the previous challenge and we could actually read the email field correctly.


`username=administrator..%2f..%2f..%2fusers%2fcarlos%2ffield%2femail%23`



![](https://i.imgur.com/l9EinZR.png)



Just to be sure this parameter actually exists i decided to change the field value to `username` instead and got the below error talk about the API version.


![](https://i.imgur.com/WJXXpCB.png)


Starting to research i came across this [blog](https://medium.com/apis-you-wont-hate/api-versioning-has-no-right-way-f3c75457c0b7) talking about API versioning and discovered we can traverse 4 times and then start our query with a encoded `/v1....` which worked.

`administrator..%2f..%2f..%2f..%2fv1%2fusers%2fcarlos%2ffield%2fusername%23`


![](https://i.imgur.com/TlCi4xe.png)


Now we need to find the hidden parameter for the password reset token. I started to look up JS files by using the crawl feature of burpsuite as used in other challenges and came across a JS file located at `/static/js/forgotPassword.js`  which revealed the hidden parameter (`passwordResetToken`)

```JS
forgotPwdReady(() => {
    const queryString = window.location.search;
    const urlParams = new URLSearchParams(queryString);
    const resetToken = urlParams.get('reset-token');
    if (resetToken)
    {
        window.location.href = `/forgot-password?passwordResetToken=${resetToken}`;
    }
    else
    {
        const forgotPasswordBtn = document.getElementById("forgot-password-btn");
        forgotPasswordBtn.addEventListener("click", displayMsg);
    }
});
```


Made a request and this was successful as expected we got the reset token value.
`administrator..%2f..%2f..%2f..%2fv1%2fusers%2fadministrator%2ffield%2fpasswordResetToken%23`


![](https://i.imgur.com/tJNuSvD.png)


We can place this value in the following parameter on your browser `/forgot-password?passwordResetToken=${resetToken}`, which should give you access to a password reset page.



![](https://i.imgur.com/v64wSwe.png)


Then we can login as user `administrator` and delete the user `carlos` as requested.

![](https://i.imgur.com/aWUzfBh.png)

### **Resources**


- [https://hackerone.com/reports/1695454](https://hackerone.com/reports/1695454)
- https://hackerone.com/reports/1218680
- https://hackerone.com/reports/1218461
- https://hackerone.com/reports/232650
- https://github.com/reddelexc/hackerone-reports/blob/master/tops_by_bug_type/TOPAPI.md
- https://hackerone.com/reports/1140631
- https://hackerone.com/reports/745171
- https://hackerone.com/reports/220774
- https://medium.com/techiepedia/api-exploitation-business-logic-bug-c176d9df47ee
- https://github.com/z5jt/API-documentation-Wordlist/






