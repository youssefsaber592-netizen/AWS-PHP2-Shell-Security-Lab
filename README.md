# 🛡️ AWS PHP2Shell – Command Injection & Remediation Lab

A hands-on cybersecurity project focused on identifying, testing, and mitigating an **OS Command Injection vulnerability** in a Dockerized PHP web application deployed on an AWS Ubuntu EC2 instance.

This project documents the practical workflow, including environment setup, Docker deployment, source code analysis, controlled vulnerability testing, security remediation, and post-remediation verification.

---

## 📌 Project Overview

| Category | Details |
|---|---|
| Project Name | AWS PHP2Shell Security Lab |
| Security Topic | OS Command Injection |
| Infrastructure | AWS EC2 |
| Operating System | Ubuntu Linux |
| Containerization | Docker & Docker Compose |
| Application | PHP Web Application |
| Testing Tool | cURL |
| Version Control | Git & GitHub |

## 🧰 Technologies & Tools

- **AWS EC2:** Hosting the lab environment.
- **Ubuntu Linux:** Server operating system.
- **Docker:** Running the application in containers.
- **Docker Compose:** Managing application services.
- **PHP:** Analyzing and modifying application code.
- **cURL:** Performing HTTP-based security tests.
- **Git & GitHub:** Version control and documentation.

---

## 📂 Repository Structure

```text
AWS-PHP2-Shell-Security-Lab/
│
├── README.md
│
└── php2shell gap command injection/
    ├── Screenshot 2026-11-02 042540.png
    ├── Screenshot 2026-11-02 042722.png
    ├── Screenshot 2026-11-02 043012.png
    ├── Screenshot 2026-11-02 043317.png
    ├── Screenshot 2026-11-02 044911.png
    ├── Screenshot 2026-11-02 045019.png
    ├── Screenshot 2026-11-02 053500.png
    ├── Screenshot 2026-11-02 054041.png
    └── Screenshot 2026-11-02 054621.png
```

---

## 🚀 Lab Implementation

### 1. Environment Setup

The lab environment was prepared on an AWS EC2 Ubuntu instance.

The vulnerable PHP application was obtained from the following repository:

[tnsk/vulnweb – GitHub](https://github.com/tnsk/vulnweb)

The application was deployed using Docker Compose:

```bash
cd ~/vulnweb
docker compose up -d
```

To verify the running containers:

```bash
docker ps
```

**Deployment Evidence:**

![Environment Setup](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20042540.png)

### 2. Docker Container Verification

After deployment, the application services were checked to ensure that the required containers were running.

The environment includes:

- `vulnweb-app` – PHP web application.
- `vulnweb-db` – Database service.
- `vulnweb-adminer` – Database management interface.

**Running Containers:**

![Docker Containers](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20042722.png)

**Docker Build and Initialization Logs:**

![Build Logs](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20043012.png)

### 3. Source Code Discovery

The application directory structure was explored to locate the PHP source files responsible for handling security vulnerabilities.

The target file was:

```text
app/src/vulnerabilities/command_injection/source/low.php
```

**Source Discovery Evidence:**

![Source Discovery](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20043317.png)

### 4. Vulnerability Analysis – Command Injection

During source code inspection, an insecure command execution pattern was identified.

**Original Vulnerable Code:**

```php
$cmd = 'ping -c 4 ' . $ip;
$out = (string) shell_exec($cmd . ' 2>&1');
```

**Security Issue:**

The application directly concatenates user-controlled input into a system command before passing it to `shell_exec()`.

Without proper validation and safe command handling, an attacker may manipulate the command and execute unintended shell instructions.

**Vulnerability Analysis Evidence:**

![Vulnerable PHP Code](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20044911.png)

### 5. Pre-Exploitation Verification

Before performing security tests, the Docker environment was checked to confirm that the application services were running.

**Evidence:**

![Pre-Exploitation Verification](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20045019.png)

### 6. Controlled Exploitation & Testing

Controlled testing was performed within the lab environment to evaluate whether the application improperly handled user input.

The following cURL request was used:

```bash
curl -G --data-urlencode "ip=127.0.0.1; id" \
"http://localhost:8088/vulnerabilities/command_injection/source/low.php"
```

This test checks whether additional shell commands can be injected through the IP input parameter.

**Testing Evidence:**

![Command Injection Testing](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20053500.png)

### 7. Security Remediation

After identifying the vulnerable input-handling logic, security improvements were applied.

The remediation introduced:

- `filter_var()` with `FILTER_VALIDATE_IP` to validate the supplied IP address.
- `escapeshellarg()` to escape the value before using it as a shell argument.
- Rejection of invalid input before command execution.

**Updated PHP Code:**

```php
<?php
$ip = $_REQUEST['ip'] ?? '';

if ($ip !== '') {
    if (filter_var($ip, FILTER_VALIDATE_IP)) {
        $cmd = 'ping -c 4 ' . escapeshellarg($ip);
        $out = (string) shell_exec($cmd . ' 2>&1');
    } else {
        $out = "Error: Invalid IP address format.";
    }
}
?>
```

**Security Improvements:**

1. Input validation prevents values that are not valid IP addresses from reaching the command execution logic.
2. Shell argument escaping helps prevent shell metacharacters from changing the intended argument.
3. Invalid input is rejected with an error message.

**Remediation Evidence:**

![Security Remediation](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20054041.png)

> **Security Note:** This is a lab-level mitigation, not a complete production security design. The preferred approach is to avoid shell execution whenever possible and use safer process execution APIs with argument arrays, together with strict validation and least-privilege permissions.

### 8. Post-Remediation Verification

After applying the security changes, the same controlled test was repeated to verify how the application handled the malicious input.

**Verification Request:**

```bash
curl -G --data-urlencode "ip=127.0.0.1; id" \
"http://localhost:8088/vulnerabilities/command_injection/source/low.php"
```

The expected secure behavior is to reject the invalid IP address rather than execute the injected command.

**Post-Remediation Evidence:**

![Post-Remediation Verification](./php2shell%20gap%20command%20injection/Screenshot%202026-11-02%20054621.png)

---

## 🎯 Key Learning Outcomes

Through this project, I gained practical experience in:

- Deploying Dockerized applications on AWS EC2.
- Managing containers using Docker Compose.
- Understanding OS Command Injection vulnerabilities.
- Inspecting PHP source code for insecure command execution.
- Performing controlled security tests using cURL.
- Applying input validation and shell argument escaping.
- Verifying application behavior after security changes.
- Documenting technical findings and remediation steps.

---

## 📚 References

- [Original Vulnerable Lab – tnsk/vulnweb](https://github.com/tnsk/vulnweb)
- [OWASP – OS Command Injection Defense Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
- [PHP – filter_var() Documentation](https://www.php.net/manual/en/function.filter-var.php)
- [PHP – escapeshellarg() Documentation](https://www.php.net/manual/en/function.escapeshellarg.php)
- [Docker Documentation](https://docs.docker.com/)

---

## ⚠️ Disclaimer

This project is intended strictly for educational purposes and authorized security testing.

The vulnerable application must be kept in an isolated environment and should never be exposed to the public internet or used with real user data.

---

## 👨‍💻 Author

**Youssef Saber**

Computer Science Undergraduate | Software Engineer | Cloud & DevOps Enthusiast

- [GitHub](https://github.com/youssefsaber592-netizen)
- [LinkedIn](https://linkedin.com/in/youssef-saber-/)
