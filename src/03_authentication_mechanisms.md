# Authentication Mechanisms

---

# Authentication answers the following question:<br /><br />Are They Really Who They Say They Are?

---

# Agenda

* HTTP Basic
* HTTP Token
* HTTP Digest
* Form Based
* Web Services Security
* OpenID
* Other Authentication Mechanisms

---

# HTTP Basic Authentication

---

# HTTP Basic Authentication

The oldest and **most common format for web authentication**.

Requests are authenticated in the form of the user’s username and password:

    !text
    https://username:password@www.example.org/

**Low security**, but can be reinforced through the use of SSL/TLS.

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

    !text
    WWW-Authenticate: Basic realm="insert realm"

<br>
<blockquote class="no-before-icon">
    <p>Q: What does "realm" standfor?</p>
    <p>A: <a href="http://en.wikipedia.org/wiki/Realm_(disambiguation)">http://en.wikipedia.org/wiki/Realm_(disambiguation)</a></p>
</blockquote>

---

# HTTP Basic - Client

When the client wants to send the server authentication credentials it may use
the `Authorization` header:

    !text
    Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ=

The `Authorization` header is constructed as follows:

* `username` and `password` are combined into a string `username:password`;
* the resulting string literal is then encoded using **Base64**;
* the authorization method and a space i.e. `"Basic "` is then put before the
encoded string.

---

# Apache And The `.htaccess` File

Apache can be configured through a `.htaccess` file that allows to:

* restrict access to a directory or a file;
* rewrite URLs;
* display custom error pages.


### Example

    !text
    AuthUserFile /path/to/.htpasswd
    AuthName "Restricted Area"
    AuthType Basic
    require valid-user

Add username/password using the following command:

    !text
    $ htpasswd -c .htpasswd username

---

# HTTP Token Authentication

---

# HTTP Token Authentication

Requests are authenticated through a **token**.

Each user possesses a **unique token**, retrievable on that user’s settings
page. Rather than entering username/password information, **users just key in
their token**.
Most of the time, this token will be passed in the query string of an URL:

    !text
    http://example.org/?token=3dZR23REFERGfe

But you can also use the `Authorization` header:

    !text
    Authorization: Token token="3dZR23REFERGfe"

**Low security**, but requires a more active role from the user. Security can
also be reinforced through SSL/TLS.
It can be useful when you need to secure a private API, because it is really
**easy to implement** such a mechanism.

---

# HTTP Digest Authentication

---

# HTTP Digest Authentication

HTTP Digest authentication ([RFC 2069](http://tools.ietf.org/html/rfc2069), and
[RFC 2617](http://tools.ietf.org/html/rfc2617)) **encrypts the password using a
one-way hash** so it's impossible to reverse-engineer even if SSL is not used.

It is an **application of MD5 cryptographic hashing with usage of nonce values to
prevent replay attacks**.

However, it still must be implemented by the browser and can't be designed into
a nice UI.

---

# The HTTP Digest Dance (1/2)

A client requests a page:

    !text
    GET /skynet.html HTTP/1.0
    Host: localhost

The server responds with the `401 Unauthorized` response code, providing the
authentication `realm` and a **randomly-generated**, **single-use value**
called a `nonce`:

    !text
    HTTP/1.0 401 Unauthorized
    Server: HTTPd/0.9
    Date: Sat, 19 Apr 2011 20:11:00 GMT
    WWW-Authenticate: Digest realm="a realm",
                             qop="auth,auth-int",
                             nonce="dcd90f0e8b11d0f600bfb0c093",
                             opaque="5cccebaf9f0171e9517f40e41"
    ...

At this point, the browser will present the authentication realm to the user
and **prompt for a username and password**.

---

# The HTTP Digest Dance (2/2)

Once a username and password have been supplied, the **client re-sends the same
request** but adds an **authentication header** that includes the **response
code**:

    !text
    GET /skynet.html HTTP/1.0
    Host: localhost
    Authorization: Digest username="sarah",
                          realm="a realm",
                          nonce="dcd90f0e8b11d0f600bfb0c093",
                          uri="/skynet.html",
                          qop=auth,
                          nc=00000001,
                          cnonce="0a4f113b",
                          response="6629ae49393a05397450974ef1",
                          opaque="5cccebaf9f0171e9517f40e41"

The server accepts the authentication and the page is returned:

    !text
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

# HTTP Digest Authentication In Picture

<br />

![](../images/http-digest.png)

---

# Form Based Authentication

---

# Form Based Authentication

It refers to the notion of a user being presented with an editable **form to
fill in and submit** in order to log in to some system or service.

As a user, you visit a web page that **prompts you for a username and password**.
If authentication is successful, then under the covers you are granted a unique
cookie, which your web browser sends with each subsequent request.

It just looks like you logged in and now the site works, but under the covers it
is quite different from _Basic_.

Credentials are **sent in plaintext** if you don't rely on SSL to create an
encrypted tunnel between the client and server.

---

# The Form Authentication Flow

<br />

![](../images/form_authentication.png)

<center><small>Source: [Single Sign On (SSO) for cross-domain ASP.NET
applications](http://www.codeproject.com/Articles/106439/Single-Sign-On-SSO-for-cross-domain-ASP-NET-applic).</small></center>

---

# Web Services Security

---

# Web Services Security

**W**eb **S**ervices **SE**curity (WSSE) also known as WSS or WS-Security is a
family of open security specifications for web services, specifically SOAP web
services.

Authentication can be done using **security tokens**.

WS-Security allows you to use any security token you like to use. There are
two different options:

* **username/password authentication** also known as **WSSE Username Token**;
* **binary authentication tokens**, also know as **WSSE Binary Token**  in the
form of _Kerberos_ tickets or _X.509_ certificates;

In addition, **custom binary security tokens** can be applied.

---

# WSSE Username Token

The **Username Token algorithm** is not SOAP-specific.

The benefits of WSSE Username Token are:

* Username / Password encryption;
* Safe guarding against replay attacks;
* No web server configuration required.

---

# WSSE Username Token

The client requests a page:

    !text
    GET /somewhere HTTP/1.1

The server returns a `401 Unauthorized` status code:

    !text
    HTTP/1.1 401 Unauthorized
    WWW-Authenticate: WSSE realm="foo", profile="UsernameToken"

The client sends the same request with his authentication credentials:

    !text
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

.fx: no-border

---

# OpenID

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
user's identity. OpenID authentication is now used and provided by several
large websites.

Providers include _Google_, _Yahoo!_, _PayPal_, _BBC_, _MySpace_, _Steam_, etc.

---

# In-Depth With OpenID

An **end-user** is the entity that wants to assert a particular identity.

A **Relying Party** (RP) is a web site or application that wants to verify the
end-user's identifier.

An **identity provider**, or **OpenID Provider** (OP) is a service that
specializes in registering OpenID URLs or e**X**tensible **R**esource
**I**dentifiers (XRIs).

OpenID enables an end-user to communicate with a relying party.  This
communication is done through the **exchange of an identifier or OpenID**, which
is the URL or XRI chosen by the end-user to name the end-user's identity.

An Identity provider provides the OpenID authentication.

<blockquote class="no-before-icon">
    <i class="fa fa-lightbulb-o"></i>
    <p>eXtensible Identifier Resources are a new form of Internet identifier designed
    specifically for cross-domain digital identity.</p>
</blockquote>

---

# OpenID Overview

<br />
<br />

![](../images/openid_overview.jpg)

<center><small>Source: [OpenID and Rails: Authentication 2.0](http://www.devx.com/opensource/Article/37692).</small></center>

---

# The OpenID Flow

Assuming a user wants to access his account on `example.com`.

The application `example.com` (the **Relying Party**) asks the user for his
**OpenID**.

User enters his OpenID.

Then, `example.com` **redirects** the user **to his OpenID provider**.

User authenticates himself to the **OpenID provider**.

OpenID provider **redirects the user back** to `example.com`.

Finally, `example.com` allows the user to access his account.

---

# The OpenID Flow

![](../images/openid_flow.jpg)

<center><small>Source: [OpenID and Rails: Authentication 2.0](http://www.devx.com/opensource/Article/37692).</small></center>

---

# Why Should You Use OpenID?

### Faster And Easier To Sign Up

An OpenID is **a way of identifying yourself no matter which web site you
visit**. It's like a **driver's license for the Internet**.

### Faster And Easier To Sign In

You only have to remember **one username and one password**.  You might already
use one username and one password online, but OpenID lets you do this in a
**secure way**. That's because **you only give your password to your OpenID
provider**, and then your provider tells the websites you're visiting that you
are who you say you are.

### Closer To A Unified Web Identity

Once you establish yourself as the person who uses a particular OpenID, whenever
someone sees your OpenID in use, anywhere on the Internet, they will know that
it's you.

---

# Other Authentication Mechanisms

---

# Kerberos

Kerberos is a **secure method for authenticating a request for a service in a
computer network**.

Kerberos protocol messages are **protected against eavesdropping** and **replay
attacks**. It builds on **symmetric key cryptography** and **requires a trusted
third party**.

Kerberos lets a user request an encrypted _ticket_ from an authentication
process that can then be used to request a particular service from a server. The
user's password does not have to pass through the network.

When the Kerberos server is **down**, **no one can log in**.

---

# How Does Kerberos Work? (1/2)

Suppose you want to access a server on another computer. You know that this
server requires a Kerberos "ticket" before it will honor your request.

![](../images/kerberos.jpg)

---

# How Does Kerberos Work? (2/2)

To get your ticket, you first **request authentication** from the
**Authentication Server** (AS).

The Authentication Server creates a **session key** basing it on your password
and a random value that represents the requested service. The session key is a
**ticket-granting ticket**.

You next send your ticket-granting ticket to a **Ticket Granting Server** (TGS).
The TGS returns the ticket that can be sent to the server for the requested
service.

The service either **rejects the ticket or accepts it** and performs the service.

Because the ticket you received from the TGS is time-stamped, it allows you to
make additional requests using the same ticket within a certain time period
without having to be reauthenticated.

---

# Remote Authentication Dial-In User Service (RADIUS)

A **networking protocol** that **provides centralized Authentication**,
Authorization, and Accounting (AAA) management for computers to connect and use
a network service.
RADIUS is a client/server protocol that runs in the application layer, using UDP
as transport. It is described in [RFC 2865](http://www.ietf.org/rfc/rfc2865.txt).

In RADIUS, **authentication and authorization are coupled together**. If the
username is found and the password is correct, the RADIUS server returns an
`Access-Accept` response, including a list of attribute-value pairs that describe
the parameters to be used for this session.

![](../images/radius_auth.gif)
