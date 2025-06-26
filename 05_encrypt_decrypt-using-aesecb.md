# AES-ECB Encryption/Decryption Test

This process will generate a new symmetric key with OpenSSL and import it into the YubiHSM2. That key will then be used to encrypt and decrypt test data using AES-ECB.

---

### 1. Create a working directory

Open a command prompt/terminal window and create a new directory such as `ecbtest` and change to that directory.

---

### 2. Generate symmetric key

Create a new symmetric key with OpenSSL 32 bytes in length and save the file. In this example the filename is `symmetric.key`.

```bash
openssl rand -hex 32 > symmetric.key
```

---

### 3. Import symmetric key into YubiHSM2

Use `yubihsm-shell` to import the symmetric key as a new object. Replace `<sessionID>` with your session, `<label>` with a suitable label for the key, and `symmetric.key` with the file you created in step 2.

```bash
put symmetric <sessionID> 0 <label> all all aes256 symmetric.key
```

Make a note of the stored symmetric key ID after running the command.

---

### 4. Create test data file

Generate a test text file containing a short line of test text (for example `This is a test file`) and save it as `testdata.txt`.

---

### 5. Encrypt with OpenSSL

Using OpenSSL, encrypt the test data file created in step 4. Open the `symmetric.key` file, copy the string. Replace `<symmetric>` with your key.

```bash
openssl enc -aes-256-ecb -in testdata.txt -K '<symmetric>' -a
```

Example:

```bash
openssl enc -aes-256-ecb -in testdata.txt -K 010da1e505006c6092014af904c87667e57b6eae36ad49fcf1207bb14a93bbac -a
```

Copy the base64 output to the clipboard.

---

### 6. Decrypt with YubiHSM2

Decrypt the test data on the YubiHSM2 with the following command. Replace `<sessionID>` with your session, `<objectID>` with the symmetric key ID from step 3, and `<base64>` as used in the previous step.

```bash
decrypt aesecb <sessionID> <objectID> <base64>
```

Example:

```bash
yubihsm> decrypt aesecb 0 0x7ca5 gC7ypw/yl85IJ7VrpwfdUbvsgBLG+ZXOWc7eC1OlNeY=
```

Expected output:

```
This is a block of test data.
```

---

### 7. Base64 encode test string

Generate a line of test text and encode it with base64 using OpenSSL.

macOS/Linux:

```bash
echo -n "this is a line of text to test encode and decode" | openssl base64
```

Windows:

```cmd
echo | set /p="this is a line of text to test encode and decode" | openssl base64
```

---

### 8. Encrypt with YubiHSM2

Use `yubihsm-shell` to encrypt the test text. Replace `<sessionID>` with your session, `<objectID>` with the symmetric key object created in step 3, and `<base64>` with the text generated in step 7.

```bash
encrypt aesecb <sessionID> <objectID> <base64>
```

Example:

```bash
encrypt aesecb 0 0x7ca5 dGhpcyBpcyBhIGxpbmUgb2YgdGV4dCB0byB0ZXN0IGVuY29kZSBhbmQgZGVjb2Rl
```

---

### 9. Decrypt with OpenSSL

Copy the output and save it in a new file called `encryptedecb.txt`. Using OpenSSL, enter the following command to decrypt the data. Replace `<symmetric>` with the value generated in step 2.

```bash
openssl enc -d -aes-256-ecb -in encryptedecb.txt -K <symmetric> -a -nopad
```

Example:

```bash
openssl enc -d -aes-256-ecb -in encryptedecb.txt -K 010da1e505006c6092014af904c87667e57b6eae36ad49fcf1207bb14a93bbac -a -nopad
```

---

### 10. Decode final base64

Copy the output to the clipboard and run the following command. Replace `<copiedtext>` by pasting the output when required.

```bash
echo <copiedtext> | openssl base64 -d
```

Example:

```bash
echo dGhpcyBpcyBhIGxpbmUgb2YgdGV4dCB0byB0ZXN0IGVuY29kZSBhbmQgZGVjb2Rl | openssl base64 -d
```

Expected output:

```
this is a line of text to test encode and decode
```

---

**Test Complete**  
You have successfully encrypted and decrypted AES-256-ECB test data using the YubiHSM2 and OpenSSL.
