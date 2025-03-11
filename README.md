# ___Content___

1. [SSL-certificate](#основная-часть)

   1.1. [What is an SSL-certificate?](#подраздел-1)

   1.2. [How do SSL-certificates work?](https://github.com/NikitaPrimakov/Certificate?tab=readme-ov-file#how-do-ssl-certificates-work "How do SSL-certificates work?")

   1.3. [Certificate issue](https://github.com/NikitaPrimakov/Certificate?tab=readme-ov-file#certificate-issue "Certificate issue")

   1.4. [Additional settings for Gitlab](https://github.com/NikitaPrimakov/Certificate?tab=readme-ov-file#additional-settings-for-gitlab "Additional settings for Gitlab")

3. [Conclusion](#Conclusion)

# ___SSL-certificate___

## ___What is an SSL-certificate?___

```An SSL certificate``` is a digital certificate that authenticates a website and allows you to use an encrypted connection. The abbreviation SSL stands for Secure Sockets Layer, a security protocol that creates an encrypted connection between a web server and a web browser.

Companies and organizations need to add SSL certificates to websites to protect online transactions and ensure the confidentiality and security of customer data.

SSL ensures the security of Internet connections and prevents intruders from reading or altering information transmitted between the two systems. If a lock icon is displayed in the address bar next to the web address, then this website is secured using SSL.

Since the SSL protocol was created about 25 years ago, it has been available in several versions. When using each of these versions, there were security issues at some point. Then there was an updated renamed version of the protocol – TLS (Transport Layer Security), which is still in use. However, the abbreviation SSL has stuck, so the new version of the protocol is still often referred to by the old name.

## ___How do SSL-certificates work?___

Using SSL ensures that data transmitted between users and websites or between two systems cannot be read by third parties or systems. SSL uses algorithms to encrypt the transmitted data, which prevents attackers from reading it when it is transmitted over an encrypted connection. This data includes potentially sensitive information such as names, addresses, credit card numbers, and other financial data.

The process works as follows:

1. The browser or server is trying to connect to an SSL-protected website (web server).
2. The browser or server requests identification from the web server.
3. In response, the web server sends a copy of its SSL certificate to the browser or server.
4. The browser or server checks whether this SSL certificate is trusted. If so, it informs the web server about it.
5. The web server then returns a digitally signed confirmation and starts the session encrypted using SSL.
6. Encrypted data is shared between the browser or server and the web server.

```Let's run an example of issuing this certificate for our GitLab server.```


## Certificate issue

Before we issue a certificate for our server, we will need to make changes to the ```/etc/hosts``` file by specifying the DNS name of our server - ```git01.local```.

<u>Checklist of completed commands:</u>

___```1. Configure hosts - file```___  
```
nano /etc/hosts
```

Let's perform the change as follows:

___```2. Add hostname```___
```
127.0.0.1 git01.local git01
```

Save the file and restart the system.

After restarting the system, I will create a folder where I will issue the certificate using the ```OpenSSL utility```.

<u>Checklist of completed commands:</u>

___```1. Creating a directory for certificates:```___

```
mkdir -p ~/ssl/gitlab
cd ~/ssl/gitlab
```

___```2. Creating a root key and certificate:```___

```
# Generating a private key for the root CA

openssl genrsa -out rootCA.key 4096

# Creating a root certificate

openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.crt -subj "/C=RU/ST=Moscow/L=Moscow/O=MyOrganization/OU=IT/CN=My Root CA" 
```

___```3. Creating a configuration file for a host certificate:```___
 
```
cat > gitlab.conf << EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = req_ext

[dn]
C=RU
ST=Moscow
L=Moscow
O=MyOrganization
OU=IT
CN=git01.local

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = git01.local
EOF
```

___```4. Creating a key and CSR for the host:```___

```
# Generating a private key for a host
    
openssl genrsa -out gitlab.key 2048

# Creating a CSR (Certificate Signing Request)
    
openssl req -new -key gitlab.key -out gitlab.csr -config gitlab.conf
```

What is CSR? ___A certificate signing request (CSR)___ is one of the first steps towards getting your own SSL/TLS certificate. Generated on the same server you plan to install the certificate on, the CSR contains information (e.g. common name, organization, country) the Certificate Authority (CA) will use to create your certificate.

___```5. Creating a configuration for signing:```___

```
cat > gitlab.ext << EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = git01.local
DNS.2 = git01
EOF
```

___```6. Signing the host certificate with the root CA:```___

```
openssl x509 -req -in gitlab.csr -CA rootCA.crt -CAkey rootCA.key \
-CAcreateserial -out gitlab.crt -days 365 -sha256 \
-extfile gitlab.ext
```

___```7. Installing certificates for GitLab:```___

```
sudo mkdir -p /etc/gitlab/ssl
sudo cp gitlab.crt /etc/gitlab/ssl/git01.local.crt
sudo cp gitlab.key /etc/gitlab/ssl/git01.local.key
sudo chmod 600 /etc/gitlab/ssl/git01.local.key
```

___```8. Configuring GitLab to use SSL:```___

```
sudo nano /etc/gitlab/gitlab.rb

external_url 'https://git01.local'
nginx['ssl_certificate'] = "/etc/gitlab/ssl/git01.local.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/git01.local.key
```

___```Important notes:```___

1. Save RootCA.key and RootCA.crt in a safe place. 
2. Add RootCA.crt to the trusted root certificate authorities on the client computers.
3. Make sure that the DNS name git01.local resolves to the correct IP address (via DNS or /etc/hosts)
4. The validity period of the host certificate is 365 days, the root certificate is 3650 days.

___```To add the root certificate to the trusted ones on Linux:```___

```

sudo cp rootCA.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

```


## Additional settings for Gitlab

___```Configure port 443 for Nginx in GitLab. Here are the necessary changes:```___

