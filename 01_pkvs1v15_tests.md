# PKCS1v15 Encryption/Decryption Test

This test demonstrates encrypting data using OpenSSL with PKCS1v15 padding and decrypting it using a private RSA key stored in the YubiHSM2.

---

### 1. Open a command prompt/terminal window 
Create a new directory such as **pkcstest** and change to that directory.

```bash
mkdir pkcstest && cd pkcstest
```

### 2. Create a plain text file 
Use an editor such as vi, nano or notepad on Windows.  Insert a few lines of text you will be able to recognise later (in this example it’s **testdata.txt**) and save this file in the same directory you changed to in step 1.

### 3. Generate RSA 2048-bit private key
Create an RSA2048 key with openssl.  Replace **priv.key** with a filename you can locate later.  It will be stored in the current directory

```bash
openssl genrsa -out priv.key 2048
```

### 4. Extract the public key
Extract the public key from your private key.  Replace **public.pem** with a filename you can locate later.  It will be stored in the current directory,
```bash
openssl rsa -pubout -in priv.key -out public.pem
```

### 5. Encrypt the plaintext using the public key (PKCS1v15 is default)

Using openssl, encrypt your plain text file created in step 2, replacing **testdata.txt** in the command below.  Replace **public.pem** with your public key created in step 4, and finally replace **encrypted.pem** with a suitable filename you can find in the next step.  This process will encrypt the plain text file using pkcs1v15 using your public key. Note that openssl does not require any options to encrypt using pkcs1v15 as this is the default operating mode.

```bash
openssl pkeyutl -in testdata.txt -encrypt -pubin -inkey public.pem -out encrypted.pem
```

### 6. Import the private key into YubiHSM2

Import the private key into the YubiHSM2 with yubihsm-shell. Replace **<session>** with your YubiHSM 2 session number,  **pkcs15test** with a suitable label and **priv.key** with your private key created in step 3.

```bash
yubihsm> put asymmetric <session> 0 pkcs15test all all priv.key
```

Make note of the returned **Object ID** (e.g., `0x93a1`).

### 7. Decrypt with YubiHSM2

Decrypt the data you encrypted in step 5 using the following command.  Replace **<session>** with your YubiHSM2 session number, **<objectID>** with the object ID created in step 6, for example 0x93a1 and **encrypted.pem** with the filename you created in step 5.

```bash
yubihsm> decrypt pkcs1v1_5 <session> <objectID> encrypted.pem
```
After running the command you will either see the unencrypted text on screen or a base64 encoded text block depending on the OS used to complete the test.  To decrypt the base64 code, copy it to your clipboard and run the following openssl command

```bash
openssl base64 -d
```

After pressing enter on the above command, paste the base64 string into the window press enter and then press ctrl-d.  The base64 text will be decoded.  

---

**Test Complete**  
You’ve now encrypted test data with PKCS1v15 and successfully decrypted it using YubiHSM2.
