# GPG 

## Symmetric Encryption

### **Encrypt**

`gpg -o newfilename.gpg --symmetric --cipher-algo AES256 filetocrypt.txt`

or

`gpg -o newfilename.gpg -c --cipher-algo AES 256 filetocrypt.txt`

### **Avoid to store passfrase in gpg-agent cache:**

`gpg -o new_filename.gpg --symmetric --cipher-algo AES256 --no-symkey-cache filetocrypt.txt`

### **Decrypt**

`gpg -o originalfilename.txt -d file.txt.gpg`

**Aromored option**

If you need to copy and paste your encrypted data (e.g. into an email), then use the `--armor` option. This will produce ascii armored text (base64 encoded) which is very portable. This option is mainly intended for sending binary data through email, not via transfer commands such as `ftp` or `shiftc` with the `-secure` option. The file size tends to be about 33% bigger than without this option, and encrypting the data takes about 10-15% longer. Taking AES256 as an example, you would simply use it like this:

`gpg --armor --symmetric --cipher-algo AES256 file.txt`

By default, this will produce file.txt.asc as the encrypted ascii armored file. You would then decrypt normally using something like:

`gpg -o file.txt -d file.txt.asc`

---

## Digitally Signing Symmetrically Encrypted Data

If you have set up a public/private key pair, you can use your private key to sign the data before symmetrically encrypting it.

To sign and symmetrically encrypt file.txt using `AES256`, use the `--sign` option like this:

`gpg --sign --symmetric --cipher-algo AES256 file.txt`

Then to verify the signature and decrypt: ` gpg -d file.txt.gpg`

> The -d option will automatically try to verify any signature and also decrypt

### **Factors that Affect Encrypt/Decrypt Speed**

The level of compression used when encrypting/decrypting affects the time required to complete the operation. There are three options for the compression algorithm: `none`, `zip`, and `zlib`.

- `--compress-algo none` or `--compress-algo 0`
- `--compress-algo zip` or `--compress-algo 1`
- `--compress-algo zlib` or `--compress-algo 2`

For example:

`gpg --output test.gpg --compress-algo zlib --symmetric test.out` 

If your data is not compressible, `--compress-algo 0` (`none`) gives you a performance increase of about 50% compared to `--compress-algo 1` or `--compress-algo 2`.

If your data is highly compressible, choosing the `zlib` or `zip` option will not only increase the speed by 20-50%, it will also reduce the file size by up to 20x. For example, in one test on a NAS system, a 517 megabyte (MB) highly compressible file was compressed to 30 MB.

The `zlib` option is not compatible with PGP 6.x, but neither is the cipher algorithm AES256. Using the `zlib` option is about 10% faster than using the `zip` option, and `zlib` compresses about 10% better than zip.

[NASA - Source of data](https://www.nas.nasa.gov/hecc/support/kb/using-gpg-to-encrypt-your-data_242.html)

---

## Asymmetric encryption

### **Generating A GPG Key Pair**

`gpg --gen-key`

After following the process you will be prompted with a summary, and the important parts to remember are:

- Name used to generate the key: `uid`
- The Key-id: `pub 2048R/...here is the key-id...` (in this example the first number is given by the key length we choose to generate)
- The Key fingerprint: `pub SNIP ... Key Fingerprint = ...`

### **Encrypt**

To encrypt a file you can use the `-e` (or `--encrypt`) option along with the `-r` (or `--recipient`) option, it's also possible to sign directly the encrypted file with `--sign` option and as previous mentioned the consideration of using `--armor` it depends on your needs. And remember that when making operation with key-pairs the use of `key-id` or `uid` and `mail@address` are all valid ways of comunicating with the gpg commands and make operations on specific keys.

`gpg -e -r key-id filename.txt` 

or by using the User Name (uid) choosen when generating the key-pair:

`gpg -e -r user-name file.txt`

This will produce an ecnrypted file called file.txt.gpg that only the recipient or obviously the owner of the conresponding pair public/private key can decrypt. If you need to change the name of the resulting encrypted file use the `-o` (or `--output`) option:

`gpg -o file.gpg -e -r uid file.txt`

> Note that if you had not verified the public key yet, you'll get a warning message to that effect when you try to encrypt data using that public key.

To specify the secret key to be used and the recipient public key:

`gpg -e -u sender-username -r receiver-usernaname file.txt`

### **Decrypt**

For the recipient to decrypt the encrypted data created in the steps above, they need to specify the output file using -o and also use the -d (or --decrypt) option. So to decrypt file.txt.gpg from above, the recipient (and owner of the private key) would execute this command:

`gpg -o file.txt -d file.txt.gpg`

The recipient will be prompted to enter the passphrase for their private key. If the correct passphrase is used, the decryption algorithm will proceed and the original data will be stored in file.txt. It's quite important to note that if no output file is specified, the decrypted ciphertext i.e. the plaintext (the original data) gets sent to standard out. So unless you pipe it to a file or another program, it will be displayed in your terminal and not stored to file.

---

## Encrypt a directory

As mentioned above compress a file is allways a better option for adding entropy to a detereminate object/file/system before encrypting it, and inn this situation is mandatory.

First turn the directory into a comopressed file:

- `tar czf new_compressed_file.tar.gz directory_name/`

To revert the previous command run: `tar xzf new_compressed_file.tar.gz`

It depends on your needs but for only storing an encrypted folder i suggest to encrypt it with `AES Symmetric` encryption. I will show both ways of doing it, with Symmetric and Asymettric encryption.

**Symmetric** 

Compress the folder as shonw above and procede as a normal symmetric encryption:

- `gpg -o compressedfile.gpg --symmetric --cipher-algo AES256 --no-symkey-cache compressedfile.tar.gz`

For the reverse process decrypt the file then decompress as shown above.

**Asymmetryc**

After generating the key-pair:

- `gpg -e -r user_name ~user_name/file_to_crypt.txt`

- `gpg -d -o ~user_name/decrypted_dir ~user_name/previously_crypted_file` 

Then if it's needed decompress the data.

---

## Keys Operations

### **Checks public Keyring**

`gpg --list-keys` or `gpg --list-public-keys`

**View the keys on your private (secret) keyring**

`gpg --list-secret-keys`


### **Export Public Keys**

`gpg --armor --export uid-of-choosen-key > public_key.asc`

Use the `--armor` specification only if it's needed, in case of transfer the key through mail, otherwise is not so usefull.

> Note that the name of the file (`public_key.asc`) is arbitrary, but informational in this case.

### **Import Public Key**

`gpg --import public_key.asc`

> note that when a public key is imported you should sign it as shown next.


## Validate Imported Public Keys By Signing

`gpg --edit-key key-uid`

Dsplay the fingerprint: `fpr`

Once verified with owner sign the key: `sign`

Doublecheck: `check`

Then exit with: `quit` or `Ctrl + d`

---

## Fingerprint

**Verify Identiy - Fingerprint**

Get the fingerprint of a public key:

- `gpg --fingerprint key-id` 
- `gpg --fingerprint email@address.com`

You can compare the result fingerprint string with the one givent to you or prentended to be the the original.

**Sign external Pub Key**

By doing that you authorize the key that you have been provided with:

- `gpg --sign-key key-id`
- `gpg --sign-key email@address.com`

---

## Export/Import Your Private Key

To save a raw data or ascii armored copy of the private key, you can use the `--export-secret-keys` option, below is a raw data example (include `--armor` for ascii version):

`gpg --export-secret-keys key-id > private_key`

`gpg --import private_key`

---

## Keyserver

To store your public key on a key server, use the `--keyserver` option along with the `--send-keys` option:

- `gpg --keyserver servername --send-keys key-id`

If someone wants to import that public key from the keyserver, they can use the `--recv-keys` option like so:

- `gpg --keyserver servername --recv-keys key-id`

Search into the key server:

- `gpg --keyserver key_server_name  --search-keys search-parameter`

Use this command to search by name or email address.

**Update Key information**

- `gpg --refresh-keys`

Fetch update informations from the key servers.

---

## Key Revocation

If you think that your public key is no longer valid, you can revoke it using a revocation certificate.

`gpg --output revocation_cert.asc --gen-revoke key-id`

> Note that you cannot create this revocation certificate after you've lost your private key! So it is recommended to create one at the same time that you create the key pair, then keep it in a really safe place. 

If you had to use this revocation certificate, you would do the following to update your keyring:

`gpg --import revocation_cert.asc`

Then run this command to update the keyserver:

`gpg --keyserver keyservername --send-keys key-id`

