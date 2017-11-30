---
layout: post
title: 'Using Cookies to store Untappd Tokens'
---
# {{ page.title }}
Before switching to server side rendering for Festival Hopper, I was storing a user's Untappd token in Local Storage where it could be easily retreived and used for any of the public API calls to be made. This strategy is both insecure and inneffective when I need to know if a user is logged in from their http request. I'll talk about both of those concerns seperately.

## Cookies for user state
I needed a way of detecting on the server whether the user was logged in or not. Depending on which, I would either provide them with the page they requested, or redirect them to the login page. This was impossible with the current setup because local storage information is not sent to the server along with requests. The solution? Switch to server side authentication and save the token as a cookie, which DO get sent to the server when making requests.

If you are unfamiliar, cookies are simple pieces of information that you can store client-side. Sure, there are much more efficient ways nowadays to store information that way...LocalStorage and SessionStorage sure have that covered, but the nice thing about cookies is that you can send them down in server responses by setting a special header that looks like this:

`Set-Cookie: <cookie-name>=<cookie-value>`

This will create a Session Cookie that will expire once the user closes their browser/tab. In my case, I want them to persist a bit longer than that, so I add a max-age of 1 year, an arbitrary time I chose so that the cookie doesn't have to be set very often. Since I'm using Express, the code looks like this: 

```
// send the cookie and redirect to the curated page
res.cookie('untappd_access_token', bodyObj.response.access_token, {maxAge: 31536000}); // max age = 1 year
res.writeHead(302, {
    Location: '/',
});
res.end();
```

By sending this cookie in an http response, it is automatically stored in hte user's browser and sent with any http requests to my server that the client makes. My server can simply check for the existance of the cookie to determine the login status of the user. Then they will be directed to the appropriate page of the app.

## Security Concerns
As I was researching cookies and ways of using them to determine the login status of users, I came across a lot of information about web security and went down a bit of a rabbit hole. What I learned doesn't much affect the current version of Festival Hopper, but will be very important if I decide to add more robust services to it in the future. There are 2 types of attacks that an application needs to be careful about. They are XSS (Cross Site Scripting) and CSRF (Cross Site Request Forgery) attacks.

### XSS Attacks
An XSS attack occurs when a malicious person is able to execute code on a user's session. They come in a few forms: reflected, stored, and DOM based.

Reflected: An injected script is 'reflected' off a web server. A user is clicks on the link that directs them to a vulnerable site. The site shows an error that contains some of the provided content from the request link and there is a malicious script hidden in there. So the server basically reflected the malicious script back to the user and executed it.

Stored Attack: An injected script is stored on a victim's servers and served to clients. For example, a malicious user types a script into a comments section of a blog and submits that to the server. If it isn't escaped, then that script has the potential to execute on all users who visit the page that contains that comment.

DOM: A malicious script is injected client-side, as opposed to server side, which is where the other 2 methods occur.


### XSS Defenses
The golden rule here is to to sanitize and escape any input that could possibly be untrustworthy before including it in your app, especially input that comes directly from a user (such as form values and comments sections). When an app saves user input, it must make sure that the contents are escaped and read as text, and not as script that can be executed on the server or database. Same thing with content on the web page. For example, text that is rendered in a comments section should be escaped so that the contents are not read as executable script, only as text.

The only information I use for the user is their Untappd token and I don't want anyone to be able to get a user's token and use it for whatever they want. Untappd doesn't really handle any sensitive user information, so the damage a person can do is trivial (delete some checkins maybe?), but trolls on the internet are never above messing with peoples accounts simply for the thrill of it. Because of that, I want to make sure no unwanted person can access this token.

So, are we safe from XSS attacks? First, I need to check if I am taking input from anywhere and storing it. If so, it needs to be escaped. Currently there is nowhere in the app where I take user input and run it through my server or database. Beers have an input field, but that information goes directly to Untappd, so it is their responsibility to escape that input. So from this attack point, the app is fine. However, I will have to keep this in mind if I ever add the ability for users to create their own lists, name them, comment on lists, etc.

Second, I need to check that I am not including untrusted input in my html. The only outside information that is rendered on a page is information from Untappd. I trust them, but you can never be too careful. The good news is, since I'm using React to render this content, I should be all set. React automatically escapes everything to prevent XSS attacks.

So, all in all, it looks like my app should be safe from XSS attacks. That is, as long as I make sure that I sanitize any future server features and escape any input that I display without React's help. Next I'll talk about CSRF attacks.

### CSRF Attacks
A CSRF attack occurs when a malicious user is able to make a request to another site on the behalf of a user, and perform an action on that site as if they were that user.

Example: A user visits a web page with a malicious script on it. The script sends a request to a banking website. The bank website uses cookies to automatically log people in. This user has an account with this band and their cookie is sent automatically with their request. The request targets an endpoint that performs an action, `such as www.bank.com/sendmoney?sendmoneyto=evilperson` and now the malicious user has just been sent a bunch of money.

### CSRF Defenses
Currently my web app does not perform any actions through any endpoints, which means I'm fine regarding CSRF. Nonetheless, its good to know about how to defend myself once I start adding in more complicated features like user accounts and the ability to create your own lists of beers. Here are the different ways you can defend yourself (summarized from this great article: [owasp csrf](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet)) 

1. Synchronizer (i.e.,CSRF) Tokens (requires session state)
Server generates a cryptographic token, stores it server side, then sends it to the client, where it is saved inside a hidden field. When any http request is made, the token must match the one saved on the server. Lack of a token means that the request did not originate from someone directly using the app.

2. Double Cookie Defense
Send a randomly generated value as a cookie and also save it within a hidden csrf field. When a request is made, you can make sure both of these exist and that they match before allowing any actions to be made, thus ensuring that the request came from a user working directly in the app.

3. Encrypted Token Pattern
Encrypt a token and send it to the user where it is saved as a hidden csrf field. Include this field in future http requests and the server will decrypt and verify the information thats inside. Requests from outside the app won't include this token and even if a malicious user somehow includes one, the data includes timestamps and other information that can be used to determine if the request is forged.

4. Custom Header - e.g., X-Requested-With: XMLHttpRequest
This is one of the simplest methods of thwarting CSRF attacks and is the method I will use if I add more robust features to the web app in the future. Features that perform actions through the use of server endpoint. The gist is that whenever you make an AJAX call through your application, you set a custom http header on the request that the server can look for and verify. This works because only javascript can be used to create a custom header, and only within its own origin, or else it would violate the same origin policy.