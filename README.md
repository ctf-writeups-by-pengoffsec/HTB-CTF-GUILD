

---

# 🐧 Penguin's Offensive Security - Hack The Box: Guild CTF Writeup

**Welcome to the world of Penguin's Offensive Security!**

This is the writeup for the "Hack The Box" CTF challenge named **"Guild"**.

---

## 📖 Overview

We will proceed step-by-step to:

1. Explore the web page
2. Analyze the source code
3. Assess vulnerabilities
4. Exploit them to retrieve the flag 🏁

---

## 🧠 Skills Required

* Code Analysis
* Vulnerability Assessment & Penetration Testing (VaPT)
* Server-Side Template Injection (SSTI)
* Steganography Techniques

---

## 🔍 Initial Web Page Analysis

### Step 1: Sign Up & Login

We begin by signing up and logging into the application.

![Step 1](/imgs/1.png)

---

### Step 2: Uploading a Guild Badge

After logging in, we are asked to upload an image for the guild badge.

![Step 2](/imgs/2.png)

Once uploaded, the image is sent to the **Guild Master** for approval.

![Step 3](/imgs/3.png)

---

## 🛡️ Privilege Escalation

Naturally, there is no actual "Guild Master" in a CTF environment. So, we must **escalate privileges** to become the master ourselves.

Attempting to directly access `/admin` yields a forbidden message:

![Step 4](/imgs/4.png)

---

## 🔍 Code Analysis

### 1. Image Verification

We see that the code checks for valid image extensions like `.png`, `.jpg`, `.jpeg`.

![Step 5](/imgs/5.png)

---

### 2. Forget Password Logic

The "forgot password" function checks if the email exists and claims to send a reset link, but it **doesn't actually send any email**.

![Step 6](/imgs/6.png)

Instead, we discover an endpoint:

```
/changepasswd/<HASH>
```

Here, if we **SHA256 hash an existing email**, we can craft a password reset URL and change that user's password! This is a **critical vulnerability** in real-world applications.

---

### 3. Finding the Admin Email

Trying SQL Injection won’t help — regex filtering is in place to detect such payloads.

![Step 8](/imgs/8.png)

In `init.py`, we find valuable clues:

* Admin's username: `admin`
* Email domain: `@master.guild`
* The actual email is **randomly generated**, so brute-forcing won't work.

![Step 9](/imgs/9.png)

---

## 💉 Exploiting SSTI to Get Admin Email

Visiting the **Profile > Bio** section reveals an SSTI vulnerability. Inject the following Jinja2 payload:

```jinja
{{ User.query.filter(User.username=="admin").first().email }}
```

![Step 10](/imgs/10.png)

Click the "Get Link" button to retrieve the profile page URL. Navigate to that page to reveal the admin email:

![Step 11](/imgs/11.png)

---

## 🔑 Resetting the Admin Password

Use the revealed email in the forgot password form:

![Step 12](/imgs/12.png)

Then generate its SHA256 hash:

```bash
echo -n '<admin-email>' | sha256sum
```

Navigate to:

```
/changepasswd/<GENERATED-HASH>
```

Now reset the Guild Master's password.

![Step 13](/imgs/13.png)

---

## 🛠️ Admin Panel Access

Login using the new admin credentials. You’ll see the admin dashboard:

![Step 14](/imgs/14.png)

Here you can **verify images** submitted by normal users.

---

## 🖼️ Steganography & SSTI Payload Injection

To be marked as "Verified", the image must contain the `Artist` EXIF metadata tag.

![Step 15](/imgs/15.png)

You can add this using:

```bash
exiftool -Artist="any text" image.jpg
```

However, to retrieve the **flag**, we inject an SSTI payload into the `Artist` tag:

```bash
exiftool -Artist="{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read() }}" image.jpg
```

Upload this **malicious image** as a regular user.

![Step 16](/imgs/16.png)

---

## ✅ Final Step

Logout, then log back in as the admin. Navigate to the admin panel and verify the uploaded image.

🎉 **Boom! The flag is revealed!**

![Step 17](/imgs/17.png)

---

## 🏁 Conclusion

If you've followed all the steps, you should have successfully retrieved the flag!

* ✅ If you did it – **Congratulations!**
* ❓ If you're stuck – feel free to reach out via the social links on my [GitHub profile](https://github.com/ctf-writeups-by-pengoffsec).

---

Let me know if you’d like a version with clickable image links or hosted screenshots for better GitHub rendering.
