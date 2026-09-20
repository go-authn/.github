<p align="center"><img src="https://raw.githubusercontent.com/go-authn/brand/main/social/go-authn.png" alt="go-authn" width="640"></p>

<h1 align="center">go-authn</h1>
<p align="center">Who is calling, in pure Go — from a security key to a Kerberos realm.</p>
<p align="center">
  <img src="https://img.shields.io/badge/Go-1.26-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img src="https://img.shields.io/badge/license-BSD--3--Clause-0A6E96?style=flat-square">
  <img src="https://img.shields.io/badge/cgo-none-0079A8?style=flat-square">
  <a href="https://go-authn.github.io"><img src="https://img.shields.io/badge/docs-go--authn.github.io-0079A8?style=flat-square"></a>
</p>

---

## What this is

Everything needed to answer *who is calling* — the factors, the policy that
weighs them, the directory the answers come from, and the protocols a service
speaks to ask.

The phrase is multi-**factor**, not multi-check. A passphrase and a recovery
code are both things you know: whoever learned one has usually learned the
other from the same place, and requiring both buys far less than the count
suggests. That distinction is what `mfa` exists to keep.

Everything here is pure Go with `CGO_ENABLED=0`, and nothing is
platform-specific. The macOS transport for a security key is
[`go-macos/fido`](https://github.com/go-macos/fido).

## Repos

**Where people come from**

| | Repo | |
|---|---|---|
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-directory.png" width="36"> | [`directory`](https://github.com/go-authn/directory) | Identities from SQL, LDAP or a configuration file, and what each one can *prove* — a password, an NT hash, a verifier, a key. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-authnd.png" width="36"> | [`authnd`](https://github.com/go-authn/authnd) | An LDAP server for people who are somewhere else, because almost everything speaks LDAP. With a `kerberos` block, a KDC for the same people. |

**What they can prove**

| | Repo | |
|---|---|---|
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-fido.png" width="36"> | [`fido`](https://github.com/go-authn/fido) | The FIDO client-to-authenticator protocol: CTAPHID, `GetInfo`, `MakeCredential`, `GetAssertion`. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-totp.png" width="36"> | [`totp`](https://github.com/go-authn/totp) | The six digits on a phone (RFC 6238), and the replay window a server needs around them. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-mfa.png" width="36"> | [`mfa`](https://github.com/go-authn/mfa) | The policy: how many factors, of which kinds, and what to say when some refuse. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-keyfactor.png" width="36"> | [`keyfactor`](https://github.com/go-authn/keyfactor) | A security key as one factor among others. |

**What a service speaks**

| | Repo | |
|---|---|---|
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-krb5.png" width="36"> | [`krb5`](https://github.com/go-authn/krb5) | The **accepting** half of Kerberos: verify an AP-REQ against a keytab, answer with the AP-REP `gokrb5` will not build, sign and seal. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-kdc.png" width="36"> | [`kdc`](https://github.com/go-authn/kdc) | The **issuing** half: AS-REQ and TGS-REQ, backed by a directory. Judged by MIT's own `kinit`. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-oidc.png" width="36"> | [`oidc`](https://github.com/go-authn/oidc) | The resource-server half of OpenID Connect: verify a bearer token, and say what it entitles. |

## How it is checked

Against the implementations that read the specifications independently, years
earlier — OpenLDAP's `ldapsearch`, MIT Kerberos' `kinit` and `gss-client`,
`pyotp`, Samba's `smbclient`. A stand-in built from our own reading of a
specification can only ever confirm that reading.

Those judges run in **every** lane that runs the tests, not only the first one:
without the judge a differential test skips, and a skip is not a failure.

## Links

- 📖 Documentation — <https://go-authn.github.io>
- 🎨 Brand assets — <https://github.com/go-authn/brand>

---

<p align="center"><sub>No cgo. No frameworks. Every repository gates its own coverage, and says what it enforces.</sub></p>
