---
title: App/Server communication with Same-origin security policy
date: 2018-01-01 12:00:03 Z
categories:
- API
- javascript
- MVC
- MySQL
- PHP
- Silex
- Gotchas
layout: default
---

Few days ago, I had a problem with a mobile app that I’m working on. The app includes a credit card payment which required what’s called 3d secure authentication, that’s a code that gets sent to your mobile to confirm it’s really you who’s trying to make a payment using your credit card.

Problem is, the 3d secure page is located on a secure server that belongs to the card issuer, that – once completed – redirects back to a callback page on our server and we had to display it in an iframe inside the app then detect once the operation has completed to confirm if the user has paid or not, however, You can’t access an <iframe> with Javascript. For the same-origin policy browsers block scripts trying to access a frame with a different origin. Bear in mind that the app code is running in the file:// protocol, while the callback page is loaded from our https web server.

Origin is considered different if at least one of the following parts of the address isn’t maintained:

```<protocol>://<hostname>:<port>/path/to/page.html ```
“Protocol”, “hostname” and “port” must be the same of your domain, if you want to access a frame content.

Solution
Since I had access to both the page and app, I worked around this problem using window.postMessage and its relative message event to send messages between the page and app, like this:

In the iframe:
```parent.postMessage(Status, '*'); // Status is the value that should be sent to the app```
In the app:
```window.addEventListener('message', function(event) { 
    // Checking for the origin of the data
    if (event.origin.indexOf('http://mysite.com')) { 
        // The data has been received from my site 
        // Data sent with postMessage is passed as event.data 
        console.log(event.data); 
    } 
    return; // Data wasn't sent from my site, do nothing 
});``` 
This method can be used in both directions, creating a listener in the iframe  too, and receiving responses from the main page.
