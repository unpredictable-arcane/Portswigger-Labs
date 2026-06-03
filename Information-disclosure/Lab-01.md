

#  Lab 01 : Information Disclosure in Error Messages

---

## Overview

This lab demonstrates how verbose error messages can expose sensitive information.

By giving unexpected input to a request parameter, the application returns a detailed stack trace containing information about the backend technology and framework version.

---

## Objective

Obtain and submit the version number of the framework used by the application.

---

## Reconnaissance

While browsing product pages, it was observed that the application uses a `productId` parameter to retrieve product information. Every product have different Ids.


```http
GET /product?productId=12
```

### Screenshot - 1

![Normal Product Page](/Screenshots/Screenshot-01-Lab-01.png)

---

## Exploitation

The `productId` parameter was modified by appending unexpected characters:

```http
GET /product?productId=1223443-394
```

The application attempted to parse the supplied value as an integer and generated an exception.


### Exposed Information

- Java exception details
- Backend technology
- Framework version

### Disclosed Framework Version

```text
Apache Struts 2 2.3.31
```

### Screenshot - 2

![Error Message Disclosure](/Screenshots/Screenshot-02-Lab-01.png)

---

## Impact

Verbose error messages may disclose:

     ▤ Framework versions
     ▤ Programming languages
     ▤ Internal file paths
     ▤ Application architecture
     ▤ Debugging information

---


## Key Takeaways

     ▤ Error messages can be valuable sources of      information for attackers.

     ▤ Information disclosure vulnerabilities often aid further exploitation.

     ▤ Production applications should never expose stack traces or framework versions.

     ▤ Proper error handling is a critical security control.

