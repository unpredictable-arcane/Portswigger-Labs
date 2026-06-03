# Lab: Information Disclosure on Debug Page

## Lab Overview

This lab demonstrates how information disclosure vulnerabilities can occur when debugging functionality is left accessible in a production environment. 

The objective of this lab was to discover a secret key exposed through a debug page and use it to solve the challenge.

---

## Vulnerability Description

Information disclosure occurs when an application unintentionally exposes sensitive data to unauthorized users (common users sometimes).

In this lab, a publicly accessible debug page disclosed server-side environment variables. Among the exposed information was a secret key that should never have been accessible to external users.

---

## Reconnaissance and Enumeration

The first step was to perform content discovery to identify hidden directories and files that were not linked from the application's user interface.

For this purpose, I used **Feroxbuster** along with a **SecLists** wordlist.

### Command

```bash
feroxbuster -u https://0aa2009603a39e2d80643fad001b00d6.web-security-academy.net/ -w /home/kali/Downloads/common.txt
```



The scan returned several responses, including a directory that appeared interesting:

```text
/cgi-bin/
```

Since CGI directories are commonly associated with administrative scripts, debugging tools, or legacy server functionality, I decided to investigate it further more.

### Screenshots

![Command Execution](/Screenshots/Screenshot-01-Lab-02.png)

---

![Directory Enumeration](/Screenshots/Screenshot-02-Lab-02.png)

---

## Discovery of Debug Functionality

While exploring the discovered directory, I found the following file(only one there):

```text
/cgi-bin/phpinfo.php
```

Opening the page revealed a large amount of server-side information including:

- PHP configuration details
- Loaded modules
- Server environment information
- System variables
- Environment variables

---

## Investigating Exposed Information

Using the browser search functionality, I searched for keywords commonly associated with sensitive information such as:

```text
secret
key
token
password
api
```

This approach quickly led to the discovery of a secret key stored within the **Environment Variables** section.

### Screenshot

![Secret Key Disclosure](/Screenshots/Screenshot-03-Lab-02.png)

---

## Exploitation

The disclosed secret key was copied directly from the debug page.

Because the application exposed the value without requiring authentication or special privileges, any user capable of accessing the page could obtain it.

The extracted secret key was then submitted to complete the lab successfully.

---

## Root Cause

The vulnerability existed because a development/debugging resource (`phpinfo.php`) was accessible from the public-facing application.

The page displayed sensitive environment information without any access control, resulting in the exposure of confidential application data.

---

## Security Recommendations

To prevent similar vulnerabilities:

     ▤ Remove debugging pages from production environments.

     ▤ Restrict access to administrative and diagnostic resources.

     ▤ Avoid storing sensitive values in locations that may be displayed to users.

     ▤ Audit publicly accessible files before deployment.

---

## Conclusion

This lab demonstrated how exposed debugging functionality can lead to information disclosure vulnerabilities. By doing directory enumeration with Feroxbuster and analysis of a publicly accessible `phpinfo()` page, sensitive environment variables were disclosed, including the application's secret key. 
