# Insecure Direct Object References



# Overview

This lab demonstrates an **Insecure Direct Object Reference (IDOR)** vulnerability where sensitive files are referenced directly through predictable object identifiers.

The application provides a live chat feature that allows users to save and download conversation transcripts. The transcript files are stored using sequential numeric filenames.

---

# Objective

The objective of this lab was to:

1. Interact with the live chat feature.
2. Download a chat transcript.
3. Analyze how transcript files are referenced.
4. Manipulate the transcript identifier.
5. Access another user's transcript.
6. Recover Carlos's password.
7. Log in as Carlos.
8. Solve the lab.

---

# Understanding the Vulnerability

The application allows users to download chat transcripts generated during conversations Live Chat Section with bot.

Transcript files are stored using numeric identifiers as:

```http
/download-transcript/2.txt
/download-transcript/3.txt
/download-transcript/4.txt.. so on
```
---

# Transcript Generation

So, after the conversation was completed, the **View Transcript** option was used to download the generated chat log.

### Observation

While reviewing the download request in Burp Suite, the transcript file followed the pattern:

```http
/download-transcript/2.txt
```

Additional transcripts generated later used:

```http
/download-transcript/3.txt
/download-transcript/4.txt
```


### Screenshot

![Transcript Download Request](/Screenshots/Screenshot-02-Lab-03.png)

---

# Identifying the Missing Transcript

During testing, transcript files with identifiers:

```text
2.txt
3.txt
4.txt
```

were observed.

However, transcript:

```text
1.txt
```

had not been accessed.

This raised the possibility that the first transcript belonged to another user and might contain sensitive information.

---

# Parameter Manipulation

The intercepted request was modified within Burp Suite.

Original Request:

```http
GET /download-transcript/2.txt HTTP/2
```

Modified Request:

```http
GET /download-transcript/1.txt HTTP/2
```

The manipulated request was then forwarded to the server.

### Screenshot

![Modified Transcript Request](/Screenshots/Screenshot-03-Lab-03.png)

---

# Unauthorized Access to Another User's Transcript

The server returned the contents of transcript `1.txt`.

The transcript contained a previous conversation between a user and the support chatbot.

### Exposed Information

The transcript revealed Carlos's password:

```text
9t6rgk7k1gr0iomfhn4y
```

The password had been provided by the chatbot during the conversation.

### Screenshot

![Recovered Password](/Screenshots/Screenshot-04-Lab-03.png)

---

# Authentication as Carlos

Using the credentials obtained from the transcript, authentication was attempted using Carlos's account.

### Credentials Used

```text
Username: carlos
Password: 9t6rgk7k1gr0iomfhn4y
```

The login was successful.

After authentication, the application granted access to Carlos's account, automatically solving the lab.

### Screenshot

![Lab Solved](/Screenshots/Screenshot-05-Lab-03.png)

---

# Technical Analysis

## Original Request

```http
GET /download-transcript/2.txt HTTP/2
Host: TARGET
```

---

## Modified Request

```http
GET /download-transcript/1.txt HTTP/2
Host: TARGET
```

---

## Root Cause

The vulnerability exists because:

1. Transcript files are referenced using predictable identifiers.
2. File ownership is not verified.
3. Sensitive files are directly accessible through user-controlled URLs.

---

# Attack Flow

```text
Access Live Chat
        ⥥
Generate Transcript
        ⥥
Download Transcript
        ⥥
Observe Sequential File IDs
        ⥥
Identify Missing Transcript
        ⥥
Modify Request
        ⥥
Access Transcript 1.txt
        ⥥
Recover Carlos Password
        ⥥
Login as Carlos
        ⥥
Lab Solved
```

---

# Remediation

## ⇒ Enforce Authorization Checks

Before serving any transcript file, verify that the authenticated user owns the requested resource.

---

## ⇒ Avoid Predictable Object References

Sequential identifiers such as:

```text
1.txt
2.txt
3.txt
```

should not be exposed to users.

Use randomly generated identifiers instead.

Like:

```text
f72c3a9f-4f2e-8e6c7b8f93d4.txt
```

---

## ⇒ Restrict Access to Sensitive Files

Transcript files should be stored outside publicly accessible directories and served only after authorization checks.

---


## ⇒ Implement Access Logging

Monitor requests for:

```text
/download-transcript/*
```

and alert on abnormal access patterns or identifier enumeration attempts.

---

# Conclusion

The application stored chat transcripts using predictable numeric identifiers and failed to verify ownership before serving transcript files. By modifying the transcript filename from `2.txt` to `1.txt`, it was possible to access another user's chat history. The exposed transcript contained Carlos's password, which was then used to authenticate as Carlos and successfully solve the lab.

This lab demonstrates how insecure direct object references can expose sensitive information and ultimately lead to unauthorized account access.