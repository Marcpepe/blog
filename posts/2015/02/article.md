# Dealing with third-party proxy

This last week, my team and I were faced with a mysterious behavior in our application

## So, what's the problem ?

what you have to know about our application is that:
- it's build on a full-javascript stack (nodejs, ...)
- we use a component called [offline](https://github.com/hubspot/offline), as our application requires to be partly operational offline

| About [offline](https://github.com/hubspot/offline) :
It is an open-source javascript package with no dependencies, which helps enhance your user's experience by detecting whether they're connected to the internet or not, and in the latter case, can perform some actions to enable a minimum usage of your app. The user, once offline, can still use your app because it has been cached.

The issue we were having was that at times we would appear offline while in fact we weren't.

## Diagnostic

We identified pretty quickly that it was an authentication problem.

If you've ever developed web applications for a large company, you must be familiar with having authentication done by a third-party proxy. And by third-party I mean handled by another team.

Here what we we're working with :








Each time we would wrongly appear offline, we'd get this error in chrome console : `XMLHttpRequest cannot load https://auth-server.com?sourceUrl=https%3A%2F%2Fmy-app.com. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin https://my-app.com is therefore not allowed.`. Additionnaly, we were seeing 302 responses.

What's happening is, since our application is cached (the client code is in a cache manifest in the browser), when our session expires at the proxy, the offline component is making https://my-app.com/api/connection-check calls which are redirected (302) by the proxy to the Authentication server. But since the Authentication server is behind another domain, we are getting cross-domain errors and it seemed that the offline component wasn't able to

So why does the offline component think that we're offline. It is because a 302 response is dealt with at a browser level. So the offline component doesn't actually see the 302 response. And since the cross-domain restriction prevents the redirect to go through, offline never receives any response and considers you're are thus offline!

#### About the 'Access-Control-Allow-Origin' header



## Solution

#### Ideal

We soon understood what context triggered the untimely offline status.


#### Workaround

Even if you can communicate with that team, having to require their help takes time and introduces delays, so generally you try to make do with what you have! However sometimes you may just not have a choice!
