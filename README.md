# Download PDFs protected by [pdf-embedder](https://wordpress.org/plugins/pdf-embedder/) Wordpress Plugin

1. Open the website through burp

2. Activate Intercept

3. Forward requests until you see a POST containing https://example.com/?pdfemb-serveurl=...

4. Select this request

6. In the panel below, right click the request and select "Copy as Curl (bash)"

7. Paste it in bash, remove the `-i` flag, add `-o out.enc` in the end of the line, run it

8. Turn off Intercept

9. Navigate to the originaal page where the PDF is displayed, open DevTools, Console; type in the console: `pdfemb_trans.k`. This is the RC4 key used to encrypt the file

10. Use this script:

```python
#!/usr/bin/env python3
import sys

def rc4(key: bytes, data: bytes) -> bytes:

    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) & 0xFF
        S[i], S[j] = S[j], S[i]

    i = j = 0
    out = bytearray()
    for byte in data:
        i = (i + 1) & 0xFF
        j = (j + S[i]) & 0xFF
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) & 0xFF]
        out.append(byte ^ K)
    return bytes(out)

def main():
    if len(sys.argv) != 4:
        print(f"Usage: {sys.argv[0]} <rc4_key_ascii> <encrypted.pdf> <decrypted.pdf>")
        sys.exit(1)

    key_ascii, enc_path, dec_path = sys.argv[1], sys.argv[2], sys.argv[3]


    with open(enc_path, "rb") as f:
        ciphertext = f.read()

    key = key_ascii.encode('utf-8')


    plaintext = rc4(key, ciphertext)


    with open(dec_path, "wb") as f:
        f.write(plaintext)

    print(f"✅ Decrypted PDF written to {dec_path}")

if __name__ == "__main__":
    main()
```

like this:

`python3 decrypt.py <pdfemb_trans.k value> <out.enc> <filename.pdf>`

that's it :)




