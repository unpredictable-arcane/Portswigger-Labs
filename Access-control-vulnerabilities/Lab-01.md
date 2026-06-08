# User ID Controlled by Request Parameter


# Overview

This lab demonstrates an **Insecure Direct Object Reference (IDOR)** vulnerability caused by inadequate access control over a user identifier supplied through a URL parameter.

---

# Objective

The objective of this lab was to:

1. Log in using the provided credentials.
2. Access the account page.
3. Identify how the application references user accounts.
4. Manipulate the user identifier in the URL.
5. Retrieve Carlos's API key.
6. Submit the API key to solve the lab.

---

# Understanding the Vulnerability

The application stores user account information on a page that uses a URL parameter to identify the account being viewed.

Example:

```http
/my-account?id=wiener
```

Instead of validating whether the authenticated user is allowed to access another user's account, the application simply returns the account information associated with the supplied identifier.

As a result, changing the parameter value allows unauthorized access to other users' data.

---

# Initial Authentication

The assessment began by logging into the application using the credentials provided by the lab.

### Credentials

```text
Username: wiener
Password: peter
```

After successful authentication, the application redirected to the account page.

### Observations

The page displayed:

- Username
- API Key
- Email update functionality

The URL contained the following parameter:

```http
/my-account?id=wiener
```

### Screenshot

![Wiener Account Page](/Screenshots/Screenshot-01-Lab-01.png)

---

# Parameter Analysis

While reviewing the account page, it was observed that the user identifier was directly controlled through the URL parameter.

Current request:

```http
GET /my-account?id=wiener HTTP/1.1
```

This suggested that the application relied entirely on the `id` parameter to determine which account data should be returned.

No additional validation appeared to be present.

---

# Parameter Manipulation

To test for an access control weakness, the value of the `id` parameter was modified from:

```http
id=wiener
```

to:

```http
id=carlos
```

Modified URL:

```http
/my-account?id=carlos
```

The request was sent directly through the browser.

### Screenshot

![Modified User ID](/Screenshots/Screenshot-02-Lab-01.png)

---

# Unauthorized Access to Carlos's Account

After modifying the parameter value, the application returned account information belonging to Carlos.

The page displayed:

```text
Username: carlos
API Key: <Carlos API Key>
```

This confirmed that the server was not enforcing authorization checks and was trusting user-supplied input to determine which account information should be displayed.

---

## API Key Retrieval

### Retrieved API Key

```text
6IRiYe4YFMBrEjFgS51UjmdnFoUFUUd4
```

The submission was accepted and the lab was marked as solved.

### Screenshot

![Lab Solved](/Screenshots/Screenshot-03-Lab-01.png)

---

# Technical Analysis


## Attack Flow

```text
Login as Wiener
        ⥥
Access My Account Page
        ⥥
Observe URL Parameter
        ⥥
Change id=wiener
        ⥥
Change to id=carlos
        ⥥
Access Carlos Account
        ⥥
Retrieve API Key
        ⥥
Submit API Key
        ⥥
Lab Solved
```

---

# Remediation

## Enforce Server-Side Authorization

The application should verify ownership of resources before returning data.

Example:

```php
if ($requestedUser != $authenticatedUser) {
    denyAccess();
}
```

---

## Use Session-Based Identity

Instead of trusting user-supplied identifiers:

```http
/my-account?id=wiener
```

The server should derive the user identity directly from the authenticated session.

Like:

```php
$currentUser = $_SESSION['user'];
```

---

## Implement Access Control Checks

Every request accessing sensitive data should verify:

- Authentication
- Authorization
- Resource ownership

---

# Conclusion

The application relied on a user-controlled request parameter to determine which account information should be displayed.

By modifying the value of the `id` parameter from `wiener` to `carlos`, it was possible to access another user's account information without authorization. The exposed API key was retrieved and submitted, successfully solving the lab.

So, basically this lab demonstrates a classic Insecure Direct Object Reference (IDOR) vulnerability resulting from insufficient access control trust in client-supplied identifiers.