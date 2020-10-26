# python-pkcs11

This is a high-level "more pythonic" interface using the PKCS#11 standard intefaces for HSM and smartcard devices. For more details check the following artifacts:

- Source: https://github.com/danni/python-pkcs11
- Documentation: http://python-pkcs11.readthedocs.io/en/latest/

Install `python-pkcs11` using the following command (ensure that all pre-requistcis are achieved as mentioned in the above documents):
```bash
$ pip3 install python-pkcs11
```

Next we discuss how to use `python-pkcs11` for accessing a SoftHSM2 token. 

### Step-1: Create a SoftHSM2 token

We have discussed SoftHSM2 in details [here](softhsm.md). Let's assume that we have configured the SoftHSM properly and created a token using softhsm2-util as given below:

```bash
$ softhsm2-util --init-token --slot 0  --label "test" --so-pin 1234 --pin 123456
```

### Step-2: Access the token using python-pkcs11

#### Step-2.1: Set environment variable 'PKCS11_MODULE'

`libsofthsm2.so` is the native PKCS#11 library for SoftHSM2. The python-pkcs11 expects that `PKCS11_MODULE` environment variable is set to the location of this native library.

```bash
$ export PKCS11_MODULE=/usr/lib/softhsm/libsofthsm2.so
```

#### Step-2.2: Python script to generate an AES key and encrypt data

This example is taken from the official document of python-pkcs11.

```python
import pkcs11
import os

# Initialise our PKCS#11 library
lib = pkcs11.lib(os.environ['PKCS11_MODULE'])
token = lib.get_token(token_label='test')

data = b'INPUT DATA'

# Open a session on our token
with token.open(user_pin='123456') as session:
    # Generate an AES key in this session
    key = session.generate_key(pkcs11.KeyType.AES, 256)

    # Get an initialisation vector
    iv = session.generate_random(128)  # AES blocks are fixed at 128 bits
    # Encrypt our data
    crypttext = key.encrypt(data, mechanism_param=iv)
    print(data)
    print(crypttext)
```

To explore more examples, chek the official [document](https://python-pkcs11.readthedocs.io/en/latest/index.html).