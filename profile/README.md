<p align="center"><img src="https://raw.githubusercontent.com/go-authn/brand/main/social/go-authn.png" alt="go-authn" width="640"></p>

<h1 align="center">go-authn</h1>
<p align="center">Authentication in pure Go — the factors that prove somebody is present, and the directory that says who they are and what can prove them.</p>
<p align="center">
  <img src="https://img.shields.io/badge/Go-1.26-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img src="https://img.shields.io/badge/license-BSD--3--Clause-0A6E96?style=flat-square">
  <img src="https://img.shields.io/badge/cgo-none-0079A8?style=flat-square">
  <a href="https://github.com/go-macos"><img src="https://img.shields.io/badge/transport-go--macos-0079A8?style=flat-square"></a>
</p>

---

## What this is

Two halves of one question, and they are not the same half.

**Is this person here, now?** The phrase is multi-**factor**, not multi-check. A
passphrase and a recovery code are both things you know: whoever learned one has
usually learned the other from the same place, and requiring both buys far less
than the count suggests. `mfa` is the part that has to be right whatever the
factors are — how many answers, of which kinds, and what to say when some of
them fail — and `fido`, `keyfactor` and `totp` are answers it can ask for.

**Who is this person, and what would prove them?** A site's people are already
written down somewhere: a database, an LDAP server, a file. `directory` is one
model over all three, and it carries what *proves* each person rather than a
password field, because what proves somebody is not one thing — an LDAP bind
cannot authenticate an SMB session, and neither can a bcrypt. `oidc` verifies a
token into an identity for the things that speak that instead, and `authnd`
serves the whole lot back out over LDAP, because almost everything speaks LDAP.
`krb5` is the **accepting** half of Kerberos, for the services that speak it: an
AP-REQ verified against a keytab, answered with the AP-REP that proves the
service is genuine — and then messages signed and sealed under the session key,
which is what `sec=krb5`, `krb5i` and `krb5p` are. It issues nothing; `kdc` is
the **issuing** half, AS-REQ and TGS-REQ backed by the same `directory`, judged
by MIT's own `kinit`. Two halves, two repositories, so a service that only
accepts tickets does not link a KDC.

## What proves somebody is not one thing, and now there is a table of it

That sentence used to be an assertion here. It is a measurement now.
[`go-fileshare/fileshare`](https://github.com/go-fileshare/fileshare) serves one
set of people over five protocols out of one `directory`, and asks a different
question of each person for each one — `canServeUser` in its `protocol.go`:

| Serving a person over | needs, from their directory entry |
|---|---|
| **SMB** | `NTHash`. NTLMv2 works from the password or its `MD4(UTF16LE(password))`, and nothing else will do. |
| **S3** | `Password` — the secret itself. SigV4 proves possession by computing an HMAC **from** it. |
| **WebDAV** | `Verifier` **or** `Password`. A source that can only *check* a password is enough here. |
| **SFTP** | `PublicKeys`, or a trusted user CA that makes everybody able to present a certificate. |
| **NFS** | a Kerberos ticket: `sec=krb5` through `krb5` and a keytab, not a directory credential at all. |

SMB and S3 are the pair worth reading twice. They are neighbours, not copies: an
identity holding only an NT hash serves SMB and **cannot** serve S3, because an
HMAC cannot be computed from an MD4. And a directory that can only answer *yes*
to a password serves WebDAV and answers neither.

That is what `directory` carries what *proves* somebody for, instead of a
password field. The same shape decides who a realm can issue to: only a source
holding the password can back a KDC, because pre-authentication is decrypted and
a verifier answers a yes rather than a key.

`fileshare` links three of these — `directory`, `krb5` and `oidc`, the last for
the bearer tokens its HTTP surface accepts instead of a directory.
[`go-filesystems/nfs`](https://github.com/go-filesystems/nfs) links `krb5`
directly too, one layer below. NFSv3's `AUTH_UNIX` is a claim the client makes
about itself; a ticket is not.

Everything here is pure Go with `CGO_ENABLED=0`. Nothing that reaches a device
is platform-specific: a `Transport` moves 64-byte reports to and from an
authenticator, and where those come from is somebody else's problem. The
transports are [`go-macos/fido`](https://github.com/go-macos/fido) and
[`go-gnulinux/fido`](https://github.com/go-gnulinux/fido); Windows needs none,
because since 1903 it will not let a normal program open a FIDO device at all
([`go-mswin/webauthn`](https://github.com/go-mswin/webauthn) goes through
`webauthn.dll` instead).

Above those sit three repositories that hand a platform's own authentication to
`mfa` as factors, one per system, each satisfying `mfa.Factor` so a policy
written here never learns which one it got:

| | gives `mfa` |
|---|---|
| [`go-macos/factors`](https://github.com/go-macos/factors) | `TouchID`, `DeviceOwner`, `SecurityKey`, `VerifiedSecurityKey` |
| [`go-gnulinux/factors`](https://github.com/go-gnulinux/factors) | `SecurityKey`, `VerifiedSecurityKey` over hidraw |
| [`go-mswin/factors`](https://github.com/go-mswin/factors) | `WindowsHello`, `SecurityKey`, `VerifiedSecurityKey` — through `webauthn.dll`, since a Transport is not available to it |

Touch ID and Windows Hello are the reason `mfa` counts factors by KIND rather
than by name: neither is FIDO, and a policy asking for two distinct kinds is
satisfied by one of them plus a key, and not by two keys.

## Repos

| | Repo | |
|---|---|---|
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-fido.png" width="36"> | [`fido`](https://github.com/go-authn/fido) | The FIDO client-to-authenticator protocol: CTAPHID, `GetInfo`, `MakeCredential`, `GetAssertion`, `ClientPIN`. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-mfa.png" width="36"> | [`mfa`](https://github.com/go-authn/mfa) | The policy: how many factors, of which kinds, and what to say when some refuse. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-keyfactor.png" width="36"> | [`keyfactor`](https://github.com/go-authn/keyfactor) | The piece between the two: a security key as an `mfa.Factor`, over any transport. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-totp.png" width="36"> | [`totp`](https://github.com/go-authn/totp) | Time-based one-time passwords (RFC 6238), the replay guard the RFC requires, and an `mfa.Factor`. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-directory.png" width="36"> | [`directory`](https://github.com/go-authn/directory) | Who somebody is and what proves them, over a database (`sqldir`), an LDAP server (`ldapdir`) or a file (`hcldir`). |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-oidc.png" width="36"> | [`oidc`](https://github.com/go-authn/oidc) | An OpenID Connect token verified into an identity: discovery, JWKS, and the refusals that matter. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-krb5.png" width="36"> | [`krb5`](https://github.com/go-authn/krb5) | The ACCEPTING half of Kerberos: an AP-REQ verified against a keytab, and the AP-REP `gokrb5` will not build. Signs and seals, so `sec=krb5`, `krb5i` and `krb5p`. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-kdc.png" width="36"> | [`kdc`](https://github.com/go-authn/kdc) | The ISSUING half: AS-REQ and TGS-REQ over UDP and TCP, backed by a `directory`. Judged by MIT's own `kinit`, with no MIT KDC in the loop. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-authnd.png" width="36"> | [`authnd`](https://github.com/go-authn/authnd) | A command, not a library: an LDAP server for people who are somewhere else, with a second factor at the bind — and, with a `kerberos` block, a KDC for the same people. |

## Why it exists

**The factors.** In pure Go, client-side, there was nothing to reuse. The
reference of the field is Yubico's **libfido2**, written in C; its one serious Go
binding wraps it through cgo. The pure-Go candidates are thinner in stars than in
substance — `telesma-app/ctap` is 15,000 lines with 56 test files and an active
week, which an earlier reading of this page dismissed on a star count. So
libfido2 and the CTAP specification are read here as *documentation*, that
project is read too, and the code is owned.

Reading them first caught two faults that testing against a key would not have,
because a short ping comes back the same either way: `CTAPHID_KEEPALIVE` is not
an answer, and the message limit is the framing's rather than the length field's.

**The identity.** That half exists because the same thing had been written
twice. A file server deciding who may mount a share and an authentication server
answering for it had each grown a `users` configuration block, and three
packages had each grown an LDAP server to test against. They live in `directory`
once now — the block in `hcldir`, the fixture in `ldaptest` — and `authnd` and
the file server describe the same directory the same way.

The same reading habit applies on this side. `ldaptest` is `glauth/ldap`
underneath, an independent implementation, and it is read in CI by **OpenLDAP's
own `ldapsearch`**, which knows nothing about any of this: a fixture built from
one reading of a protocol can only ever confirm that reading.

## Links

- 🌐 Landing page — <https://go-authn.github.io>
- 🎨 Brand assets — <https://github.com/go-authn/brand>

---

<p align="center"><sub>No cgo. No frameworks. The standard is 100% coverage: <code>fido</code>, <code>mfa</code> and <code>keyfactor</code> are gated on it, and the rest carry a floor in CI with the shortfall printed on every run.</sub></p>
