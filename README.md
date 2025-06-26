# YubiHSM 2.4 Firmware PoC Tests

This repo provides a set of practical test cases for validating firmware 2.4+ on the YubiHSM2, using the YubiHSM SDK and OpenSSL.

> **Note**: These tests are **not intended for production** environments. All objects are created with unrestricted capabilities and full domain access for demonstration purposes only.

## 🔍 Tests Included

1. [PKCS1v15 Encryption/Decryption](01_pkvs1v15_tests.md)
2. [RSA-OAEP Encryption/Decryption](02_rsa-oaep-tests.md)
3. [Asymmetric Authentication](03_asmmetric-auth.md)
4. [AES-CBC Encryption/Decryption](04_aes_cbc-encryption_decryption.md)
5. [AES-ECB Encryption/Decryption](05_encrypt_decrypt-using-aesecb.md)
6. [Object Export under RSA-Wrap](06_export-under-wrap_RSA.md)
7. [HMAC Generation & Validation](07_hmac-generation-validation.md)


Each test includes:
- OpenSSL command sequences
- yubihsm-shell object creation & usage
- Sample expected outputs

## 🧰 Requirements

- YubiHSM2 (firmware 2.4+)
- YubiHSM SDK (v2024.09+)
- OpenSSL (latest)
- Linux, macOS, or WSL

## 📚 References

- [YubiHSM2 User Guide](https://docs.yubico.com/hardware/yubihsm-2/hsm-2-user-guide/)
- [YubiHSM2 SDK Releases](https://developers.yubico.com/YubiHSM2/Releases/)

---

## ⚠️ Disclaimer

This repo is provided for personal learning and internal test validation only. It is **not affiliated with or endorsed by Yubico.**
