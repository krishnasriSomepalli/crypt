# Code review report

## Review scope

This review focuses on maintainability, correctness, and AES/security accuracy. It intentionally does not spend time on visual design or UI polish. The project is treated as a sophomore undergraduate learning project, so recommendations are grouped into practical fixes and optional future learning topics rather than production-security requirements.

Files reviewed most closely:

- `encrypt2.html`
- `decrypt2.html`
- `index.html`
- `howto.html`
- `about.html`
- `README.md`

## Summary

The project has a clear purpose: encrypt and decrypt local text files in the browser without uploading user content. That is a good learning target because it combines browser APIs, file handling, user input, and cryptography concepts in one small app.

The main issues are not the idea or the UI. The main issues are that the crypto and file-handling code is embedded directly in HTML, duplicated between pages, and not covered by tests. The AES code also needs more careful documentation and validation so the project does not overclaim what it provides.

## What the project does well

### Clear local-first behavior

The app uses `FileReader` in the browser and does not require a backend. For a privacy-focused student project, that is a strong design choice because the selected file can be processed locally.

### Simple user workflow

The encrypt/decrypt flow is easy to understand:

1. Enter a key word.
2. Choose a `.txt` file.
3. Submit.
4. Download the result.

That simplicity makes the project approachable for users and for future maintainers.

### Good learning coverage

The project touches several useful web-development concepts:

- Static HTML/CSS pages.
- Browser file input.
- `FileReader`.
- Programmatic downloads.
- AES rounds, key expansion, and CTR mode.

That is a meaningful amount of learning for an undergraduate project.

### Some attribution is already present

`encrypt2.html` includes an attribution comment for Chris Veness's MIT-licensed JavaScript AES implementation. That is good and should be kept. The project should make the attribution consistent wherever that code is reused.

## What should be fixed

### 1. Move JavaScript out of the HTML files

Most of the important application logic lives inside `<script>` tags in `encrypt2.html` and `decrypt2.html`. This makes the files bulky and hard to scan.

Practical fix:

- Create a shared JavaScript file for AES/CTR logic.
- Create a small JavaScript file for file input and download helpers.
- Keep the HTML focused on page structure.

This is the highest-value refactor because it improves readability without changing the app's feature set.

### 2. Remove duplicated AES code

The AES classes are repeated in both the encryption and decryption pages. Duplicating cryptographic code is risky because a future fix might be applied in one page but missed in the other.

Practical fix:

- Keep one copy of the AES implementation in a shared file.
- Have both pages call the same shared functions.

This is a simple maintainability improvement, not a complex feature.

### 3. Add small correctness tests for AES behavior

The project would benefit from a few known-answer tests. This is especially important because AES is easy to get subtly wrong even when the code looks reasonable.

Practical fix:

- Add tests for at least one AES block cipher known-answer vector.
- Add one encrypt-then-decrypt test for a short ASCII text string.
- Add one encrypt-then-decrypt test for a string with punctuation or non-ASCII characters if the app intends to support that.

These tests do not need a large testing framework. Even a small standalone test page or Node script would help.

### 4. Fix or simplify UTF-8 decoding

`decrypt2.html` uses `new TextEncoder().decode(...)` in the UTF-8 decode helper. `TextEncoder` is for encoding, not decoding. The matching browser API is `TextDecoder`.

Why it matters:

- ASCII-only files may appear to work.
- Non-ASCII text may fail or rely on the fallback path.
- The code is harder to trust because the API name is wrong.

Practical fix:

- Use `TextDecoder` for decoding.
- Add a small test with text like `hello`, punctuation, and a non-ASCII character if Unicode support is desired.

### 5. Prevent default form submission

The forms call `encrypt()` and `decrypt()` from `onsubmit`, but the handlers do not clearly call `event.preventDefault()` or return `false`.

Why it matters:

- A normal form submit can reload or navigate the page.
- The download may still work in some browsers, but the behavior is fragile.

Practical fix:

- Attach submit listeners from JavaScript.
- Call `event.preventDefault()` before reading the file.

### 6. Be precise about file support

The app currently accepts `.txt` files and reads them with `readAsText`. That means the current implementation is text-file encryption, not general local-file encryption.

Practical fix:

- Keep the README and UI language specific to `.txt` files.
- Do not claim support for arbitrary images, PDFs, archives, or binary files unless the file handling is changed.

### 7. Clarify key handling

The UI says the user can enter a key of any size. Internally, the AES-128 path uses the first bytes of the password and pads missing bytes with zeroes before deriving the AES key.

Why it matters:

- Users may think a very long password is fully used.
- The implementation is educational, but the behavior should be documented honestly.

Practical fix:

- Document that this is simple educational key handling.
- If improving later, use a standard password-based key derivation function.

### 8. Document AES-CTR limitations

AES-CTR can encrypt and decrypt data, but by itself it does not detect tampering. If the encrypted text is edited or the wrong key is used, the app may output unreadable text instead of a clear "wrong key" error.

Practical fix for this project:

- Document this limitation clearly.
- Treat stronger authenticated encryption as a future learning topic, not an immediate requirement.

### 9. Improve nonce randomness if the project grows

The encryption code builds the CTR nonce from the current time plus `Math.random()`. For a learning project this is understandable, but CTR mode depends on never reusing the same key and counter stream.

Practical future improvement:

- If keeping custom AES-CTR, use `crypto.getRandomValues()` for random bytes.
- Store the nonce with the ciphertext, as the app already does conceptually.

This should be framed as a future improvement, not as something that must be solved before the project is useful as a class project.

### 10. Keep third-party attribution consistent

The encryption page contains an attribution comment for an MIT-licensed AES implementation. The decryption page appears to reuse the same implementation but does not show the same comment near the copied code.

Practical fix:

- Keep the attribution in the shared AES file.
- Mention the source in the README.
- Add a license file if the project continues to redistribute third-party MIT-licensed code.

## Nice-to-haves

These are useful but lower priority than the fixes above:

- Rename `encrypt2.html` and `decrypt2.html` to simpler names like `encrypt.html` and `decrypt.html`.
- Rename vague IDs and variables such as `test`, `call`, and `pom`.
- Use `const` and `let` consistently instead of `var`.
- Use `Blob` and `URL.createObjectURL()` for downloads instead of long `data:` URLs.
- Show a small status message after encryption or decryption.
- Add basic error handling for empty files, invalid Base64 input, and failed decryption.
- Format the HTML and JavaScript consistently.

## Suggested refactor path

A good next pass would be:

1. Extract the AES implementation into one shared file, keeping attribution at the top.
2. Extract helper functions for reading a text file and downloading text.
3. Replace inline `onsubmit` handlers with JavaScript event listeners.
4. Add a tiny test file for AES vectors and encrypt-then-decrypt behavior.
5. Update page names and variable names only after the logic is easier to follow.

This keeps the project close to its original scope while making it easier to understand, review, and improve.
