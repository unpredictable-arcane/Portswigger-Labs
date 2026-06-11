# Unprotected Admin Functionality with Unpredictable URL

## Lab Description

This lab contains an administrative interface that is not properly protected by access controls. Instead of implementing proper authentication and authorization checks, the application attempts to hide the admin panel by using an obscure and unpredictable URL.

The objective of the lab is to discover the hidden administrative endpoint and use it to access the admin panel and delete the user **carlos**.

---

## Vulnerability Overview

Although the admin panel link is not visible to normal users, the endpoint information is exposed within client-side JavaScript code. Since client-side code is accessible to every user, an attacker can inspect the source code, discover the hidden administrative URL, and directly access privileged functionality.

---

## Methodology

### 1 ~ Initial Application Reconnaissance

Upon opening the application, no login credentials or administrative links were provided. So, I began by exploring the application manually and reviewing available pages and functionality.

The homepage contained only standard navigation options and did not expose any sort of visible administrative functionality.

![ Initial Application View ](/Screenshots/Screenshot-01-Lab-04.png)

---

### 2 ~ Inspecting Client-Side Source Code

Since no obvious entry point was available So, I followed the lab hint and inspected the page source using the browser Developer Tools (basically inspect the page).

While reviewing the HTML document, I discovered a JavaScript block containing logic related to administrative functionality.

**Which is :**

```javascript
var isAdmin = false;

if (isAdmin) {
    var adminPanelTag = document.createElement('a');
    adminPanelTag.setAttribute('href', '/admin-s6ypk2');
    adminPanelTag.innerText = 'Admin panel';
}
```

This revealed two important pieces of information:

1. The application contained an administrator panel.
2. The hidden administrative endpoint was:

```text
/admin-s6ypk2
```

The variable `isAdmin` merely controlled whether the link appeared in the user interface and did not actually protect the endpoint itself.

![ Unprotected admin code ](/Screenshots/Screenshot-03-Lab-04.png)

---

### 3 ~ Identifying the Hidden Administrative Endpoint

After reviewing the JavaScript code, I extracted the hidden administrative path:

```text
/admin-s6ypk2
```

This endpoint was not linked anywhere in the visible application interface but was fully disclosed within the client-side source code.

---

### 4 ~ Direct Access to the Admin Panel

I manually appended the discovered endpoint to the URL:

```text
https://<LAB-ID>.web-security-academy.net/admin-s6ypk2
```

The request successfully executed and opens up the administrator interface without requiring any authentication.

This confirmed that the administrative functionality was completely exposed to unauthenticated users.

![ Unprotected admin Panel ](/Screenshots/Screenshot-05-Lab-04.png)

---

### 5 ~ Enumerating Administrative Actions

Inside the administrator panel, a list of users was displayed.

One of the listed users was:

```text
carlos
```

The interface provided a direct **Delete** action for user management.

---

### 6 ~ Deleting the Target User

Then, I selected the **Delete** option associated with the user **carlos**.
![Carlos Deleted](/Screenshots/Screenshot-05-Lab-04.png)

The application processed the request successfully and removed the account.



---

### 7 ~ Lab Completion

After deleting the target user, the lab marked itself as solved.

This confirmed successful exploitation of the access control vulnerability.

![Lab Completion](/Screenshots/Screenshot-06-Lab-04.png)

---
## Impact

In a real one application, this could result in:

- Unauthorized account deletion.
- Privilege abuse.
- Data manipulation.
- Administrative takeover.
- Full compromise of business operations.

---

## Security Recommendations

### ▣ Implement Server-Side Authorization

Every administrative endpoint should validate whether the user has appropriate privileges before processing requests.

Example:

```text
If user role != Administrator
    Return HTTP 403 Forbidden
```

---

### ▣ Never Rely on Hidden URLs

Administrative functionality should not depend on secret or unpredictable URLs for protection.

Hidden endpoints should always be considered discoverable.

---

### ▣  Enforce Role-Based Access Control (RBAC)

Ensure privileged functionality is accessible only to authorized roles.

Example:

```text
Admin → Full Access
User → Restricted Access
Guest → No Administrative Access
```

---

### ▣ Avoid Exposing Sensitive Routes in Client-Side Code

Like:

- JavaScript files
- HTML comments
- Hidden form fields
- Client-side variables

---

### ▣ Perform Authorization Checks on Every Request

Authorization must be enforced on the server regardless of how a request is generated.

---


## Conclusion

So, Overall from a macro level perspective,this application exposed a hidden administrative endpoint within client-side JavaScript code. By inspecting the source, the administrative URL `/admin-s6ypk2` was discovered and accessed directly without authentication. The lack of server-side authorization, enabling deletion of the user **carlos** and successful completion of the lab.

