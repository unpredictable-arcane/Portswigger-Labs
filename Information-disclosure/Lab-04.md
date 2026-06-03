# Authentication Bypass via Information Disclosure


## Objective

The objective of this lab is to gain unauthorized access to the administrator panel by exploiting information disclosure and then delete the user account `carlos`.

---

## Methodology

### Step 1: Login as a Normal User

Logged into the application using the provided credentials:

```text
Username: wiener
Password: peter
```

After successful authentication, normal user privileges were obtained.

---

### Step 2: Attempt to Access the Administrator Panel

Manually browsed to:

```http
/admin
```

The application returned the message:

```text
Admin interface only available to local users
```

### Screenshot

![Access Denied](/Screenshots/Screenshot-01-lab-04.png)

---

### Step 3: Send the Request to Burp Repeater

Then, I captured the request and forwarded it to Burp Repeater for further analysis.

Original request:

```http
GET /admin HTTP/2
Host: target-site
Cookie: session=<session-cookie>
```

The response continued to deny access (as expected).

---

### Step 4: Test Alternative HTTP Methods

Modified the request method from:

```http
GET
```

to:

```http
TRACE
```

Modified request:

```http
TRACE /admin HTTP/2
Host: target-site
Cookie: session=<session-cookie>
```


---

### Step 5: Discover Internal Authorization Header

Carefully reviewing the TRACE response revealed an additional header:

```http
X-Custom-IP-Authorization: 122.180.208.129
```

### Observation

The application appeared to trust requests based on the value contained in the `X-Custom-IP-Authorization` header.

### Screenshot

![TRACE Disclosure](/Screenshots/Screenshot-03-Lab-04.png)

---

### Step 6: Configure Header Injection Using Burp Suite

Navigated to:

```text
Proxy → Match and Replace
```

Created a new Match and Replace rule.

#### Configuration

**Type**

```text
Request Header
```

**Match**

```text
Nothing (empty)
```

**Replace**

```http
X-Custom-IP-Authorization: 127.0.0.1
```

This configuration caused Burp Suite to automatically append the trusted header to all outgoing requests.

### Screenshot

![Match and Replace](/Screenshots/Screenshot-04-Lab-04.png)

---

### Step 7: Revisit the Administrator Panel

Requested the administrator page again:

```http
GET /admin HTTP/2
```

Because the request now contained:

```http
X-Custom-IP-Authorization: 127.0.0.1
```

the server treated the request as originating from localhost.

Administrative access was granted successfully (after attempting through many other ways this finally worked).

### Screenshot

![Admin Panel Access](/Screenshots/Screenshot-05-Lab-04.png)

---

### Step 8: Delete the Target User

Within the administrator interface, the users were :

```text
wiener
carlos
```

Selected:

```text
Delete → carlos
```

The user account was removed successfully.


---

### Step 9: Lab Solved

After deleting the target user, the application confirmed successful completion of the lab.

### Screenshot

![Solved](/Screenshots/Screenshot-06-lab-04.png)

---

## Technical Analysis

### Root Cause

The application exposed sensitive internal authorization details through the TRACE HTTP method.

Like:

- TRACE reflected internal request processing data.
- Internal authorization headers were disclosed.
- Access control relied on a client-supplied HTTP header.

### Attack Flow

```text
Access Denied
      ⥥
Discover Authorization Logic
      ⥥
TRACE Request
      ⥥
Header Disclosure
      ⥥
Spoof Trusted Header
      ⥥
Authentication Bypass
      ⥥
Admin Access
      ⥥
Delete Carlos
```
---

## Security Recommendations

### Disable TRACE Method

Disable unnecessary HTTP methods such as TRACE:

```apache
TraceEnable Off
```

### Avoid Trusting Client Headers

Never rely on client-controlled headers for authorization decisions.

### Restrict Administrative Interfaces

Ensure administrator functionality is accessible only after proper authentication and authorization.

### Remove Information Disclosure

Do not reveal internal security mechanisms through:

- Error messages
- Diagnostic responses
- Debug functionality

---


## Conclusion

The application disclosed an internal authorization header through the TRACE HTTP method. By identifying this mechanism and spoofing the trusted localhost value using Burp Suite Match and Replace rules, administrative access was obtained successfully. This allowed deletion of the target user `carlos`, completing the lab.