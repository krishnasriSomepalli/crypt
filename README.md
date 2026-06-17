# Encrypt | Decrypt

Encrypt | Decrypt is a static web app for encrypting and decrypting local text files in the browser. It was built as an undergraduate project to explore client-side file handling and the AES encryption algorithm.

The app does not upload files to a server. A user selects a `.txt` file, enters a key word, and the browser downloads a new encrypted or decrypted text file.

## What it does

- Encrypts local `.txt` files from the `Encrypt` page.
- Decrypts files produced by the app from the `Decrypt` page.
- Runs entirely as static HTML, CSS, and JavaScript.
- Uses an embedded JavaScript AES-CTR implementation.
- Downloads the result automatically with an `E` prefix for encrypted files and a `D` prefix for decrypted files.

## How to run it

No build step is required.

1. Clone or download the repository.
2. Open `index.html` in a browser.
3. Click **Encrypt** or **Decrypt**.
4. Enter the same key word for encryption and decryption.
5. Select a `.txt` file and submit the form.

You can also serve the folder locally:

```bash
python -m http.server
```

Then open `http://localhost:8000`.

## Project structure

| File | Purpose |
| --- | --- |
| `index.html` | Home page with links to encrypt and decrypt. |
| `encrypt2.html` | Encrypt form, file-reading logic, AES code, and download logic. |
| `decrypt2.html` | Decrypt form, file-reading logic, AES code, and download logic. |
| `howto.html` | User-facing instructions. |
| `about.html` | Project description. |
| `contact.html` | Contact page. |
| `main.css`, `crypt.css`, `decrypt1.css`, `common*.css` | Page styling. |
| `bootstrap.js`, `jquery-2.1.4.min.js` | Local JavaScript dependencies for page behavior. |
| `images/` | Static images used by the site. |

## Important limitations

This project is useful as a learning project, but it should not be treated as production-grade file encryption.

- The app is designed for text files, not arbitrary binary files.
- The encryption and decryption logic is embedded directly inside HTML files, which makes it harder to read, test, and maintain.
- The AES code is duplicated between the encrypt and decrypt pages.
- The current password-to-key handling is simple and educational. It does not use a modern password-based key derivation function.
- AES-CTR encryption does not detect whether a ciphertext was modified or whether the wrong key was entered.
- The project does not currently include automated tests against known AES test vectors.

## AES implementation note

The codebase contains a JavaScript AES implementation embedded in `encrypt2.html` and `decrypt2.html`. The encryption page includes an attribution comment for Chris Veness's MIT-licensed AES JavaScript implementation from movable-type.co.uk. If this implementation remains in the project, keep the attribution visible and document the source clearly.

## Suggested next improvements

Good next steps for this project are small and practical:

1. Move the AES code into a shared JavaScript file instead of duplicating it in both pages.
2. Move file-reading and download helpers into separate functions.
3. Add a few known-answer tests for AES and AES-CTR behavior.
4. Clarify error messages when decryption fails or when a non-text file is selected.
5. If the project ever becomes more than educational, consider using the browser's Web Crypto API instead of maintaining custom cryptography code.
