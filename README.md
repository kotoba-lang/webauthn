# webauthn

EDN substrate for WebAuthn / passkey registration and assertion flows.

Cryptographic verification and browser APIs are host ports. This repo keeps
challenge, credential, assertion, and passkey PRF envelope shapes portable.

For the raw W3C WebAuthn relying-party ceremony spec-as-data substrate (zero
deps, no host assumptions), see [kotoba-lang/org-w3-webauthn](https://github.com/kotoba-lang/org-w3-webauthn).
This repo is the result-shape/substrate layer consumed by
[kotoba-lang/authentication](https://github.com/kotoba-lang/authentication).
