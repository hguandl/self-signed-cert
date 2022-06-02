# Self-signed SSL Certificates Tutorial

## Generate CA

```shell
$ openssl req -x509
    -config ca.conf \
    -newkey rsa:4096 \
    -days 824 \
    -sha256 -nodes -outform PEM \
    -keyout ca_data/cakey.pem \
    -out ca_data/cacert.pem
```

If you prefer to use ECC, change the 3rd line to
```shell
    -newkey ec -pkeyopt ec_paramgen_curve:secp384r1 \
```

All the available curves are listed by execuating `openssl ecparam -list_curves`.

> Install `cacert.pem` to your device. Keep `cakey.pem` secret.

## Issue a certificate

### Fill out alternate names

```shell
$ cp server.template.conf server.conf
```

Append IP or domains to the end of `server.conf`, eg:

```ini
DNS.1 = *.example.com
DNS.2 = local.yourdomain.com

IP.1 = 192.168.0.1
IP.2 = ::1
```

### Generate CSR

```shell
$ openssl req \
    -config server.conf \
    -newkey rsa:2048 \
    -days 397 \
    -sha256 -nodes -outform PEM \
    -keyout server-key.pem \
    -out server.csr
```

The configuration of using ECC is the same as above.

### Sign the certificate with CA

If you are signing for the first time, some files should be created.
```shell
$ touch ca_data/index.txt
$ echo '01' > ca_data/serial.txt
```

```shell
$ openssl ca \
    -config ca.conf \
    -policy signing_policy \
    -extensions signing_req \
    -infiles server.csr \
    -out server-cert.pem
```

> Use `server-cert.pem` and `server-key.pem` for your server. Keep `server-key.pem` secret.

### (Optional) Create a PKCS #12 bundle

```shell
$ openssl pkcs12 -export -in server-cert.pem -inkey server-key.pem -out server-bundle.p12
```

## Deploy the Certificates

- Install the CA certificate to devices.
- Use the signed certicate for server.

## References

- <https://stackoverflow.com/a/21340898>
- <https://stackoverflow.com/a/27931596>

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.
