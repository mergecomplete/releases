# mergecomplete releases

For security, every build of the [mergecomplete](https://mergecomplete.com) **runner** is signed with an `ed25519` key
the app doesn't hold. The runner checks that signature before it installs anything, so an update is
always a build we signed.

The public key and each build's SHA-256 are published here, so you can verify your first download, or
any update after it, for yourself.

## What's here

| File | What it is |
|---|---|
| `releases/<version>.json` | One manifest per runner version: each build's SHA-256, the signature, and the key it was signed with |
| `mergecomplete-release.pub.pem` | That same public key, as PEM, which is the form `openssl` reads |

A version names the commit that built it, so the same version is always the same bytes.
`mergecomplete version` prints the one you're running.

## Checking a build

Download it from the app and take its checksum:

```sh
curl -fsSL -o mergecomplete-darwin-arm64 \
  https://app.mergecomplete.com/api/runner/download/darwin/arm64
shasum -a 256 mergecomplete-darwin-arm64
```

The platforms are `darwin-arm64`, `darwin-amd64`, `linux-arm64` and `linux-amd64`.

Then compare that checksum with the same platform's line here:

```sh
jq -r '.builds["darwin-arm64"].sha256' releases/<version>.json
```

The two should match.

## Checking the signature

Verifying `ed25519` takes OpenSSL 3. Most Linux distributions ship it. macOS doesn't: `/usr/bin/openssl`
is LibreSSL, so install OpenSSL 3 with `brew install openssl`.

```sh
jq -rj .release releases/<version>.json > release.json
jq -r .signature releases/<version>.json | base64 -d > release.sig
openssl pkeyutl -verify -pubin -inkey mergecomplete-release.pub.pem \
  -rawin -in release.json -sigfile release.sig
```

`Signature Verified Successfully` means we signed that release. The signature covers everything in it:
the version, the protocol, and every build's SHA-256.

The runner checks updates against that same key, compiled into its binary. You can see it there:

```sh
grep -aqF "$(jq -r .key releases/<version>.json)" mergecomplete-darwin-arm64 &&
  echo "this build carries that key"
```

## Where the runner comes from

The install command is on [app.mergecomplete.com](https://app.mergecomplete.com), and
[the docs](https://docs.mergecomplete.com/start/install/) say what it does. The runner reads your
code with git, has Claude Code write the review, and sends the review to the app. It runs on your
machine, on your own Claude plan.
