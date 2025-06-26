# Export an Object Under Wrap Using an RSA Keypair

The following process will transfer a symmetric key between two YubiHSM2s without exposing the wrap key data on screen. This can be useful for securely transmitting objects without the need for an in-person key exchange ceremony. This can be completed on all supported object types. To complete this test you will require two YubiHSM2s running firmware 2.4+. For the purposes of this test, one YubiHSM2 will be called `primary` and the other `backup`.

---

### 1. Create a working directory

Open a command prompt/terminal window and create a new directory such as `hsmtest` and change to that directory.

---

### 2. Generate symmetric key and IV

Create a new symmetric key with OpenSSL 32 bytes in length and an IV 16 bytes in length and save both files.
In this example the filename is `symmetric.key`. Then generate an IV 16 bytes in length, the example filename is `iv.txt`.

```bash
openssl rand -hex 32 > symmetric.key
openssl rand -hex 16 > iv.txt
```

---

### 3. Create a test data file

Generate a test text file containing a short line of test text (for example `This is a test file`) and save it as `testdata.txt`.

---

### 4. Generate RSA wrap key on backup HSM

Use `yubihsm-shell` to generate a new wrap key on the backup YubiHSM2. Replace `<sessionID>` with your session, `<label>` with a suitable label for the key.

```bash
generate wrapkey <sessionID> 0 "<label>" all all all rsa2048
```

Example output:

```
Generated Wrap key <wrapID>
```

---

### 5. Extract public wrap key

Extract the public key to a file called `pubwrap.pem`. Replace `<sessionID>` with your session and `<wrapID>` with the object ID from step 4.

```bash
get pubkey <sessionID> <wrapID> wrap-key pubwrap.pem
```

Output:

```
Stored public wrap key <objectID>
```

---

### 6. Import public key to primary HSM

Use `yubihsm-shell` to import the public key on the primary YubiHSM2. Replace `<sessionID>` with your session and `<label>` with a suitable label.

```bash
put pub_wrapkey <sessionID> 0 "<label>" all all all pubwrap.pem
```

Output:

```
Stored public wrap key <objectID>
```

---

### 7. Import symmetric key to primary HSM

On the primary YubiHSM2, create a new symmetric object. Replace `<sessionID>` with your session, `<label>` with a suitable label, and `symmetric.key` with the file from step 2.

```bash
put symmetric <sessionID> 0 <label> all all aes256 symmetric.key
```

Make a note of the stored symmetric key ID.

---

### 8. Export wrapped object from primary

Wrap the object created in step 7 into a file called `wrappedobj.enc`. Replace `<sessionID>`, `<publickeyID>`, and `<objectID>`.

```bash
get rsa_wrapped <sessionID> <publickeyID> symmetric-key <objectID> aes256 rsa-oaep-sha384 mgf1-sha256 wrappedobj.enc
```

---

### 9. Import wrapped object on backup

Use `yubihsm-shell` on the backup YubiHSM2 to import the wrapped symmetric object. Replace `<sessionID>` and `<wrapID>`.

```bash
put rsa_wrapped <sessionID> <wrapID> rsa-oaep-sha384 mgf1-sha256 wrappedobj.enc
```

---

### 10. Encrypt test data with OpenSSL

Use the symmetric key and IV generated in step 2 to encrypt the `testdata.txt` file. Replace `<symmetric>` and `<ivtext>`.

```bash
openssl enc -aes-256-cbc -in testdata.txt -K '<symmetric>' -iv '<ivtext>' -a
```

Example:

```bash
openssl enc -aes-256-cbc -in testdata.txt -K 010da1e5... -iv e46773af... -a
```

Copy the base64 output to the clipboard.

---

### 11. Decrypt with backup HSM

Use the symmetric key ID from step 9 to decrypt on the backup HSM.

```bash
decrypt aesecb <sessionID> <objectID> <base64>
```

Example:

```bash
yubihsm> decrypt aesecb 0 0x7ca5 gC7ypw/yl85IJ7VrpwfdUbvsgBLG+ZXOWc7eC1OlNeY=
```

Expected output:

```
This is a test.
```

---

### 12. Decrypt with primary HSM

Repeat the decryption on the primary HSM using the object ID from step 7.

```bash
decrypt aesecb <sessionID> <objectID> <base64>
```

Example:

```bash
yubihsm> decrypt aesecb 0 0x3fa2 gC7ypw/yl85IJ7VrpwfdUbvsgBLG+ZXOWc7eC1OlNeY=
```

Expected output:

```
This is a test.
```

---

**Test Complete**  
You have now successfully transferred an object under RSA wrap between two YubiHSM2s and validated decryption from both devices.
