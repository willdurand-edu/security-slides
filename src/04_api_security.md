# API Security Recommendations

---

# Use API Keys For Non-Sensitive Data

If you have an **open API**, one that exposes data you’d make public on the
Internet anyway, **consider issuing non-sensitive API keys**.

These are **easy to manipulate** and still **give you a way to identify users**.

Armed with an API key, you have the option of establishing a **quota** for your
API, or at least **monitoring usage by user**.

### Example

The Yahoo Maps Geocoding API issues API keys so it can track its users and
establish a quota, but the data that it returns is not sensitive so it’s not
critical to keep the key secret.

---

# Use Username/Password Auth. For APIs Based On Web Sites

If your **API is based on a web site**, **support username/password
authentication**. That way, **users can access your site using the API the same
way that they access it using their web browser**.

However, if you do this, you may want to **avoid session-based authentication**,
especially if you want people to be able to write API clients in lots of
environments.


### Example

It is very simple to use `curl` for instance, to grab some data from the Twitter
API because it uses HTTP Basic authentication.

It is a lot more complex if I instead have to call _login_ and extract a session ID
from some cookie or header, and then pass that to the real API call I want to make.

---

# Use SSL For Everything <s>Sensitive</s>

Unless your API has only open and non-sensitive data, **support SSL**, and
**consider even enforcing it** (that is, redirect any API traffic on the non-SSL
port to the SSL port).

It makes other authentication schemes more secure, and **keeps your user’s
private API data from prying eyes**.

---

# And ...

1. Use **nonce tokens** (randomly-generated, single-use values) to avoid
   **replay attacks**;

2. Rely on **request signing** to ensure data integrity;

3. Use **strong** methods to generate random values.
