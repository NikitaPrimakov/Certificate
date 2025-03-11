# ___Content___

1. [SSL-certificate](#основная-часть)

   1.1. [What is an SSL-certificate?](#подраздел-1)

   1.2. [How do SSL-certificates work?](https://github.com/NikitaPrimakov/Certificate?tab=readme-ov-file#how-do-ssl-certificates-work "How do SSL-certificates work?")

   1.3. [Certificate issue](https://github.com/NikitaPrimakov/Certificate?tab=readme-ov-file#certificate-issue "Certificate issue")

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

1.  ```
    nano /etc/hosts
    ```

Let's perform the change as follows:

2.  ```
    127.0.0.1 git01.local git01
    ```
Save the file and restart the system.

After restarting the system, I will create a folder where I will issue the certificate using the ```OpenSSL utility```.

<u>Checklist of completed commands:</u>

1. Creating a directory for certificates:
     ```
    mkdir -p ~/ssl/gitlab
    cd ~/ssl/gitlab
    ```
2. Creating a root key and certificate:
    ```
    # Generating a private key for the root CA

    openssl genrsa -out rootCA.key 4096

    # Creating a root certificate

    openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -out rootCA.crt -subj "/C=RU/ST=Moscow/L=Moscow/O=MyOrganization/OU=IT/CN=My Root CA" 
    ```

3. Creating a configuration file for a host certificate:
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