# Public Key Infrastructure Assignment

I decided to use Apache as is a technology I have heard before and the instruction nudge me to gain some exposure to it for my fututre career.

## 🌐 1: Host a Local Server

I started Apache using:

<sudo apachectl start>

And then opened <http://localhost> in Chrome to see if it was working.

It indeed was and showed a *"It works!"* message.
After that I navigated to the Document Root where Apache server files are located
``` cd /Library/WebServer/Documents/
ls>

In here I created my [web page](screenshots/localhost_working.png) that given that it's only HTTP, the browser flags as [_Not Secure_](screenshots/localhost_nosecure.png)


## 🕵️‍♀️ 2: Identify why HTTP is not secure

To understand why HTTP is not secure we have to understand how Apache works:

1. Browser sends a *request to* the web server
2. Apache receives the request (it's listening on Port 80)
3. Apache looks in the Document Root the content of the web page (index.html), reads it and sends it back to the browser
4. Browser shows the webpage

Now because HTTP has no encryption, any agent connected to the same network can read the Apache signal back (step 3), this type of "spying" is called *eavesdropping* or *packet sniffing*. 

After initializing Wireshark we can select from a list of available network interfaces, the one that we are interested now is <lo0> that caputres localhost traffic. Once there we can see the caputred packets, as shown in the [screenshot](screenshots/http_response_visible.png) the eavesdropper can see all the content of the page I'm inteacting with, via the source (html) code.

Other interesting things that the packages tell a possible eavesdropper is mi OS and even the size of the [window](screenshots/window_size.png) I'm loading, this does not relates to security but it is an important observation for privacy

## 🔐 3. Create a self-signed certificate and upgrade web server to HTTPS
CA only issue certificates to public domain pages, because this is a private page, there is no way (and no point) in issue a certificate through a CA.

### 3.1 Generate a Self-Signed SSL Certificate
The next step is first to create my private (secret) key and a Public certificate (to be shared with the browser). These will be stored in a the </certs> folder that will be contained in <.gitignore> to avoid leaking any sensitive information, if the private key were to leak, any agent could pretend to be my localhost.

To generate the the command:
´´´
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout server.key \
  -out server.crt \
  -subj "/C=US/ST=CA/L=Local/O=Student/OU=Web/CN=localhost"

´´´
 - <openssl> Use OpenSSL (crypto) tool
 - <req> Create a certificate request 
 - <x509> Make self-signed certificate
 - <nodes> Do not password-protect the key bc Apache need to use it automatically
 - <days 365> Certificate valid for 1 year
 - <newket rsa:2048> Generate key-pair, 2048 bits
 - <keyout server.key> Save private key 
 - <out server.crt> Save certificate
 - <subj "..."> Certificate info

### 3.2 Activate certificates
After that I need to move the certificate for Apache's to read

´´´
sudo mkdir -p /etc/apache2/ssl
sudo cp server.key server.crt /etc/apache2/ssl/
sudo chmod 600 /etc/apache2/ssl/server.key
sudo chmod 644 /etc/apache2/ssl/server.crt
´´´

And enable SSL in Apache to encrypt the communications:
´´´
sudo nano /etc/apache2/httpd.conf
´´´

I enabled here the modules required for HTTPS:
- LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
- LoadModule ssl_module libexec/apache2/mod_ssl.so
- Include /private/etc/apache2/extra/httpd-ssl.conf

And then 
´´´
sudo nano /etc/apache2/extra/httpd-ssl.conf
´´´
to verify SSLCertificate and SSLCertificateKeyFile are in the corect locations

But after this eventhough my localhost was https, I got a [_Your connection is not private_](screenshots/connection_not_private.png) message. This behavior is expected given that the browser detects the self-signed certificate and not from a trusted authority (CA). After going to the advance settings and then Proceed to localhost [the https still appears as no secure](screenshots/nosecure_https.png) because the browser does not fully trusts the certificate because the certificate is not in my root directory

To add it in my OS system (MacOS) I had to open Keychain Access and there add the certificate, and change it to [always trust](screenshots/always_trust_cert.png). Now the browser trusts the localhost.

Now with HTTPS and the certificate the only thing that I can see is that there was a handshake between my machine and the web server, and if I try to check the information on what I'm interacting with, I only see that the connection is [encrypted](screenshots/encrypted.png), a potential eavesdropper can see that there was a request and a response but not the content of them.

