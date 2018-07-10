# SELF SIGNED CERTIFICATE

Commands and configuration file to create self-signed certificates. Configurations files are in `etc` directory.

This material is based on many websites explaining the method. It has simplified as much as possible the whole process and configuration options.

_**NOTE**_ : 

- you will still get warnings saying that the certificate is untrusted. As your CA cert is not in the trusted root CA list by your browser, the device doesn't trust your servers certificate.
- The config file have a section to define the [Subject Alternative Name (SAN)](https://support.dnsimple.com/articles/what-is-ssl-san/) extension.
- Most documentation use two configuration files. We decided to group everything in one only.


```C
$ git clone https://github.com/gabx/self-signed-certificate.git
$ cd self-signed-certificate
$ echo '01' > serial && touch index.txt
$ cp /dev/null index.txt.attr
```

Then you must edit the **self-sign.conf** configuration file according to your need. 

##  Become a Certificate Authority

The first step is to become a stand-alone Certificate Authority (CA) which will sign  as many certificates as you like.

###  generate password protected key

```C
$ openssl genrsa -des3 -out myCA.key 2048
```
Ignore the `-des3` option to remove password protection.

### generate root certificate

```C
$ openssl req -x509 -config etc/self-sign.conf -new -key myCA.key -out myCAcert.pem 
```

You should now have two files: **myCA.key** (your private key) and **myCAcert.pem** (your root certificate). Do not change their names, or if so, change it too in the **sel-sign.conf** file.

## Create CA-Signed Certificates 

### Create a private key

```C
$ openssl genrsa -des3 -out MyFQDN.key 2048
```

###Generate a certificate sign request

The CSR is sent to a Certificate Authority, that verifies the identity of the requestor and issues a signed certificate. In our case, WE are the Certificate Authority. Questions are already answered with what you indicated in **self-sign.conf**. You only have to confirm by pressing the `Enter` key.

```C
openssl req -config etc/self-sign.conf -new -key MyFQDN.key -out MyFQDN.csr
```

### Create the certificate using our CSR, the CA private key, the CA certificate

Now it is time to put everything together and do the magic:

```C
openssl x509 -req -in MyFQDN.csr -CA myCAcert.pem -CAkey myCA.key -CAcreateserial -out MyFQDN.crt.pem -days 3650 -sha256 -extfile etc/self-sign.conf
```

## Few additional commands
### view cert

```C
openssl x509 -in MyFQDN.crt.pem -text -noout
```

### encode 64 

On certain occasion, you will be asked to give a base 64 encoded certificate. Here is the command to run:

```C
cat MyFQDN.crt.pem | base64 -w 0
```

### Verify

```C
$ openssl verify -CAfile myCA.pem  MyFQDN.crt.pem
MyFQDN.crt.pem:OK
```


## Resources

- [Create your own cert for development sites](https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/)
- [implement real-world PKIs with the OpenSSL toolkit](https://pki-tutorial.readthedocs.io/en/latest/index.html)
- [Cert creator script](https://github.com/TotallyInformation/SelfSigned-Cert-Creator)
- [Stack overflow 1](https://stackoverflow.com/questions/21297139/how-do-you-sign-a-certificate-signing-request-with-your-certification-authority/21340898#21340898)
- [Stack overflow 2](https://stackoverflow.com/questions/10175812/how-to-create-a-self-signed-certificate-with-openssl/27931596#27931596)
