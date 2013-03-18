# Authentication Mechanisms

---

# Agenda

* HTTP Basic
* Form Based
* HTTP Digest
* WSSE Username Token
* Open ID

---

# HTTP Basic

The oldest and most common format for web authentication.

Requests are authenticated in the form of the user’s username and password:

    https://username:password@www.example.org/

Low security, but can be reinforced through the use of SSL/TLS.

Users can disable API access from third-party apps at any time by changing their
username or password.

HTTP Basic has a big disadvantage in that it's implemented entirely by the
browser, and can't be customized for each site.

---

# HTTP Basic - Server

When the server wants the user agent to authenticate itself towards the server,
it can send a request for authentication.

This request should be sent using the HTTP `401 Unauthorized` response code
containing a `WWW-Authenticate` HTTP header:

    WWW-Authenticate: Basic realm="insert realm"

---

# HTTP Basic - Client

When the client wants to send the server authentication credentials it may use
the `Authorization` header:

    Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==

The `Authorization` header is constructed as follows:

* `username` and `password` are combined into a string `username:password`;
* the resulting string literal is then encoded using **Base64**;
* the authorization method and a space i.e. `"Basic "` is then put before the
encoded string.

---

# Form Based

It refers to the notion of a user being presented with an editable "form" to
fill in and submit in order to log in to some system or service.

As a user, you visit a web page that prompts you for a username and password. If
authentication is successful, then under the covers you are granted a unique
cookie, which your web browser sends with each subsequent request.

It just looks like you logged in and now the site works, but under the covers it
is quite different from Basic

Credentials are sent in clear text if you don't rely on SSL to create an
encrypted tunnel between the client and server.

---

# HTTP Digest

HTTP Digest authentication ([RFC 2069](http://tools.ietf.org/html/rfc2069), and
[RFC 2617](http://tools.ietf.org/html/rfc2617)) encrypts the password using a
one-way hash so it's impossible to reverse-engineer even if SSL is not used.

It is an application of MD5 cryptographic hashing with usage of nonce values to
prevent replay attacks.

However, it still must be implemented by the browser and can't be designed in to
a nice UI.

---

# The HTTP Digest Dance (1/2)

A client requests a page:

    GET /skynet.html HTTP/1.0
    Host: localhost

The server responds with the `401 Unauthorized` response code, providing the
authentication `realm` and a **randomly-generated**, **single-use value**
called a `nonce`:

    HTTP/1.0 401 Unauthorized
    Server: HTTPd/0.9
    Date: Sat, 19 Apr 2011 20:11:00 GMT
    WWW-Authenticate: Digest realm="a realm",
                             qop="auth,auth-int",
                             nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                             opaque="5ccc069c403ebaf9f0171e9517f40e41"
    ...

At this point, the browser will present the authentication realm to the user
and **prompt for a username and password**.

---

# The HTTP Digest Dance (2/2)

Once a username and password have been supplied, the **client re-sends the same
request** but adds an **authentication header** that includes the **response
code**:

    GET /skynet.html HTTP/1.0
    Host: localhost
    Authorization: Digest username="sarah",
                          realm="a realm",
                          nonce="dcd98b7102dd2f0e8b11d0f600bfb0c093",
                          uri="/skynet.html",
                          qop=auth,
                          nc=00000001,
                          cnonce="0a4f113b",
                          response="6629fae49393a05397450978507c4ef1",
                          opaque="5ccc069c403ebaf9f0171e9517f40e41"

The server accepts the authentication and the page is returned:

    HTTP/1.0 200 OK
    ...

---

# The "response" Value

The `response` value is calculated in three steps:

* The **MD5 hash** of the combined `username`, `realm` and `password` is
calculated. The result is referred to as **HA1**;
* The MD5 hash of the combined _method_ and _URI_ is calculated, e.g. of `GET`
and `/skynet.html`. The result is referred to as **HA2**;
* The MD5 hash of the combined `HA1` result, server nonce (`nonce`), request
counter (`nc`), client nonce (`cnonce`), quality of protection code (`qop`) and
`HA2` result is calculated. The result is the `response` value provided by the
client.

Since the server has the same information as the client, the response can be
checked by performing the same calculation.

---

# WSSE Username Token

WSSE is a family of open security specifications for web services, specifically
SOAP web services. However, the Username Token algorithm is not SOAP-specific.

The benefits of WSSE are:

* Username / Password encryption;
* Safe guarding against replay attacks;
* No web server configuration required.


---

# WSSE Username Token

The client requests a page:

    GET /somewhere HTTP/1.1

The server returns a `401 Unauthorized` status code:

    HTTP/1.1 401 Unauthorized
    WWW-Authenticate: WSSE realm="foo", profile="UsernameToken"

The client sends the same request with his authentication credentials:

    Authorization: WSSE profile="UsernameToken"
    X-WSSE: UsernameToken Username="bob",
            PasswordDigest="quR/EWLAV4xLf9Zqyw4pDmfV9OY=",
            Nonce="d36e316282959a9ed4c89851497a717f",
            Created="2003-12-15T14:43:07Z"

The application looks in the `X-WSSE:` header for the actual authentication
credentials, and recreates the steps the client took in order to verify that
the client knows his password.

---

# ![](../images/openid.png)

---

# Open ID

OpenID is a **safe**, **faster**, and **easier** way to log in to web sites.

It is an **open standard** that allows users to be authenticated by certain
co-operating sites (known as **Relying Parties** or RP) using a third party
service.

OpenID was created in the summer of 2005 by an open source community trying to
solve a problem that was not easily solved by other existing identity
technologies.

Users may create accounts with their preferred OpenID identity providers, and
then use those accounts as the basis for signing on to any website which accepts
OpenID authentication.

The OpenID protocol **does not rely on a central authority** to authenticate a
user's identity.

OpenID authentication is now used and provided by several large websites.
Providers include Google, Yahoo!, PayPal, BBC, AOL, MySpace, IBM, Steam, etc.

---

# In-Depth With Open ID

An **end-user** is the entity that wants to assert a particular identity.

A **Relying Party** (RP) is a web site or application that wants to verify the
end-user's identifier.

An **identity provider**, or **OpenID Provider** (OP) is a service that
specializes in registering OpenID URLs or e**X**tensible **R**esource
**I**dentifiers (XRIs).

OpenID enables an end-user to communicate with a relying party.

This communication is done through the **exchange of an identifier or OpenID**,
which is the URL or XRI chosen by the end-user to name the end-user's identity.

An Identity provider provides the OpenID authentication.

> XRIs are a new form of Internet identifier designed specifically for
> cross-domain digital identity.

---

# The Open ID Scenario

Assuming a user wants to access his account on `example.com`.

The application `example.com` (the **Relying Party**) asks the user for his
**OpenID**.

User enters his OpenID.

Then, `example.com` **redirects** the user **to his OpenID provider**.

User authenticates himself to the **OpenID provider**.

OpenID provider **redirects the user back** to `example.com`.

Finally, `example.com` allows the user to access his account.

---

# Why Should You Use OpenID?

### Faster and easier to sign up

An OpenID is a way of identifying yourself no matter which web site you visit.

It's like a driver's license for the entire Internet.


### Faster and easier to sign in

With OpenID you only have to remember one username and one password.

Now, you might already use one username and one password online, but OpenID lets
you do this in a secure way. That's because you only give your password to your
OpenID provider, and then your provider tells the websites you're visiting that
you are who you say you are.


### Closer to a unified "web identity"

Once you establish yourself as the person who uses a particular OpenID, whenever
someone sees your OpenID in use, anywhere on the Internet, they'll know that
it's you.
