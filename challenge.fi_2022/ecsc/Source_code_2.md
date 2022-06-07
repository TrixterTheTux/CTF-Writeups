# Source Code 2
```
The developer has created a fancy new client-side authenticator app to an upcoming website. Are you able to find the solution for the correct passcode?
```
Attachments: `sourcecode2.zip`  

We are given a minimal web page with a keypad. Looking at the source code, only the `numpad.js` seems relevant for solving the challenge.

We find out the flag is there encrypted with AES, and the passphrase has the following properties:
* Made up only of numbers (= the keypad)
* The length is limited to 6 characters

Because of this, we know the passphrase is either a number between 0-999999 or 0-99999 (with left padded zeros), which we can easily bruteforce in the browser console (or use the same source code file):
```js
for(let i = 0; i <= 999999; i++) {
    const code = i.toString().padStart(6, "0");
    if (i % 10000 === 0) console.log(`Bruteforce @ ${i} (${code})`);

    try {
        const decrypted = CryptoJS.AES.decrypt("U2FsdGVkX19mkDLn6UO1O5cEEREmBiHo+gKMHyJklmvI9Miuc+U7CKhfuXiN0nGF", code).toString(CryptoJS.enc.Utf8);
        if (decrypted.startsWith("GENZ")) {
            console.log(`RESULT: ${decrypted} (${code})`);
            break;
        }
    } catch (err) {}
}
```
Note that if had this failed, we could've commented out the `.padStart(6, "0")` part due to the code having a prefix with zeros.

We find out the keypad's code is `108358`, and decrypting it gives us the flag `GENZ{GenzBadassHaxGoodz!}`.