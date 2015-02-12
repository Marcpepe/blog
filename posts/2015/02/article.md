# Dealing with third-party proxy

If you've ever developed web applications for a large company, you must be familiar with having authentication done by a third-party proxy. And by third-party I mean handled by another team.

In my case, this is what our architecture looks like:



## So, how's that the problem ?

It's not necessarily a problem! But sometimes the proxy's behavior isn't what you would have expected. Like when your session expires, you might expect a 401 from this proxy and get a 302 instead.

Even if you can communicate with that team, having to require their help takes time and introduces delays, so generally you try to make do with what you have! However sometimes you may just not have a choice!

In our case, we were having problems with API calls from our cached application :

`XMLHttpRequest cannot load https://auth-server.com?sourceUrl=https%3A%2F%2Fmy-app.com%2Fapi%2Fexample. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin https://my-app.com/api/example is therefore not allowed.` is the chrome console error we kept having. After looking it up, this is what you get when trying to make a cross-domain ajax call.





What's in fact happening is, since our application is cached (via a cache manifest), when trying to use it while our authentication session has expired, we only fire ajax API calls to https://my-app.com/api/example. And not only are those calls blocked (because they are cross-domain), but they cannot be intercepted in your app because - as shown in above figure - the 302 redirection is handled at a browser level. So in the end trying to use your app does nothing: you are not redirected to the authentication server login.

#### What does CORS stand for ?

Wikipedia says:
> Cross-origin resource sharing (CORS) is a mechanism that allows resources (e.g. fonts, JavaScript, etc.) on a web page to be requested from another domain outside the domain from which the resource originated.

By default, you are not allowed to request a resource from another domain via an ajax call. Why is that? To prevent security issues such as cross-site scripting (XSS) attacks.

This is why none of our API calls actually go through. So how do you make an exception and


But wait a minute, if they did would that make a difference



the offline component is making https://my-app.com/api/connection-check calls which are redirected (302) by the proxy to the Authentication server. But since the Authentication server is behind another domain, we are getting cross-domain errors and it seemed that the offline component wasn't able to

So why does the offline component think that we're offline. It is because a 302 response is dealt with at a browser level. So the offline component doesn't actually see the 302 response. And since the cross-domain restriction prevents the redirect to go through, offline never receives any response and considers you're are thus offline!

#### About the 'Access-Control-Allow-Origin' header



## Solution

#### Ideal

We soon understood what context triggered the untimely offline status.


#### Workaround
