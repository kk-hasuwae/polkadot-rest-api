# polkadot-rest-api binaries: prebuilt Linux x86-64 downloads

Download prebuilt, reviewed Linux x86-64 binaries of
[paritytech/polkadot-rest-api](https://github.com/paritytech/polkadot-rest-api),
the Rust REST API for Polkadot, Kusama, Asset Hub and other Substrate chains
and the drop-in replacement for substrate-api-sidecar. Upstream publishes
source releases and Docker images, not standalone binaries. This repository
fills that gap: every binary is built from one exact upstream release tag,
attested by GitHub Actions and published as an immutable GitHub release on
the [Releases page](https://github.com/kk-hasuwae/polkadot-rest-api/releases).

This is a publisher, not a source mirror or a fork. Upstream branches, tags,
and workflow files are deliberately not pushed here. Every build must first be
added to [`approved-releases.json`](approved-releases.json) with a reviewed
stable SemVer tag, its GitHub-verified signature evidence (see "Accepted
upstream tag forms" below), immutable builder and runtime image manifests, and
a source/security review record.

## Published binaries

One publisher release per reviewed upstream version, newest first. The asset
name is `polkadot-rest-api-<upstream-tag>-linux-x86_64`, with a matching
`.sha256` and `.provenance.json` next to it.

### upstream v0.3.2

- Release: [publisher-v0.3.2-r1](https://github.com/kk-hasuwae/polkadot-rest-api/releases/tag/publisher-v0.3.2-r1)
- SHA-256: `e71306fa2dd8ab0e3e9bea0e3637d489da53b1d3752994751a168739fca1ba35`
- Size: 35724824 bytes

### upstream v0.3.1

- Release: [publisher-v0.3.1-r1](https://github.com/kk-hasuwae/polkadot-rest-api/releases/tag/publisher-v0.3.1-r1)
- SHA-256: `6fff20f3c3b21b624f028d9bab5f009b9e4fff17c071685f3ae2d85a900fd095`
- Size: 35588192 bytes

### upstream v0.3.0

- Release: [publisher-v0.3.0-r1](https://github.com/kk-hasuwae/polkadot-rest-api/releases/tag/publisher-v0.3.0-r1)
- SHA-256: `c84794723c4189349dceb6067c635b22db7f779cff8f5a39688a6c9b343f8e00`
- Size: 35768552 bytes

### upstream v0.2.1

- Release: [publisher-v0.2.1-r1](https://github.com/kk-hasuwae/polkadot-rest-api/releases/tag/publisher-v0.2.1-r1)
- SHA-256: `e18411bd744e7ae09ae6b42f1136abf31edaed36ee9b35e61d14216c7748cdae`
- Size: 35699016 bytes

### upstream v0.2.0

- Release: [publisher-v0.2.0-r1](https://github.com/kk-hasuwae/polkadot-rest-api/releases/tag/publisher-v0.2.0-r1)
- SHA-256: `d73fd6daf9c291cdd215ad5288d66879f40736d87ccc05e6539373a175722e6f`
- Size: 35541416 bytes

## Download and verify

Publisher tags are intentionally distinct from upstream tags. For upstream
`v0.3.2`, publisher revision 1 is `publisher-v0.3.2-r1`:

```bash
UPSTREAM_TAG=v0.3.2
RELEASE_TAG=publisher-v0.3.2-r1
REPO=kk-hasuwae/polkadot-rest-api
ASSET=polkadot-rest-api-${UPSTREAM_TAG}-linux-x86_64

gh release download "${RELEASE_TAG}" --repo "${REPO}" \
  --pattern "${ASSET}" \
  --pattern "${ASSET}.sha256" \
  --pattern "${ASSET}.provenance.json"
sha256sum -c "${ASSET}.sha256"
gh attestation verify "${ASSET}" \
  --repo "${REPO}" \
  --signer-workflow "${REPO}/.github/workflows/sync-and-release.yml"
```

Without the GitHub CLI, the same assets are plain release downloads:

```bash
BASE=https://github.com/${REPO}/releases/download/${RELEASE_TAG}
curl -fLO "${BASE}/${ASSET}" \
     -fLO "${BASE}/${ASSET}.sha256" \
     -fLO "${BASE}/${ASSET}.provenance.json"
sha256sum -c "${ASSET}.sha256"
```

The exact release asset set is:

- `polkadot-rest-api-<upstream-tag>-linux-x86_64`
- the matching `.sha256`
- the matching `.provenance.json`

The provenance records the upstream tag object and commit, the signature
policy that verified them, the publisher workflow commit, reviewed base-image
digests, source-input hashes, build run, artifact size, and artifact SHA-256.
Also confirm that the publisher release tag points to the recorded publisher
commit. It points to trusted code on `ci`, never to upstream source.

## Mandatory adoption policy

The co-located checksum establishes consistency, not independent trust. Before a
new publisher revision is used in production:

1. Review the exact upstream source commit and Dockerfile, the upstream tag and
   its signature evidence, and the publisher commit that approved it.
2. Review the public build log and provenance. Verify the artifact attestation
   names this repository and `.github/workflows/sync-and-release.yml`; confirm
   its workflow ref/commit is the publisher commit recorded in provenance.
3. Download and hash the binary independently. Record its SHA-256, upstream tag
   object/commit, publisher commit, and Actions run outside this repository.
4. Install this exact artifact in staging and complete the required soak and
   security/functional checks. A same-version, separately built binary does not
   satisfy this requirement.
5. Run the service as a dedicated least-privilege identity, not as a node or
   other application account. Apply systemd hardening such as `NoNewPrivileges`,
   `ProtectSystem=strict`, `ProtectHome=true`, `PrivateTmp=true`, and
   `RestrictSUIDSGID=true`, adjusted only for documented runtime needs.

The binary should still be pinned by its independently adopted SHA-256 in every
deployment. GitHub transport, release notes, provenance, checksum, and
attestation are complementary evidence; none replaces source review and exact
artifact staging.

## Publisher design

The scheduled/manual workflow performs three separated phases:

1. A read-only audit validates each approval against the live upstream tag and
   its GitHub signature status, and audits any existing release's exact assets
   and hashes. New upstream stable tags are listed only as a human review
   queue.
2. A bounded, read-only job builds at most two approved releases serially, with
   a 45-minute timeout. It fetches the pinned commit, substitutes only the two
   reviewed base-image manifests, and rejects symlinks, special files, implausible
   sizes, and non-x86-64 ELF output without executing the binary.
3. A fresh, trusted publication job is the only job with `contents: write`. It
   treats the transferred bundle strictly as data, revalidates it without
   execution, generates GitHub artifact attestations, and creates a draft. The
   exact uploaded files are downloaded and compared before publication.

### Accepted upstream tag forms

Annotated tag: the record holds the tag object SHA and the peeled commit. The
audit requires the tag object to carry a GitHub-verified, valid signature and
to peel to the recorded commit. Provenance `signature_policy` is
`github-verified-annotated-tag`.

Lightweight tag: the record holds the commit SHA in both fields, which is what
marks it as lightweight. The audit requires the ref to still point at that
commit, the commit's own GitHub-verified, valid signature, and the commit to
be reachable from upstream `main`. Provenance `signature_policy` is
`github-verified-commit-lightweight-tag`. Upstream's RELEASE.md creates
lightweight tags, so this is the common case. The annotated form is preferred,
and upstream has been asked to sign tags (paritytech/polkadot-rest-api#418).

Published releases are never clobbered or rebuilt in place. A changed build gets
a new publisher revision. Upstream tag movement/deletion, signature failure,
release drift, partial assets, or a revoked release still being downloadable
fails the audit. All third-party actions are pinned to reviewed full commit SHAs.

The upstream Dockerfile still installs packages from mutable Debian repositories,
so builds are not guaranteed byte-reproducible even though its base images and
Cargo lockfile are pinned/recorded. This is why publisher revisions are immutable
and adoption pins an independently reviewed artifact hash.

See [SECURITY.md](SECURITY.md) for migration, revocation, and incident procedures.

## Branch and operations model

Only the trusted `ci` publisher branch is required. There is no deploy key and no
automated keepalive commit. GitHub may disable schedules in an inactive public
repository; manually dispatch the audit after approving a release and monitor
the schedule explicitly. Scheduled runs audit provenance and upstream movement;
they do not approve or auto-publish arbitrary new tags.
