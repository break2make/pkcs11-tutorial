# PKCS#11

*According to [Wikipedia](https://en.wikipedia.org/wiki/PKCS_11):*

> The PKCS #11 standard defines a platform-independent API to cryptographic tokens, such as hardware security modules (HSM) and smart cards, and names the API itself "Cryptoki" (from "cryptographic token interface" and pronounced as "crypto-key" - but "PKCS #11" is often used to refer to the API as well as the standard that defines it).
>
>The API defines most commonly used cryptographic object types (RSA keys, X.509 Certificates, DES/Triple DES keys, etc.) and all the functions needed to use, create/generate, modify and delete those objects.

PKCS#11 cryptographic token interface base specification is maintained by [OASIS](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html).

For an overview of PKCS#11 specification, please check [this](https://www.dnssec.cz/files/nic/doc/hsm.pdf). We suggest to read this document before going thorugh the specification as it is easy to understand.  

## Usage of PKCS#11

Steps to use crypto hardware tokens with pkcs11 interface specification are:

1. Open a session with the target token, which returns a session handler.
1. Log into the token using PIN
1. Now, access the objects stored inside the token
1. Logout
1. Close the session

[source](http://www.lsv.fr/Publis/PAPERS/PDF/BCFS-ccs10.pdf)

Once a session is initiated, the application may access the objects stored on the token, such as keys and certificates. However, access to the objects is controlled. Objects are referenced in the API via handles, which can be thought of as pointers to or names for the objects. In general, the value of the handle, e.g. for a secret key, does not reveal any information about the actual value of the key. Objects
have attributes, which may be bitstrings e.g. the value of a key, or Boolean flags signalling properties of the object, e.g. whether the key may be used for encryption, or for encrypting other keys. New objects can be created by calling a key generation command, or by ‘unwrapping’ an encrypted key packet. In both cases a fresh handle is returned.

## Object attributes

[source](http://www.lsv.fr/Publis/PAPERS/PDF/BCFS-ccs10.pdf)

When a function in the token’s API is called with a reference to a particular object, the token first hecks that the attributes of the object allow it to be used for that function. For example, if the encrypt function is called with the handle for a particular key, that key must have its encrypt attribute
set. To protect a key from being revealed, the attribute sensitive must be set to true. This means that requests to view the object’s key value via the API will result in an error message. Once the attribute sensitive has been set to true, it cannot be reset to false.

Protection of the keys essentially relies on the sensitive and extractable attributes. An object with an extractable attribute set to false may not be read by the API, and additionally, once set to false,
the extractable attribute cannot be set to true.

## Attacsk on PKCS#11

TODO 

## Experiment on PKCS#11

We have added two addition totorials for this purpose, please the read those:

- [SoftHSM](softhsm.md) (simulated HSM token for exploring PKCS#11 API)
- [python-pkcs11](python-pkcs11.md) (high-level pythonic API using base PKCS#11 API)

## Useful Resource

- A dive into SoftHSM. [Link](https://medium.com/@clydecroix/a-dive-into-softhsm-e4be3e70c7bc)
- SoftHSM Documentation v2 [Link](https://wiki.opendnssec.org/display/SoftHSMDOCS/SoftHSM+Documentation+v2)
- Module 7: Simulating hardware security integration. [Links](https://docs.aws.amazon.com/greengrass/latest/developerguide/console-mod7.html)
- An introduction to the use of HSM. NLnet Labs document 2008-draft. [source](https://www.dnssec.cz/files/nic/doc/hsm.pdf)
- Attacking and Fixing PKCS#11 Security Tokens [source](http://www.lsv.fr/Publis/PAPERS/PDF/BCFS-ccs10.pdf)
