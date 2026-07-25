# webauthn

EDN substrate for WebAuthn / passkey registration and assertion flows.

Cryptographic verification and browser APIs are host ports. This repo keeps
challenge, credential, assertion, and passkey PRF envelope shapes portable.

For the raw W3C WebAuthn relying-party ceremony spec-as-data substrate (zero
deps, no host assumptions), see [kotoba-lang/org-w3-webauthn](https://github.com/kotoba-lang/org-w3-webauthn).
This repo is the result-shape/substrate layer consumed by
[kotoba-lang/authentication](https://github.com/kotoba-lang/authentication).

## `webauthn.adapters.edge` — the ceremony crypto, for real

The portable namespaces above model the *shapes*. `webauthn.adapters.edge`
(CLJS-only) does the actual WebAuthn Level 2 verification on WebCrypto — no
npm, no platform binding — so a Cloudflare Worker can accept passkeys:

```clojure
(edge/verify-registration! {:rp-id "authn.example" :origin "https://authn.example"}
                           {:client-data-json-b64url … :attestation-object-b64url …
                            :challenge <the one you issued>})
;=> Promise<{:ok true :credential-id … :public-key-b64 … :sign-count n}>

(edge/verify-authentication! config
                             {:client-data-json-b64url … :authenticator-data-b64url …
                              :signature-b64url … :challenge … :public-key-b64 <stored>})
;=> Promise<{:ok true :sign-count n :user-verified? bool}>
```

It is a pure function of its inputs plus WebCrypto. **Storage, key custody and
session issuance are deliberately the caller's** — the prior art this is ported
from fused ceremony verification, a KV schema, an AES-GCM key-wrapping scheme
and CACAO minting into one namespace, which made all four untestable without
Cloudflare bindings.

Two responsibilities the caller must not skip, because they need persistence
this namespace does not have:

- **make the challenge single-use.** A replayable challenge is a replayable
  sign-in. This adapter checks that the presented challenge equals the one you
  issued; ensuring it can only be issued once is a storage property.
- **apply `sign-count-ok?`** against your stored baseline, then persist the new
  count (WebAuthn L2 §7.2 step 19, clone detection).

Verified against `webauthn.virtual-authenticator` — a P-256 authenticator that
really signs, including ceremonies that misbehave on purpose:

```bash
nbb --classpath "src:test:../org-chainagnostic-cacao/src" test/webauthn/edge_smoke.cljs
```

