PKCS#11 support for easy-rsa
============================

[verification needed] OpenSSL only ever operated on one key at time for any given command.
We're leveraging this fact to provide ubiquitous support for keys stored in pkcs#11 tokens
by creating a new form of key file format.  Instead of providing key material, it will effectively
become a pointer to a specific key in a specific device.  We'll start by 

Supported Operations

* *`build-ca` - Creates a CA on the configured device
* *`gen-req` - Create a request based off a private key on the device
* `sign-req` - Sign a request using the CA on the device
* `build-client-full` - Build a client certificate with a CA on the device
* `build-server-full` - Build a server certificate with a CA on the device
* `revoke` - Revoke a certificate signed by a CA on the device
* `renew` - Renew a certificate originally signed by a CA on the device


Environment Variables

* `PKCS11_MODULE` - The pkcs module to load
* `PKCS11_SLOT` - The slot to load objects from
* `PKCS11_PIN` - *INSECURE* useful for testing and automatically logs the user in
* `PKCS11_LABEL` - The label of the key to use.  (Not deduplicated!!)

## Thought Process

* EasyRSA creates a CA and creation is the perfect time to define everything we need to point to the key on the token
* Maybe setting up the PKCS#11 Module should be done during ca initialization.  Need to understand what the safe conf actually is other than an expanded config
  * Looks like the `vars` files are inside `./pki` and are wiped out on `init-pki`
* Should be able to convert a non-token based CA to a token.
* OpenSSL should be able to do all key operations on PKCS#11 but not all algorithms will be available with each token.  Good error messages will be important for debugging.

## TODO:

* [x] Create a self signed CA on a token
* [x] Create a CA CSR with a key on a token
* [ ] ... and import signed certificate
* [x] Sign a server certificate
* [x] Sign a client certificate
* [x] Revoke a certificate
* [x] Renew a certificate
* [x] BUG: Fix issue with `openssl: undefined symbol: C_GetFunctionList` when trying to revoke
* [x] Get PKCS11 module information from key file (if configured)
* [ ] If a key is being created on a device, ensure the label isn't alreay used by the same type of key
* [ ] Add check to ensure openssl pkcs11 engine is installed and library able to be found.
* [ ] Create command to extract a certificate from a key and bootstrap a new CA (maybe ask the user if that is what they want if the slot has everything that is needed)
