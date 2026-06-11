# Unprotected Admin Functionality


## Lab Description

So, this lab contains an administrative interface that is not protected by proper access controls. The administrator panel is discoverable through publicly accessible resources and  directly without even authentication.

---

## Vulnerability Overview

So, the administrator URL is disclosed within the `robots.txt` file, which can be accessed by anyone visiting the website.

Since no authentication or authorization checks are enforced on the administrator endpoint, an attacker can directly access privileged functionality and perform administrative actions.

---

### Objective

Access the administrator interface and delete the user **carlos**.

---

## Methodology

###  1 ~ Initial Application Reconnaissance

Upon opening the application as usual, no login credentials or administrator links were provided.

I began by manually exploring the application and reviewing publicly accessible resources that commonly expose sensitive information.

Since administrative functionality was not visible on the homepage, I decided to enumerate common files that may disclose hidden application paths.


### 2 ~ Accessing robots.txt

As part of basic reconnaissance, I visited the following endpoint:


**/robots.txt**


The response contained the following content:

```text
User-agent: *
Disallow: /administrator-panel
```

Which means, the `robots.txt` file revealed a hidden administrative endpoint that was intended to be excluded from search engine indexing.


![ robots.txt Disclosure](/Screenshots/Screenshot-01-Lab-05.png)

---

### 3 ~ Discovering the Administrative Interface

From the information disclosed in `robots.txt`, I identified the hidden administrator path:


**/administrator-panel**


This immediately suggested the presence of an administrative interface that might be directly accessible.

---

### 4 ~ Direct Access to the Admin Panel

I manually browsed to the discovered endpoint:

```text
https://<LAB-ID>.web-security-academy.net/administrator-panel
```

The request successfully loaded the administrative interface.

No authentication, authorization, or privilege verification was required before accessing the panel.

This confirmed that the administrator functionality was completely unprotected.

---

### 5 ~ Reviewing Administrative Functions

Inside the administrator panel, a list of users was displayed.

One of the users listed was:

**carlos**


The interface exposed administrative management actions, including a **Delete** option for user accounts.

### Screenshot

![Carlos User Listed](/Screenshots/Screenshot-02-Lab-05.png)

---

### 6 ~ Deleting the Target User

I selected the **Delete** option associated with the user **carlos**.

The application processed the request successfully and removed the account.

The action was executed without requiring any authentication or additional authorization checks.And, obviously then after, deleting the user **carlos**, the lab marked itself as solved.

---

### 7 ~ Lab Completion

After deleting the user **carlos**, the lab marked itself as solved.


### Screenshot

![Lab Solved](/Screenshots/Screenshot-03-lab-05.png)


---

## Root Cause

The primary issues were:

1. Sensitive administrative paths were disclosed through `robots.txt`.
2. Administrative functionality lacked authentication controls.
3. Administrative functionality lacked authorization validation.
4. Access control was not enforced on the server side.


---

## Security Recommendations

### 1. Enforce Authentication on Administrative Endpoints

All administrative functionality should require valid authentication before access is granted.

Like:

```text
Unauthenticated User → Redirect to Login Page
Authenticated Administrator → Allow Access
```

---

### 2. Implement Server-Side Authorization

Every administrative request should verify that the authenticated user possesses the required privileges.

Example:

```text
If role != Administrator
    Return HTTP 403 Forbidden
```

---

### 3. Avoid Exposing Sensitive Paths

Administrative endpoints should not be disclosed through:

- robots.txt
- JavaScript files
- HTML comments
- Public documentation

---

### 4. Apply Role-Based Access Control (RBAC)

Restrict privileged functionality based on user roles.

Example:

```text
Administrator → Full Administrative Access
Standard User → No Administrative Access
Guest → No Administrative Access
```

---

## Conclusion

The application exposed a sensitive administrative endpoint through the publicly accessible `robots.txt` file. By reviewing the file, the hidden path `/administrator-panel` was discovered and accessed directly. Due to the absence of authentication and authorization controls, unrestricted access to the administrator panel was obtained, allowing deletion of the user **carlos** and successful completion of the lab.