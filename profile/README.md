<p align="center"><img src="https://raw.githubusercontent.com/go-authn/brand/main/social/go-authn.png" alt="go-authn" width="640"></p>

<h1 align="center">go-authn</h1>
<p align="center">Authentication factors in pure Go — a security key, a fingerprint, and a policy that knows the difference.</p>
<p align="center">
  <img src="https://img.shields.io/badge/Go-1.26-00ADD8?style=flat-square&logo=go&logoColor=white">
  <img src="https://img.shields.io/badge/license-BSD--3--Clause-0A6E96?style=flat-square">
  <img src="https://img.shields.io/badge/cgo-none-0079A8?style=flat-square">
  <a href="https://github.com/go-macos"><img src="https://img.shields.io/badge/transport-go--macos-0079A8?style=flat-square"></a>
</p>

---

## What this is

The phrase is multi-**factor**, not multi-check. A passphrase and a recovery
code are both things you know: whoever learned one has usually learned the other
from the same place, and requiring both buys far less than the count suggests.

`go-authn` is the part that has to be right whatever the factors are — how many
answers, of which kinds, and what to say when some of them fail — plus the
protocol needed to make a security key one of those answers.

Everything here is pure Go with `CGO_ENABLED=0`, and nothing is
platform-specific: a `Transport` moves 64-byte reports to and from an
authenticator, and where those come from is somebody else's problem. The
transports are [`go-macos/fido`](https://github.com/go-macos/fido) and
[`go-gnulinux/fido`](https://github.com/go-gnulinux/fido); Windows needs none,
because since 1903 it will not let a normal program open a FIDO device at all
([`go-mswin/webauthn`](https://github.com/go-mswin/webauthn) goes through
`webauthn.dll` instead).

## Repos

| | Repo | |
|---|---|---|
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-fido.png" width="36"> | [`fido`](https://github.com/go-authn/fido) | The FIDO client-to-authenticator protocol: CTAPHID, `GetInfo`, `MakeCredential`, `GetAssertion`. |
| <img src="https://raw.githubusercontent.com/go-authn/brand/main/avatar/go-authn-mfa.png" width="36"> | [`mfa`](https://github.com/go-authn/mfa) | The policy: how many factors, of which kinds, and what to say when some refuse. |
| | [`keyfactor`](https://github.com/go-authn/keyfactor) | The piece between the two: a security key as an `mfa.Factor`, over any transport. |

## Why it exists

In pure Go, client-side, there was nothing to reuse. The reference of the field
is Yubico's **libfido2**, written in C; its one serious Go binding wraps it
through cgo. The pure-Go candidates are thinner in stars than in substance —
`telesma-app/ctap` is 15,000 lines with 56 test files and an active week, which
an earlier reading of this page dismissed on a star count. So libfido2 and the
CTAP specification are read here as *documentation*, that project is read too,
and the code is owned.

Reading them first caught two faults that testing against a key would not have,
because a short ping comes back the same either way: `CTAPHID_KEEPALIVE` is not
an answer, and the message limit is the framing's rather than the length
field's.

## Links

- 🎨 Brand assets — <https://github.com/go-authn/brand>

---

<p align="center"><sub>No cgo. No frameworks. Covered to 100%.</sub></p>
