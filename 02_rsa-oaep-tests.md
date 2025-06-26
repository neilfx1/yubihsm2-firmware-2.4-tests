# RSA-OAEP Encryption/Decryption Test

This test demonstrates encrypting test data with RSA-OAEP and subsequently decrypting it using the YubiHSM2.

---

### 1. Create a working directory

Open a command prompt/terminal window and create a new directory such as `rsatest` and change to that directory.

```bash
mkdir rsatest && cd rsatest
```

---

### 2. Create a plain text file

Create a plain text file with an editor such as `vi`, `nano` or Notepad on Windows. Insert a few lines of text you will be able to recognise later (in this example it’s `testdata.txt`) and save this file in the same directory you changed to in step 1.

---

### 3. Generate RSA 2048-bit private key

Create an RSA2048 key with `openssl`. Replace `priv.key` with a filename you can locate later. It will be stored in the current directory.

```bash
openssl genrsa -out priv.key 2048
```

---

### 4. Extract the public key

Extract the public key from your private key. Replace `public.pem` with a filename you can locate later. It will be stored in the current directory.

```bash
openssl rsa -pubout -in priv.key -out public.pem
```

---

### 5. Encrypt the plaintext

Using openssl, encrypt your plain text file created in step 2, replacing `testdata.txt` in the command below. Replace `public.pem` with your public key created in step 4, and finally replace `encrypted.pem` with a suitable filename you can find in the next step. This process will encrypt the plain text file using RSA-OAEP using your public key.

Note that openssl does not require any options to encrypt using PKCS1v15 as this is the default operating mode.

```bash
openssl pkeyutl -in testdata.txt -encrypt -pubin -inkey public.pem \
-pkeyopt rsa_padding_mode:oaep \
-pkeyopt rsa_oaep_md:sha256 \
-pkeyopt rsa_mgf1_md:sha256 -out encrypted.pem
```

---

### 6. Import private key into YubiHSM2

Import the private key into the YubiHSM2 with `yubihsm-shell`. Replace `<session>` with your YubiHSM 2 session number, `rsaoaeptest` with a suitable label and `priv.key` with your private key created in step 3.

```bash
yubihsm> put asymmetric <session> 0 rsaoaeptest all all priv.key
```

Make a note of the object ID — this will be needed in the next step.

---

### 7. Decrypt with YubiHSM2

Decrypt the data you encrypted in step 5 using the following command. Replace `<session>` with your YubiHSM2 session number, `<objectID>` with the object ID created in step 6 (e.g. `0x93a1`), and `encrypted.pem` with the filename you created.

```bash
yubihsm> decrypt oaep <session> <objectID> encrypted.pem
```

After running the command you will either see a base64 encoded text block. To decode that output, copy it and run:

```bash
openssl base64 -d
```

Paste the string, press Enter, then Ctrl+D. Your plaintext will be revealed.

---

**Test Complete**  
You’ve now encrypted test data with RSA-OAEP and successfully decrypted it using YubiHSM2.
