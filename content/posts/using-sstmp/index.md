---
author: ["Tony Nguyen"]
title: "Lightweight Email Sending with sSMTP: A Quick Guide"
date: "2025-04-26"
description: "Lightweight Email Sending with sSMTP: A Quick Guide"
summary: "Using sstmp to send email in linux"
categories: ["linux"]
tags: ["linux", "ssmtp", "smtp", "CLI", "command"]
ShowToc: true
TocOpen: true

cover:
  image: "images/ssmtp.jpg"
  alt: "Simple SMTP"
  caption: "Simple SMTP"
  relative: true
---

To send an email using Gmail from the Linux shell, you can use `mutt` or `ssmtp` configured to use Gmail's SMTP server. Here's how to do it:
### What is sSMTP?
sSMTP (Simple SMTP) is a lightweight program used to send emails from Unix/Linux systems via an external SMTP server (like Gmail, Outlook, etc.). It's a simple alternative to the full sendmail system, ideal for systems that don’t need to receive emails—just send them.

### Key Features
- Sends emails via a remote SMTP server
- Lightweight and easy to configure
- No mail queue or local delivery—just forwarding
- Useful for cron jobs, system alerts, or small servers

### Usage
#### Step 1: Install `ssmtp`

If you don't have `ssmtp` installed, install it:

- **Debian/Ubuntu:**
```bash
sudo apt-get install ssmtp
```

#### Step 2: Configure `ssmtp`

Edit the configuration file:

```bash
sudo nano /etc/ssmtp/ssmtp.conf
```

Add the following configuration:

```plaintext
root=your_email@gmail.com
mailhub=smtp.gmail.com:587
AuthUser=your_email@gmail.com
AuthPass=your_password
UseSTARTTLS=YES
```

#### Step 3: Send Email

You can send an email using:

```bash
echo -e "Subject: Subject of the Email\n\nThis is the body of the email." | ssmtp recipient@example.com
```

### Notes
- **App Passwords**: If you have 2-Step Verification enabled on your Google account, you’ll need to generate an App Password and use that in place of your regular password.
- **Less Secure Apps**: If you don't use 2-Step Verification, you may need to allow less secure apps in your Google account settings.

### Important
Make sure to secure your credentials and do not expose them public or shared scripts.


