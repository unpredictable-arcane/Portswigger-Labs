# User ID Controlled by Request Parameter, with Unpredictable User IDs


# Overview

This lab demonstrates an **Insecure Direct Object Reference (IDOR)** vulnerability where user accounts are referenced using seemingly unpredictable identifiers instead of predictable usernames.

Although the application does not expose usernames directly through the account page URL, it still relies on user-controlled identifiers to determine which account information should be returned.

---

# Objective

The objective of this lab was to:

1. Log in using the provided credentials.
2. Identify how user accounts are referenced.
3. Discover Carlos's unique user identifier.
4. Modify the account page request.
5. Access Carlos's account information.
6. Retrieve Carlos's API key.
7. Submit the API key to solve the lab.

---

# Understanding the Vulnerability


here, usernames have been replaced with unpredictable identifiers:

```http
/my-account?id=30f92ad7-bc52-4ddb-a28c-47da82c51f4d
```

At first glance, this appears more secure because the identifiers are difficult to guess.

---

# Initial Authentication

So, the assessment began by logging into the application using the credentials provided by the lab.

### Credentials

```text
Username: wiener
Password: peter
```

### Observations

The URL contained a UUID-like identifier rather than a username.

Example:

```http
/my-account?id=30f92ad7-bc52-4ddb-a28c-47da82c51f4d
```

### Screenshot

![Wiener Account Page](/Screenshots/Screenshot-01-Lab-02.png)

---

# Reconnaissance and Blog Enumeration

Now, the next step was to search for locations where user identifiers might be disclosed.

The application's blog section was reviewed and several blog posts were examined.

While navigating through blog content, a post authored by **Carlos** was discovered (Pretty weird blog ngl).

### Screenshot

![Carlos Blog Post](/Screenshots/Screenshot-02-lab-02.png)

---

# Discovery of Carlos's Identifier

Upon examining the URL associated with Carlos's blog post, a unique identifier was observed within the request parameter.

Example:

```http
/blogs?userId=f80515d5-a1ed-4c97-8009-f6d1206c9913
```

This appeared to represent Carlos's account.

### Observation

Although the application attempted to hide user identities using UUIDs, it unintentionally exposed those identifiers through publicly accessible content.

---

# Parameter Manipulation

After obtaining Carlos's identifier, the account page URL was modified.

Original URL:

```http
/my-account?id=30f92ad7-bc52-4ddb-a28c-47da82c51f4d
```

Modified URL:

```http
/my-account?id=f80515d5-a1ed-4c97-8009-f6d1206c9913
```

The modified request was submitted directly through the browser.


---

# Unauthorized Access to Carlos's Account

The application responded with Carlos's account information despite being authenticated as Wiener.

The page displayed:

```text
Username: carlos
API Key: SRmyP0uREDKeILKF8wje7Z52Vaw9ScGJ
```

This confirmed that authorization checks were not being enforced.

The application trusted the user-supplied identifier and returned data belonging to another user.

---

# API Key Retrieval

The API key exposed on Carlos's account page was copied and submitted through the lab interface.

### Retrieved API Key

```text
SRmyP0uREDKeILKF8wje7Z52Vaw9ScGJ
```

The submission was accepted and the lab was successfully completed.

### Screenshot

![Lab Solved](/Screenshots/Screenshot-03-Lab-01.png)

---

# Technical Analysis



---

## Root Cause

The vulnerability exists because:

1. User accounts are identified through client-supplied parameters.
2. The server does not validate ownership of the requested resource.
3. UUIDs are treated as authorization controls.
4. Access decisions rely solely on user input.

While UUIDs make identifiers more difficult to guess, they do not provide actual access control.

Once a valid identifier is discovered, the vulnerability becomes exploitable.

---


# Attack Flow

```text
Login as Wiener
        ⥥
Access Account Page
        ⥥
Observe UUID-Based Identifier
        ⥥
Browse Public Blog Posts
        ⥥
Find Carlos's Blog
        ⥥
Extract Carlos's User ID
        ⥥
Replace UUID in Account URL
        ⥥
Access Carlos's Account
        ⥥
Retrieve API Key
        ⥥
Submit API Key
        ⥥
Lab Solved
```

---

# Remediation

## ⇒ Enforce Server-Side Authorization

The server must validate whether the authenticated user is permitted to access the requested account.

Example:

```php
if ($requestedAccount != $authenticatedUser) {
    denyAccess();
}
```

---

## ⇒ Do Not Rely on UUIDs for Security

Using just random identifiers does not eliminate IDOR vulnerabilities.

Example:

```http
/my-account?id=f80515d5-a1ed-4c97-8009-f6d1206c9913
```

UUIDs should only serve as identifiers, not authorization mechanisms.

---

## ⇒ Derive Identity from the Session

Instead of trusting user-controlled parameters:

```http
/my-account?id=UUID
```

The application should retrieve account information directly from the authenticated session.

Example:

```php
$currentUser = $_SESSION['user'];
```

---

## ⇒ Follow the Principle of Least Privilege

Users should only be able to access resources that belong to their own account.

---

# Conclusion

The application attempted to protect user accounts by replacing usernames with unpredictable UUID-based identifiers. However, Carlos's identifier was exposed through publicly accessible blog content. 

This lab demonstrates that unpredictable identifiers alone do not prevent IDOR vulnerabilities. Proper server-side authorization checks are required to ensure users can only access resources they are authorized to view.