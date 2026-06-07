# Information Disclosure in Version Control History


# Overview

This lab demonstrates how sensitive information can remain accessible through a publicly exposed Git repository.

During the assessment, directory enumeration revealed an exposed `.git` directory. After downloading the repository locally and reviewing its commit history, an administrator password was recovered.

---

# Objective

The objective of this lab was to:

1. Identify publicly accessible Git repository files.
2. Download the exposed repository.
3. Analyze Git commit history.
4. Recover sensitive information stored in previous commits.
5. Obtain administrator credentials.
6. Access the administrator panel.
7. Delete the user `carlos`.

---

# Reconnaissance

## Directory Enumeration

The assessment began with content discovery using Feroxbuster to identify hidden files and directories exposed by the web application.

### Command Used

```bash
feroxbuster -u https://TARGET/ -w /home/kali/Downloads/common.txt
```

### Findings

This scan revealed multiple Git-related resources including:

```text
/.git/
/.git/index
/.git/config
/.git/HEAD
/.git/logs/
```

The presence of these files indicated that the application's version control repository had been unintentionally exposed to public users.

### Screenshots

![Feroxbuster Discovery](/Screenshots/Screenshot-01-Lab-05.png)

![Feroxbuster Discovery-2](/Screenshots/Screenshot-02-Lab-05.png)

---

# Verification of Exposed Repository

After identifying Git-related resources through enumeration, the `.git` directory was accessed directly through the browser.

### URL Accessed

```text
https://TARGET/.git
```

The server responded with a directory listing containing repository files and folders.

Included:

```text
HEAD
config
index
objects/
refs/
logs/
```



### Screenshot

![Exposed Git Directory](/Screenshots/Screenshot-03-Lab-05.png)

---

# Repository Acquisition

Since the Git repository was accessible, the next step was to download it locally for further analysis.

### Command Used

```bash
wget -r https://TARGET/.git
```

( The command recursively downloaded the entire repository structure, including objects, logs, references, and metadata required to reconstruct repository history. )

### Screenshot

![Downloading Repository](/Screenshots/Screenshot-04-Lab-05.png)

---

# Git History Analysis

After downloading the repository, Git commands were used to inspect previous commits and identify potentially sensitive information.

A search was performed for references related to administrative functionality.

### Command Used

```bash
git log -p | grep -i "admin"
```

### Findings

Reviewing the commit history revealed a previous configuration entry containing a hardcoded administrator password.

The output showed:

```text
ADMIN_PASSWORD=1ad1xk6tk73tieyfhgws
```


```diff
-ADMIN_PASSWORD=1ad1xk6tk73tieyfhgws
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')
```

Although the password had been removed from the current source code, it remained accessible through Git commit history.

### Screenshot

![Recovered Password](/Screenshots/Screenshot-05-Lab-05.png)

---

# Authentication as Administrator

Using the credentials recovered from Git history, an attempt was made to authenticate as the administrator.

### Credentials Recovered

```text
Username: administrator
Password: 1ad1xk6tk73tieyfhgws
```

The login was successful, resulting in access to the administrative interface.



## Administrative Access

After successful authentication, the administrative panel became accessible.

The panel displayed user management functionality, including the ability to delete user accounts.

The target user identified by the lab was:

```text
carlos
```

### Screenshot

![Admin Panel Access](/Screenshots/Screenshot-06-Lab-05.png)

---

## User Deletion

Using the administrative Authority, the account belonging to `carlos` was deleted.

After deletion, the lab status changed to solved.....


---

# Technical Analysis
---

# Security Impact

An attacker may leverage this information to:

- Bypass authentication
- Escalate privileges
- Access administrative functions
- Perform lateral movement
- Gain complete compromise of the application

In this lab, exposure of Git history resulted in disclosure of valid administrator credentials and complete administrative access.

---

# Attack Flow

```text
Directory Enumeration
        ⥥
Discovery of /.git/
        ⥥
Access Repository Listing
        ⥥
Download Repository
        ⥥
Analyze Git History
        ⥥
Recover Admin Password
        ⥥
Login as Administrator
        ⥥
Delete Carlos User
        ⥥
Lab Solved
```

---

# Remediation


### Restrict Access to Git Directories

Configure the web server to deny access to:

```text
/.git
/.svn
/.hg
```


---

### Remove Sensitive Data from Repository History

If secrets are committed:

- Rotate exposed credentials immediately.
- Rewrite repository history using tools such as:

```bash
git filter-repo
```

---

## Conduct Security Reviews

Perform periodic assessments to identify:

- Exposed repositories
- Directory listings
- Sensitive files
- Misconfigured web servers

---


# Conclusion

The application exposed its Git repository to unauthenticated users, allowing complete repository download and historical commit analysis. 

This lab highlights the security risks associated with exposed version control repositories and demonstrates how historical commit data can lead directly to privilege escalation and administrative compromise.