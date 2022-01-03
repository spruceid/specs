# Spruce Specs
This is a (WIP) public list of select specifications that are actively used or
pending consideration in the Spruce stack for decentralized identity. This
repository also contains explainers, ideas for specifications, and early
drafts. Items are selected based on their relevance to decentralized identity.
For example, JSON is not listed--while it is used liberally across the
codebase, it is not germane.

Spruce is not a standards development organization nor industry association,
seeking to become neither. Instead, Spruce recognizes existing organizations
such as W3C, IETF, NIST, ISO, CASA, and DIF, preferring to work through those
and in the open whenever possible. Spruce also strongly prefers to contribute
to and support existing work items to converge efforts over starting new ones.

# In Use (WIP)
- SIWE
- CACAO
- W3C Verifiable Credentials
- W3C Decentralized Identifiers
    - did-key
    - did-pkh (relies on CAIP-10)
    - did-web
    - did-webkey
    - Additional methods supported in `ssi`/`didkit` can be found [here](https://spruceid.dev/docs/didkit/did-methods)
- JSON Web Tokens (JWTs)
- Linked Data Proofs (relies on RDF)
- Additional proof types supported in `ssi`/`didkit` can be found [here](https://spruceid.dev/docs/didkit/did-methods#proof-types-verifiable-so-far)
- Authorization Capabilities (ZCAPs, including ZCAP-LD)
- Additional specifications, test suites, and cryptographic dependencies can be found [here](https://spruceid.dev/docs/didkit/specs_and_deps)

# Under Consideration (WIP)
- DIF Presentation Exchange
- DIF Identity Hubs
- OpenID Connnect SIOPv2

# Open Problems
The following problems present opportunities for standardization of some kind.
As mentioned above, it is preferrable not to introduce a new specification
unless absolutely necessary, and work should continue under existing work items
if reasonably possible. Focusing on problems allow more potential for holistic
solutions to emerge versus making the decision to start a draft spec upfront.

## Metadata for Verification Methods
Blockchain wallets that support signing are unlikely to support issuance of
standard JWS payloads in the near future, so it is far more pragmatic to build
new signature suites that can work with their existing outputs, such as
EIP-191, EIP-712, or Phantom's
[`signMessage()`](https://docs.phantom.app/integrating/signing-a-message).
These outputs can then be embedded into a data structure (such as W3C
Verifiable Credentials) with metadata that specify exact verification
methods, which will typically begin with detaching the embedded output.

Outside the realm of blockchain users, there are new signature schemes that do
not have a clear fit with JWS, as evidenced by the work on JSON Web Proofs
(JWPs) linked below. Solving this problem confers the ability to express those
as well.

The specification(s) solving this problem needs to outline the data model of the
embedded output, preferably in a way friendly for reliance by a variety of
other specifications, ideally non-JSON based ones as well (e.g., using CBOR,
ASN.1, protobuf, Cap'n Proto, IPLD). JSON-only is acceptable but not preferred.
Perhaps CDDL could be used to express the data model more generally. The
trickiest, most dangerous, and most ambitious part of solving this problem
would be the level of cryptoagility potentially afforded by new data models,
potentially creating new opportunities for disaster. This could be mitigated
using strict profiling, such as `(secp256k1,ecdsa,keccak,eip191,json)` for
Ethereum or `(ed25519,eddsa,sha256,signmsg,json)` for Solana, but at the
expense of increased complexity of the solution.

Instead of embedding the output within a data model, the verification method
metadata and signature could be represented as JWT or JWP claims under new
algorithms.

From the [VC data model](https://www.w3.org/TR/vc-data-model/#example-a-simple-example-of-a-verifiable-credential):
```json
  // ...
  "proof": {
    "type": "RsaSignature2018",
    "created": "2017-06-18T21:19:10Z",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "https://example.edu/issuers/565049/keys/1",
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..TCYt5X
      sITJX1CxPCT8yAV-TVkIEq_PbChOMqsLfRoPsnsgw5WEuts01mq-pQy7UJiN5mgRxD-WUc
      X16dUEMGlv50aqzpqh4Qktb3rk-BuQy72IFLOqV0G_zS245-kronKb78cPN25DGlcTwLtj
      PAYuNzVBAh4vGHSrQyHUdBBPM"
  }
  // ...
```

If solved, signing interoperability between signing methods (blockchain
accounts, PGP keys, JOSE-based) and data formats (W3C Verifiable Credentials,
ISO 18013-5 mDL, IPLD claims, etc.) would increase dramatically. Furthermore,
LD-Proofs would have a better foundation and basis by relying on a more general
purpose data model for verification method metadata.

### Related Work
- [JWS-CT](https://datatracker.ietf.org/doc/html/draft-jordan-jws-ct-00) is the
  closest draft specificaencountered so far, which uses JCS and JWS detached
  mode to allow for an embeddable representation.
- [JWPs](https://json-web-proofs.github.io/json-web-proofs/draft-jmiller-json-web-proof.html)
  seek to extend JWS with multiple claims.
- [XML-DSIG](https://www.w3.org/TR/xmldsig-core1/) attempted to do this with
  XML, but its rollout was marred with implementation difficulties and security bugs
  related to flexible canonicalization. What can we do to prevent these classes of issues?
- [EthereumEIP712Signature2021](https://w3c-ccg.github.io/ethereum-eip712-signature-2021-spec/)
  defines how to, e.g., issue a W3C Verifiable Credential from an Ethereum EOA account.
