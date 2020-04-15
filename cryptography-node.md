# Cryptography in node

## intro

### crypto module

Built in module.

- cipher/decipher classes
- DiffieHellman/ECDH exchange keys for asymmetric cryptography. (needed for https)
- hash (to generate hashes) given a hash output the input cannot be computed.
- HMAC for data in transit
- sign/verify
- random numbers, initialization vectors...

## Passwords

Passwords cause problem. They are susceptible to:

- Dictionary atacks, people choose poor passwords with common words.
- Credential stuffing, caused by using the same password in multiple websites.
- Data breaches, they make passwords public.

Hashing is the best way to protect passwords. Hashes are randomized, one way functions. Not all hash algorithms are secure, for example MD5 is now weak and should not be used for passwords. MD5 has a small output and does is not well protected against collisions.

The crypto module has a createHash function that accepts the hashing algorithm, then with the update method we provide the password and finally calling digest will compute the hash. digest and update can only be called once per instance of the hash.

```js
const crypto = require('crypto')
const hash = crypto.createHash('md5')
hash.update('pass')
const hashedPassword = hash.digest('hex')
```

A rainbow table is a db with all possible hash values to common strings.

sha(256) is longer and better protected but the rainbow table is still a problem.

### Good algorithms

From best to worst

- Argon2, (academically) new and not well integrated in node yet
- PBKDF2, good for enterprices
- scrypt
- bcrypt based on blowfish

TODO: Best practice for changing algoritm? update hash on successful login? double-hash?

### Salt

Salt is a random value added to the hash computation. Every hash need a different salt. Same passwords will have different hashes and thus, removing the rainbow table problem.

Salts can be generated with randomBytes, that returns a random buffer. Then we can use pbkdf2Sync (or async) to generate a hash with PBKDF2. Third parameter is the number of iterations. The returned hash is returned as a buffer.

```js
const crypto = require('crypto')
const password = 'pass'
const salt = crypto.randomBytes(256).toString('hex')
const hashedPassword = crypto.pbkdf2Sync(password, salt, 100000, 512, 'sha512')
```

Always store passwords hashed with a salt.

## Protect data at rest

Threats to data stored on disk (databases and files).

- Confidentiality. Show the right data to the right people.
- Integrity. Prevent data changes by an unauthorized party.
- Availability. Data available when needed.

### Encrypt

Symmetric encryption. Encrypt when saved decrypt when reading. Symmetric encription relies on one key (protect well). 

Tools in node: cypher/dechiper
`crypto.createCipheriv` iv = initialization vector. That creates a cipher then we can call `update` and `final` to encrypt the data. Both functions can only be called once or they throw. Do not use crypto.createCipher (without iv), it's deprecated and not secure.

AES-256 algorithm

- aes Advanced Encryption Standard
- 256 bits of output
- cbc cipher block chaining. chop data in blocks and encrypt each block. <http://bitly/crypto-basics>

Generate a key for encryption with scrypt

```js
// CONFIG
// common variables
const crypto = require('crypto')
// AES-256 algorithm
const algorithm = 'aes-256-cbc'
const key = crypto.scryptSync('password', crypto.randomBytes(32), 32)
// create random initialization vector
const iv = crypto.randomBytes(16)

// ENCRYPT
// cipher
const cipher = crypto.createCipheriv(algorithm, key, iv)
let encrypted = cipher.update('111-000-0000', 'utf8', 'hex')
encrypted = cipher.final('hex')

// DECRYPT
// decipher
const decipher = crypto.createDecipheriv(algorithm, key, iv)
let decrypted = decipher.update(encrypted, 'hex', 'utf8')
decrypted = decipher.final('utf8')
```

### Protect keys

Find a KMS. Store keys encrypted by a master key. Keys will be store in a "key store". Rotate keys. AWS kms, Azure key vault, vault (open source).

TODO: Best practice for rotating keys? re-encrypt with new key?

#### Vault

Master key is divided in pieces (shard) to be shared by different people/machines. With n shards the master key can be generated.

Authenticate with vault api with user+pass or certificates. Then request the key and connect to your db to decrypt or encrypt data.

## Protect data in transit

Data on the wire.

Threats:

- Lose confidentiality. Attacker accessing the data (read or change) while the data is in transit. Man-in-the-middle attack.
- Impersonation. Verify the identity of the other end of the communication.

Asymmetric encription -> encrypt with one key, decrypt with another key (public/private key pair) For solving impersonation.

HMAC -> hash based message authentication code. Hashes the data and sends it, then the receiver hashes anc compares. For knowing if data was changed.

Digital signatures -> hashing + asymmetric encryption. Big part of the PKI (backbone of https)

### Implementation

Diffie Hellman key exchange. How to generate a shared secret over a public channel.

`crypto.createDiffieHellman` creates a diffiehellman object and takes the number of bits of the key as param. (asymetric keys should be larger than symmetric). On the diffihellman object we can call `generateKeys()` to get the keys. For generating bob we use the prime number and the generator from alice. When we have both keys we can generate the shared secret.

The shared values over the channel are the prime and the generator numbers and the computed public key.

```js
const crypto = require('crypto')

const alice = crypto.createDiffieHellman(2048)
const aliceKey = alice.generateKeys()

const bob = crypto.createDiffieHellman(alice.getPrime(), alice.getGenerator())
const bobKey = bob.generateKeys()

// Generate the shared secret both will be equal
const aliceSecret = alice.computeSecret(bobKey)
const bobSecret = bob.computeSecret(aliceKey)
```

also useful: `crypto.generateKeyPair()`

HMAC. `crypto.createHmac()` with the algoritm and a secret (shared between both parties) then update it with data and calculate the signedhash with `digest()`. Now we can send the data along with the signedhash. Then Bob can calculate a signed hash with the same secret and compare.

```js
const crypto = require('crypto')
const hmac = crypto.createHmac('sha256', 'secret')
hmac.update('some data to be hashed')
const signedHash = hmac.digest('hex')
```

### HTTPS

Asymmetric encryption for exchanging keys and symmetric encryption for messages.

## Two factor auth

A factor is a method for identifying someone like password, token (physical object) or biometrics.

Time-based OTP. A secret (usually qr) is sent to the client (smartphone) and is used to generate tokens based on time (change fast). That token is used when authenticating.

### MFA Implementation

Using speakeasy and qrcode packages.

```js
// generate secret with speakeasy
const secret = speakeasy.generateSecret()
// generate qr based on secret and send it to the user
// data_url is the url of the image
qrcode.toDataURL(secret.otpauth_url, (err, data_url) => {})
// get the token from the user and validate with speakeasy
const verified = speakeasy.totp.verify({secret, encoding, token})
// when verified, store the secret in the user model
```

## Implementing cryptography

Add authentication service for authenticating requests (data in transit). Asymmetric encryption between servers/microservices (data in transit). Microservices for db access (data on rest) and a key store for the encryption keys.

Check <https://github.com/OWASP/NodeGoat> for the security errors.

- no password hashing
- no data in transit protection (uses http) we can use https module providing a key and a certificate.
- no data a rest protection.
