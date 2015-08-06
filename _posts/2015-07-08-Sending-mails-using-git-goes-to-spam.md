---
layout: post
title: Sending mails through git ends up in spam folder
categories: []
tags: []
published: True
---

Have you ever wondered why sending mails using git ends up in spam folder in your gmail mailbox. I was recently trying to send some patches of git to someone and i noticed that all the mails i sending using git-format0-mail ends up in my spam folder.

This was annoying and i wanted to make sure that it never happens again and so i digged ina bit.I was using my gmail account to send the mails using my account login and password.But still the mails ended up as spam.

This is because gmail is not able to make the channel secure using TLS/SSL autentication.The simple solution is to use a package called msmtp-mta.

You can install it by using 

```
sudo apt-get install msmtp-mta
```

Now that you have installed tha package you need to configure it using the .msmtprc in your home directory
```
vi  ~/.msmtprc
```

Now you need to configure your account details by modifying the necessary fields.

```
   # Example for a user configuration file
   # Set default values for all following accounts.
   defaults
   tls on
   tls_trust_file  /etc/ssl/certs/ca-certificates.crt
   logfile ~/.msmtp.log
   # My email service
   account gmail
   host smtp.gmail.com
   port 587
   from your@accountdetails.com
   auth on
   user your@accountdetails.com
   password your@password
   # Set a default account
   account default : gmail
```

Now change the file permission so that no one can see it 

```
chmod 600 .msmtprc

```

If you are using gmail, i strongly suggest you to use the 2 step autenticaion mechanisim as mentioned  [here](https://support.google.com/accounts/answer/180744?hl=en).

This will make sure that you dont have to expose your gmail actual password and no other person can use it even if he sees it.

Now that you have all the mail configuration done you can use the git-format-patch as earlier without any changes as below 

```

 git format-patch  -n2  --to=yourId@gmail.com 

```