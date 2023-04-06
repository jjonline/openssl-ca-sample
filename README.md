# openssl-ca-sample

Base on openssl selfsigned CA sample.

## Make CA

````
# make CA private key
openssl genrsa -aes256 -out CA/private/ca.key.pem 4096

# make CA cert request file
openssl req -config conf/ca.conf -key CA/private/ca.key.pem -new -x509 -days 3650 -sha256 -extensions v3_ca -out CA/certs/ca.cert.pem

# check CA cert
openssl x509 -noout -text -in CA/certs/ca.cert.pem

# make intermediate key/cert pair
openssl genrsa -aes256 -out intermediate/private/intermediate.key.pem 4096

# make intermediate request file
openssl req -config conf/intermediate.conf -new -sha256 -key intermediate/private/intermediate.key.pem -out intermediate/csr/intermediate.csr.pem

# use Root CA sign intermediate
openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 -notext -md sha256 -in intermediate/csr/intermediate.csr.pem -out intermediate/certs/intermediate.cert.pem

# check intermediate
openssl x509 -noout -text -in intermediate/certs/intermediate.cert.pem
openssl verify -CAfile CA/certs/ca.cert.pem intermediate/certs/intermediate.cert.pem

# make cerficate file chain
cat intermediate/certs/intermediate.cert.pem CA/certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
````

## Make domain validate cert

````
# make DV private key
openssl genrsa -aes256 -out intermediate/private/www.example.com.key.pem 2048

# make DV request file
openssl req -config conf/intermediate.conf -key intermediate/private/www.example.com.key.pem -new -sha256 -out intermediate/csr/www.example.com.csr.pem

# use CA/intermediate sign DV
openssl ca -config conf/intermediate.conf -extensions server_cert -days 3600 -notext -md sha256 -in intermediate/csr/www.example.com.csr.pem -out intermediate/certs/www.example.com.cert.pem

# remove private local password
openssl rsa -in intermediate/private/www.example.com.key.pem -out intermediate/private/www.example.com.decrypt.key.pem

# make nginx full-chain
cat intermediate/certs/www.example.com.cert.pem intermediate/certs/ca-chain.cert.pem > intermediate/certs/nginx-full-chain.pem
````
