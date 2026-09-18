# mergecomplete releases

Every build of the [mergecomplete](https://mergecomplete.com) runner is signed with an ed25519 key
the app doesn't hold, and the runner installs a build only when that signature checks against the
key compiled into it. This repository publishes what the signature covers: each build's SHA-256,
the signature itself, and the public key.

So you can check a download against something mergecomplete doesn't serve. You never have to — the
runner installs and updates itself without any of this. It's here for anyone who'd rather check.

## What's here

| | |
|---|---|
| `releases/<version>.json` | One manifest per runner version. Its history is the record: a change to a past release shows in the log |
| `mergecomplete-release.pub.pem` | The key releases are signed with. The copy that decides anything is the one compiled into the runner |

A version names the commit that built it, so the same version is always the same bytes.
`mergecomplete version` prints the one you're running.

## Checking a build

Download it from the app and take its checksum:

```sh
curl -fsSL -o mergecomplete-darwin-arm64 \
  https://app.mergecomplete.com/api/runner/download/darwin/arm64
shasum -a 256 mergecomplete-darwin-arm64
```

Then compare that with the same platform's line here:

```sh
jq -r '.builds["darwin-arm64"].sha256' releases/<version>.json
```

The platforms are `darwin-arm64`, `darwin-amd64`, `linux-arm64` and `linux-amd64`.

This is the check worth running. The binary came from the app and the checksum came from GitHub, so
the two have to agree.

## Checking the signature

`release` in a manifest is the release exactly as the app serves it — the version, the protocol and
every build's SHA-256, on the one line the signature covers.

Verifying ed25519 takes OpenSSL 3. Most Linux distributions ship it; on macOS `/usr/bin/openssl` is
LibreSSL, and `brew install openssl` gives you one that can.

```sh
jq -rj .release releases/<version>.json > release.json
jq -r .signature releases/<version>.json | base64 -d > release.sig
openssl pkeyutl -verify -pubin -inkey mergecomplete-release.pub.pem \
  -rawin -in release.json -sigfile release.sig
```

`Signature Verified Successfully` means this key signed that release. The runner you downloaded
carries the same key, which you can see for yourself:

```sh
grep -aqF "$(jq -r .key releases/<version>.json)" mergecomplete-darwin-arm64 &&
  echo "this build carries that key"
```

That's the whole chain. Whoever can change what the app serves still can't choose what runs on your
machine.

## Where the runner comes from

The install command is on [app.mergecomplete.com](https://app.mergecomplete.com), and
[the docs](https://docs.mergecomplete.com/start/install/) say what it does. The runner reads your
code with git, has Claude Code write the review, and sends the review to the app. It runs on your
machine, on your own Claude plan.
