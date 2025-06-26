# Testing HMAC Capability

This process will generate a new HMAC key with OpenSSL and import it into the YubiHSM2. That key will then be used to perform an HMAC signing operation with OpenSSL and the YubiHSM2.

---

### 1. Generate HMAC key

Open a command prompt and create a new HMAC key with OpenSSL 16 bytes in length. Copy the output to the clipboard. An example is shown below.

```bash
openssl rand -hex 16
```

Example output:

```
efa836cb379c4057160a502a4c88d839
```

---

### 2. Import HMAC key into YubiHSM2

Using `yubihsm-shell`, import the HMAC key into the YubiHSM2. Replace `<sessionID>` with your session and `<randomtext>` with the text you copied to the clipboard in step 1.

```bash
put hmackey <sessionID> 0 all all all hmac-sha256 <randomtext>
```

Example:

```bash
put hmackey 0 0 all all all hmac-sha256 efa836cb379c4057160a502a4c88d839
```

---

### 3. Perform HMAC in OpenSSL

Use OpenSSL to complete a HMAC operation on a word, for example `testing`. Replace `<randomtext>` with the key from step 1. After typing the command below, press Enter, then type the test sentence, then press Ctrl-D twice.

```bash
openssl dgst -sha256 -mac hmac -macopt hexkey:<randomtext>
```

Example:

```bash
openssl dgst -sha256 -mac hmac -macopt hexkey:efa836cb379c4057160a502a4c88d839
```

Example input:

```
testing
```

Example output:

```
SHA2-256(stdin)= 97c70d29506a644bc0bc99c04157f861bfb6bfd478b6fe36b89651f29b6e0af1
```

Make a note of the output value to compare later in this test.

---

### 4. Convert input string to hex

Use a hex converter such as [RapidTables](https://www.rapidtables.com/convert/number/ascii-to-hex.html) to convert your test string into hexadecimal.

- Input: `testing`  
- Output: `74657374696E67`

Set the output delimiter string to **None** on the site, then copy the hex output to the clipboard.

---

### 5. Perform HMAC in YubiHSM2

Use `yubihsm-shell` to perform a HMAC operation validating the imported HMAC key output matches the output from OpenSSL. Replace `<sessionID>` with your session ID and `<hmackey>` with the key generated in step 2. Replace `<hmacvalue>` with the hex output from step 4.

```bash
hmac <sessionID> <hmackey> <hmacvalue>
```

Example:

```bash
hmac 0 0x275f 74657374696E67
```

Expected output (matching OpenSSL):

```
97c70d29506a644bc0bc99c04157f861bfb6bfd478b6fe36b89651f29b6e0af1
```

---

**Test Complete**  
If the test was successful, the output from OpenSSL and YubiHSM2 will match.
