# AES-CBC Encryption/Decryption Test

This process will generate a new symmetric key with OpenSSL and import it into the YubiHSM2. That key will then be used to encrypt and decrypt test data using AES-CBC.

---

### 1. Create a working directory

Open a command prompt/terminal window and create a new directory such as `aestest` and change to that directory.

---

### 2. Generate symmetric key and IV

Create a new symmetric key with OpenSSL 32 bytes in length and an IV 16 bytes in length and save both files. In this example the filename is `symmetric.key`, then generate an IV 16 bytes in length, the example filename is `iv.txt`.

```bash
openssl rand -hex 32 > symmetric.key
openssl rand -hex 16 > iv.txt
```

---

### 3. Import key to YubiHSM2

Use `yubihsm-shell` to import the symmetric key as a new object. Replace `<sessionID>` with your session, `<label>` with a suitable label for the key, and `symmetric.key` with the file you created.

```bash
put symmetric <sessionID> 0 <label> all all aes256 symmetric.key
```

Make a note of the stored symmetric key ID after running the command.

---

### 4. Create test data

Generate a test text file containing a short line of test text (for example `This is a test file`) and save it as `testdata.txt`.

---

### 5. Encrypt the test data with OpenSSL

Using OpenSSL, encrypt the test data file created in step 4. Open the `symmetric.key` file and `iv.txt` file created in step 2, copy each string. These will be required in the following command. Replace `<symmetric>` with your key and `<ivtext>` with the IV.

```bash
openssl enc -aes-256-cbc -in testdata.txt -K '<symmetric>' -iv '<ivtext>' -a
```

Example:

```bash
openssl enc -aes-256-cbc -in testdata.txt -K 010da1e505006c6092014af904c87667e57b6eae36ad49fcf1207bb14a93bbac -iv e46773af01080c64fe2f9b0cc771b045 -a
```

Copy the base64 output to the clipboard.

---

### 6. Decrypt with YubiHSM2

Decrypt the test data on the YubiHSM2 with the following command. Replace `<sessionID>` with your session, `<objectID>` with the symmetric key ID from step 3, and `<ivtext>` and `<base64>` as used in the previous step.

```bash
decrypt aescbc <sessionID> <objectID> <ivtext> <base64>
```

Example:

```bash
yubihsm> decrypt aescbc 0 0x7ca5 e46773af01080c64fe2f9b0cc771b045 5N3Ju9ZDuhMK+rbwvHzh0FC5sM0eVtu0NsRzhkNdrho=
```

Expected output:

```
This is a block of test data.
```

---

### 7. Base64 encode sample text

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

### 8. Encrypt base64 text with YubiHSM2

Use yubihsm-shell to encrypt the test text.  Replace `<sessionID>` with your session, `<objectID>` with the symmetric key object created in step 3, `<ivtext>` from the hex value generated in step 2 and finally `<base64>` with the text generated in step 7.

```bash
encrypt aescbc <sessionID> <objectID> <ivtext> <base64>
```

Example:

```bash
encrypt aescbc 0 0x7ca5 e46773af01080c64fe2f9b0cc771b045 dGhpc2lzYXRlc3RmaWxlLg==
```

---

### 9. Decrypt encrypted output with OpenSSL

Copy the output and save it in a new file called `encryptedcbc.txt`. Then run:

```bash
openssl enc -d -aes-256-cbc -in encryptedcbc.txt -K <symmetric> -iv <ivtext> -a -nopad
```

Example:

```bash
openssl enc -d -aes-256-cbc -in encryptedcbc.txt -K 010da1e505006c6092014af904c87667e57b6eae36ad49fcf1207bb14a93bbac -iv e46773af01080c64fe2f9b0cc771b045 -a -nopad
```

---

### 10. Decode final base64

Copy the output to the clipboard and run the following:

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
You have successfully encrypted and decrypted AES-256-CBC test data using YubiHSM2 and OpenSSL.
