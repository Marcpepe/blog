# Dealing with third-party proxy

If you've ever developed web applications for a large company, you must be familiar with having authentication done by a third-party proxy. And by third-party I mean handled by another team.

In my case, this is what our architecture looks like:



## So, how's that the problem ?

It's not necessarily a problem! But sometimes the proxy's behavior isn't what you would have expected. Like when your session expires, you might expect a 401 from this proxy and get a 302 instead.

Even if you can communicate with the authentication team, having to require their help takes time and introduces delays, so generally you try to make do with what you have! However sometimes you may just not have a choice!

In our case, we were having problems with API calls from our cached application :

```
XMLHttpRequest cannot load https://auth-server.com?sourceUrl=https%3A%2F%2Fmy-app.com%2Fapi%2Fexample. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin https://my-app.com/api/example is therefore not allowed.
```
is the chrome console error we kept having. After looking it up, this is what you get when trying to make a cross-domain ajax call.


#### What does CORS stand for ?

Wikipedia says:
> Cross-origin resource sharing (CORS) is a mechanism that allows resources (e.g. fonts, JavaScript, etc.) on a web page to be requested from another domain outside the domain from which the resource originated.

By default, you are not allowed to request a resource from another domain via an ajax call. Why is that? To prevent security issues such as cross-site scripting (XSS) attacks.

This is why none of our API calls actually go through.






What's in fact happening is, since our application is cached (via a cache manifest), when trying to use it while our authentication session has expired, we only fire ajax API calls to https://my-app.com/api/example. Unfortunately those calls are blocked and you thus never get redirected in your browser to the login page.  

And not only are those calls blocked, but they cannot be intercepted in your app because - as shown in above figure - the 302 redirection is handled at a browser level.

So in the end trying to use your app does nothing: you are not redirected to the authentication server login.


#### How to allow cross-domain calls

Calling an asset via an ajax call is possible only if the domain which hosts that asset allows it. You enable it by adding a header. So this means that in our case,  
`Access-Control-Allow-Origin: https://my-app.com`

For the record, this header comes along with 3 others, which help you narrow down the rule to your specific need :

```
Access-Control-Allow-Methods: POST, GET, HEAD, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER
Access-Control-Max-Age: 1728000
```

## Solution

#### Ideal

Let's stay pragmatic here! The best solution is to ask the authentication team to change the proxy's response from 302 to 401. This way, you can easily detect when your API calls fail and 'manually' redirect to your authentication login page.

However, the authentication team may not be able to comply with that need. For instance, if they have other teams excepting a 302 and cannot work on a case-by-case basis.


#### Workaround

If you have to stick with the 302, here's what you can ask. First ask if a `Access-Control-Allow-Origin` header can be added to the authentication server. This way you won't get cross-domain errors and will be able to if not perfectly at least intercept something that will let you know when to have to force a login. If that's not possible (adding headers to the proxy's response may need security clearance and take a lot of time), a hack would be to ask the authentication team to create a static blank page and add the adequate `Access-Control-Allow-Origin` header only for that single page, then redirect your unauthenticated API calls to that page.

## Bonus

If you're using a cache manifest, you may have experienced that you randomly get redirected to that file on first login. It's pretty annoying and tricky to understand why.

Your application may be cached, but on every page request, your html template sends a request to know if the cache manifest is still up to date. IF you're not authenticated, you will be redirected to your login page, but the request to the cache manifest is also stacked. Once you've correctly logged in, 2 redirect responses are fired simultaneously : the one from your login taking you to your app's main page as expected AND the one from the cache manifest. This is how
