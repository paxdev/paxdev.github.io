---
  title: Create and install a local Root Certificate Authority on Windows
  tags:
    - certificates
    - openssl
    - powershell
    - chocolatey
  permalink: /security/create-a-local-root-ca/
---

## Introduction

In order to avoid warning messages about self-signed certificates when developing web applications on a local machine you can create your own devlopment `Root Certificate Authority`.

By trusting this `Root CA` you then implicitly trust any certificates that it has signed, so you don't need to keep adding certificates to your local store as long as they were signed by the `Root CA`.

**NB**
This post assumes you are using `Powershell`. 

## Installation

You can check to see if you have `openssl` installed by running the following at a command line

```powershell
openssl version
```

If you get a version back it's already installed.
If not you can use [chocolatey](https://chocolatey.org/).

```powershell
choco install openssl
```

You'll now need to close and re-open your `PowerShell` window to ensure that you get the updated path. *(I find `refreshenv` does not work reliably for me.)*

## Generating the Root CA

You may find it helpful to create a folder to work in. I'm working on my secondary HDD on `D:\`. Adjust your path accordingly

```shell
md \certificates
cd certificates
```

First of all we need to create a private key for our `CA`. 
Since we are going to implicitly trust any certificates signed by this `CA` it makes sense to password protect it for safety's sake

```shell
openssl genrsa -aes256 -out root-ca.key 4096
```

You will be prompted to enter a pass phrase. Make sure you choose a strong one. 
As mentioned above if a bad actor can get a hold of your `CA` and use it to sign something, you will implicitly trust it.

You should now have a private key in the file `root-ca.key`.

We'll now create the `Root CA`. We'll tell `openssl` to create a new key valid for 3650 days using our `root-ca.key` and output that to `root-ca.crt`
> For our purposes we can use the default configuration, but if you really want to lock down the creation of the Certificate Authority, you can use a configuration file. [This page has a great guide](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html) 
> Note that I have also skipped creating an `Intermediate CA`. See [here](https://aboutssl.org/root-certificates-vs-intermediate-certificates/) for details as to the difference.

```shell
openssl req -new -x509 -days 3650 -key root-ca.key -out root-ca.crt
```

You'll be prompted to enter the pass phrase you created above, then some details for the `CA` key. 

```shell
Country Name (2 letter code) [AU]:GB
State or Province Name (full name) [Some-State]:England
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:PaxDev
Organizational Unit Name (eg, section) []:PaxDev
Common Name (e.g. server FQDN or YOUR name) []:PaxDev
```

We can now import the `CA` to the `LocalMachine\Root` *(`Local Computer\Trusted Root Certification Authorities` in the Certificates MMC snap-in)*. 

From an **elevated** `PowerShell` prompt:

```powershell
Import-Certificate root-ca.crt -CertStoreLocation Cert:\LocalMachine\Root\ -Verbose
```

## Create a new SSL Certificate to use on a website

First of all we need to create a signing request. We'll pass in the configuration in a file.

Create a new file. The name is unimportant, but I'll go with `ssl.conf`

```ini
[req]
default_bits = 4096
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = v3_req

[dn]
C = GB
ST = England
O = PaxDev
CN = localhost-dev

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @subjectAltNames

[subjectAltNames]
DNS.1 = localhost
DNS.2 = *.test-domain.com
IP.1 = 10.0.0.22
```

The first section sets up the key size and mode, then sets the distinguished name to be in the section `[dn]` and instructs `openssl` that we are using the `V3 X.509` extensions and names the section they are in.

This allows us, in particular to 
* State that this is not  `CA` certificate - i.e. it cannot be used to sign other certificates
* Use `Subject Alt Names` to define multiple addresses and wildcards that this certificate will authenticate.

The `subjectAltNames` section is a list of the URLs and/or IP Addresses that this certificate will be used for

We now generate a private key for the SSL certificate

```shell
openssl genrsa -aes256 -out localhost-dev.key 4096
```

You'll be prompted for a passphrase again. 

Now create a `Certificate Signing Request` or `CSR`

```shell
openssl req -new -key localhost-dev.key -config ssl.conf -out localhost-dev.csr
```

You'll be prompted for the pass phrase you created in the previous step (not the `CA` pass phrase). 

In this case we are signing with the '`SSL` Key' to validate that it came from us.

Now we need our `Root CA` to create the certificate. 

Firstly we need to duplicate the `V3 X.509` extensions data from the above config file. I created a file called `localhost-dev.v3.ext`:

```ini
[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @subjectAltNames

[subjectAltNames]
DNS.1 = localhost
DNS.2 = *.test-domain.com
IP.1 = 10.0.0.22
```

```powershell
openssl x509 -req -in localhost-dev.csr `
                      -CA root-ca.crt `
                      -extfile localhost-dev.v3.ext `
                      -CAkey root-ca.key `
                      -CAcreateserial `
                      -out localhost-dev.crt `
                      -days 365 `
                      -extensions v3_req
```

You will be prompted for the `Root CA` pass phrase. 
(The `Root CA` signs the certificate and because we trusted it earlier we automatically trust any certificates signed by it.)

We need to combine the new `Certificate` with the `SSL` Key we created just now.

```shell
openssl pkcs12 -export -out localhost-dev.pfx -inkey localhost-dev.key -in localhost-dev.crt
```

You will be asked to confirm the `SSL` Key pass phrase. Leave the export password blank.

Finally we need to import the certificate to the `LocalMachine\My` *(`Local Computer\Personal` in the Certificates MMC snap-in)* store

```powershell
Import-PfxCertificate localhost-dev.pfx -CertStoreLocation Cert:\LocalMachine\My\ -Verbose
```

You can now select your certificate in `IIS Manager` for the `HTTPS` binding. 

**NB** You will still need to manually add exceptions for the site in Firefox but you do only need to do that once.