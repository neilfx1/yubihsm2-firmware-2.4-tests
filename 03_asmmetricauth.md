# Asymmetric Authentication into the YubiHSM 2

This test demonstrates setting up and using asymmetric authentication to access a YubiHSM2 session.

---

### 1. Create a working directory

**Open a command prompt/terminal window and create a new directory such as `authtest` and change to that directory.**

```bash
mkdir authtest && cd authtest
```

---

### 2. Generate an EC-P256 key pair

**The YubiHSM2 requires an uncompressed EC-P256 public key for authentication. To create a suitable keypair, run the following commands with `openssl`. Replace `priv.key` with your own private key filename:**

```bash
openssl genpkey -algorithm EC -out priv.key -pkeyopt ec_paramgen_curve:P-256 -pkeyopt ec_param_enc:named_curve
```

**Then run the following command to obtain the public key, again replace `priv.key` with your filename created above:**

```bash
openssl pkey -in priv.key -pubout
```

---

### 3. Copy the public key

**Copy the entire block of text to the clipboard, including the `-----BEGIN` and `-----END` statements.**

---

### 4. Import the public key as an authentication object

**Using `yubihsm-shell`, run the following command to import the asymmetric public key. Replace `<sessionID>` with your session number and `<label>` with a suitable label to identify the object on the YubiHSM2.**

```bash
put authkey_asym <sessionID> 0 <label> 1,2,3 generate-asymmetric-key,sign-pkcs sign-pkcs
```

**After pressing enter, paste the public key from the previous step and press Ctrl-D twice. Your output will look similar to the example below. Make a note of the stored authentication key object ID as this will be required every time you authenticate with this asymmetric key.**

```bash
yubihsm> put authkey_asym 0 0 asymmetric_test 1,2,3 generate-asymmetric-key,sign-pkcs sign-pkcs

-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEUM3+do+bFEmn9A2GkBKDMWRNm37I
YSOLIEf93nyAnUupL5KKBzNVSW0QleOim8WoUeLWjZwc56fFbQSj1io+fw==
-----END PUBLIC KEY-----
Stored Asymmetric Authentication key 0xbbda
```

---

### 5. Authenticate with asymmetric key

**Disconnect from the YubiHSM2 and reconnect. Use the following command to authenticate. Replace `<objectID>` with the object ID from the previous step (e.g. `0xbbda`). Replace `priv.key` with your private key created in step 2.**

```bash
session open_asym <objectID> priv.key
```

**You may receive a caution message advising the public key is not validated — this is because the keypair is self-signed for the purposes of this test.**

---

**Test Complete**  
You have now configured and used asymmetric key authentication to access the YubiHSM2.
