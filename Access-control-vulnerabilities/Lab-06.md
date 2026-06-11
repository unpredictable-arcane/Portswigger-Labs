# User Role Controlled by Request Parameter

## Lab Description

This lab contains an administrator panel located at "/admin". And, this application determines whether a user is an administrator using a client-controlled request parameter stored within a cookie.

---

## Vulnerability Overview

The application stores authorization information inside a user-controllable cookie.

Instead of validating administrative privileges on the server side, the application trusts the value of an `Admin` cookie supplied by the client.

---

## Objective

Gain administrative access by manipulating the request parameter controlling user roles and delete the user **carlos**.

---

## Methodology

### 1 ~ Logging into the Application

The lab provided the following credentials:

```text
Username: wiener
Password: peter
```

I authenticated using the supplied credentials and accessed the user account page.


![ Logged In as Wiener ](/Screenshots/Screenshot-01-Lab-06.png)

---

### 2 ~ Reviewing the Lab Description

The lab description stated that:

```text
The admin panel is located at /admin
and administrators are identified using a forgeable cookie.
```

which directly suggests that the authorization mechanism relied on a user-controlled value rather than proper server-side role validation.

Based on this hint, I decided to inspect requests using Burp Suite.

---

### 3 ~ Intercepting the Request

I enabled Burp Suite interception and browsed to:


**/admin**

The request was captured successfully.

---

### 4 ~ Identifying the Authorization Cookie

While reviewing the intercepted request, I observed the following cookie:

```http
Cookie: Admin = false
```

The application appeared to determine administrative privileges solely based on this cookie value.

which is :

```http
GET /admin HTTP/1.1
Host: target-site

Cookie: Admin = false
```

### Screenshot

![Admin Cookie Identified](/Screenshots/Screenshot-03-Lab-06.png)

---

### 5 ~ Modifying the Request Parameter

To test whether the cookie controlled authorization, I modified its value from:

```http
Admin = false
```

to:

```http
Admin = true
```

And also I configured Burp Match and Replace rules to automatically replace every occurrence of:

```text
Admin = false
```

with:

```text
Admin = true
```

So that  all requests carried the modified administrative value.


![Cookie Modified to Admin=True](/Screenshots/Screenshot-04-Lab-06.png)

---

### 6 ~ Gaining Administrative Access

After forwarding the modified request, the application granted access to the administrator panel.

The navigation menu now exposed:

**Admin panel**


which was previously unavailable to the standard user account.

This confirmed that administrative privileges were entirely controlled by the client-side cookie.


---

### 7 ~ Enumerating Administrative Functions

Inside the administrator panel, a list of application users was displayed.

The target account:

**carlos**


was visible along with a **Delete** option.

![Carlos User Listed](/Screenshots/Screenshot-05-Lab-06.png)

---

### 8 ~ Deleting the Target User

I selected the **Delete** option associated with the user **carlos**.

---

### 9 ~ Lab Completion

After deleting the user **carlos**, the lab was marked as solved.

This confirmed successful exploitation of the authorization flaw.

![Lab Solved](/Screenshots/Screenshot-06-Lab-06.png)

---

## Root Cause

The application trusted authorization information supplied by the client.

Administrative privileges were determined using a user-controlled cookie:

```http
Admin = true
```

Since cookies can be modified by any user, the application effectively allowed users to assign themselves administrator privileges.

The application failed to:

- Validate roles on the server side.
- Protect authorization data from tampering.
- Enforce proper access control checks.

---

## Security Recommendations

### ◈ Store Roles Server-Side

User roles should be maintained on the server and retrieved from a trusted source.

Example:

```text
User ID → Database Lookup → Role
```

---

### ◈ Never Trust Client-Supplied Authorization Data

Cookies, headers, URL parameters, and hidden form fields should never determine user privileges.

---

### ◈ Enforce Server-Side Authorization

Every administrative request should verify user permissions before processing.

Example:

```text
If role != Administrator
    Return HTTP 403 Forbidden
```

---

### ◈ Use Signed Session Tokens

Authorization data should be cryptographically protected against tampering.

Example:

```text
Session ID ⇒ Server Session Store
```

rather than:

```text
Admin = true
```
---

## Conclusion

The application rlied on a user-controlled cookie to determine administrative privileges. By intercepting a request in Burp Suite and modifying the cookie value from `Admin=false` to `Admin=true`, administrative access was obtained without any legitimate authorization. This exposed the administrator panel and allowed deletion of the user **carlos**, successfully solving the lab.