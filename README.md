# digital-namecard

I created this by reading this article: https://medium.com/@prestonlim/hack-your-life-how-to-make-your-own-digital-business-card-7a26ac400933

I added the following features:

- BASE64 encoded image in vcf file
- LinkedIn link on landing page


## Create an encrypted contact data blob

Below is a step‐by‐step guide on how to take your contact data, encrypt it, and produce a JSON payload (with Base64‑encoded salt, IV, and ciphertext). You can run this code in your browser’s console (or in a separate script) to generate the encrypted payload that you then include in your HTML’s data attribute.

Prepare Your Data Payload

Create a JavaScript object containing your contact info. For example:
```javascript
const contactData = {
    phone: "(+45) 6165 3935",
    emails: ["ole@covalente.dk", "ole@graae.dk"],
    mastodon: "https://social.vivaldi.net/@ovalente",
    threema: "https://threema.id/A3ABK2PF"
};
```
Setup Encryption Functions

Write a helper function to convert ArrayBuffer to Base64:
```javascript
function arrayBufferToBase64(buffer) {
    let binary = '';
    const bytes = new Uint8Array(buffer);
    bytes.forEach(b => binary += String.fromCharCode(b));
    return window.btoa(binary);
}
```


Encrypt the Data

Use the Web Crypto API to encrypt the JSON payload with the password "Password1!" using PBKDF2 (to derive an AES‑128 key) and AES‑CBC encryption. Here’s a complete function:
```javascript
async function encryptData(data, password) {
    const encoder = new TextEncoder();
    // Convert data object to JSON and then to a Uint8Array
    const dataStr = JSON.stringify(data);
    const dataBytes = encoder.encode(dataStr);

    // Generate a random salt (16 bytes) and initialization vector (iv) (16 bytes)
    const salt = window.crypto.getRandomValues(new Uint8Array(16));
    const iv = window.crypto.getRandomValues(new Uint8Array(16));

    // Import the password material for PBKDF2 key derivation
    const keyMaterial = await window.crypto.subtle.importKey(
        "raw",
        encoder.encode(password),
        "PBKDF2",
        false,
        ["deriveKey"]
    );

    // Derive a 128-bit AES key using PBKDF2
    const key = await window.crypto.subtle.deriveKey(
        {
            name: "PBKDF2",
            salt: salt,
            iterations: 100000,
            hash: "SHA-256"
        },
        keyMaterial,
        { name: "AES-CBC", length: 128 },
        false,
        ["encrypt"]
    );

    // Encrypt the data using AES-CBC
    const ciphertextBuffer = await window.crypto.subtle.encrypt(
        {
            name: "AES-CBC",
            iv: iv
        },
        key,
        dataBytes
    );

    // Convert salt, iv, and ciphertext to Base64 strings
    return {
        salt: arrayBufferToBase64(salt.buffer),
        iv: arrayBufferToBase64(iv.buffer),
        ciphertext: arrayBufferToBase64(ciphertextBuffer)
    };
}
```

Run the Encryption and Log the Result

Finally, call the function with your contact data and password:
```javascript
encryptData(contactData, "Password1!")
    .then(encryptedPayload => {
        console.log("Encrypted JSON payload:", JSON.stringify(encryptedPayload, null, 2));
        // You can now copy the JSON and embed it into your HTML like so:
        // <div id="contact-encrypted-data" data-oval='[YOUR_ENCRYPTED_JSON_HERE]'></div>
    })
    .catch(error => {
        console.error("Encryption error:", error);
    });
```

Use the Encrypted Payload

Copy the resulting JSON payload from the console and include it as the value of your custom attribute in your HTML. For example:
```html
<!-- filepath: /Users/olevalente/Source/repos/digital-namecard/index_private.html -->
<div id="contact-encrypted-data" data-oval='{"salt":"...","iv":"...","ciphertext":"..."}'></div>
```

With these steps, you now have a secure, encrypted JSON payload that can only be decrypted using the correct password ("Password1!") and the Web Crypto API code already implemented in your page.