# Releasing Vyrion (and how in-wallet updates work)

Nodes check `https://github.com/polymathdanny/Vyrion/releases/latest` every 6 hours (and 20 seconds after start). A release is only offered to users if it contains a **signed** `vyrion-update.json`. Everything else on GitHub (commits, tags, unsigned releases) is ignored by nodes, so pushing code never updates anyone by itself.

## What users see

| Release type | When to use | What nodes do |
|---|---|---|
| **Optional** | Bug fixes, UI, performance, anything that does not change which blocks or transactions are valid | Wallet shows "update available" (status bar, Overview banner, Settings → Software update). The user installs when they like. Nothing pauses. |
| **Required** | New consensus rules (e.g. a new transaction kind, a changed limit) that activate at a block height | Wallet shows "required update" with a countdown to the activation height. At that height, nodes that have not updated **pause mining and sending** (and say why) until they update. Coins and wallet are untouched; receiving and viewing still work. |

How the update is applied depends on the build:

- **Windows / Linux release builds** (from the release zip or tar.gz): one click. The node downloads the archive, checks the sha256 in the signed manifest, runs the new `vyriond -version`, backs up `wallet.dat`, `agents.json`, `vyrion.conf`, `node.key` and `relay.json` to `<datadir>/backups/<date>-before-update-from-<version>/`, renames the current programs to `*.old` and moves the new ones in, then restarts itself. The data folder is never deleted or rewritten. `vyriond -rollback` restores the `*.old` programs.
- **Source builds** (`make build`, `go build`): the wallet shows the `git checkout vX.Y.Z && make build` commands instead of an Install button.
- **System services** (binaries in a root-owned folder like `/usr/local/bin`): the wallet shows the exact download, checksum and `sudo install` / `systemctl restart` commands. Under systemd, a node that did self-update exits with code 75 and systemd restarts it.

Settings in `vyrion.conf`: `updatecheck=0` turns automatic checks off (manual "Check now" still works); `autoupdate=1` installs updates without asking (off by default).

## The release signing key

Nodes trust the public keys in `node/internal/update/update.go` (`TrustedKeys`). The matching private key signs every release.

- Keep `vyrion-release.key` **offline** (USB stick, password manager, hardware-encrypted drive) and make a second backup. Never commit it, never put it in CI.
- Anyone with this key can push software to every node; anyone without it cannot, even with full control of the GitHub account.
- To rotate, generate a new key, add its public key to `TrustedKeys` in a release signed with the **old** key, then sign later releases with the new key and eventually remove the old one:

```
cd node && go run ./cmd/vyrion-release keygen -out ~/vyrion-release-2.key
```

## Making a release

```bash
cd node
# 1. Bump the version: VERSION in the Makefile and the default in internal/node/node.go.
#    If consensus rules changed, bump ConsensusVersion in internal/blockchain/params.go.

# 2. Test
go test ./... && go test -race ./internal/...
contrib/e2e-chain.sh && contrib/e2e-agentspace.sh
RELEASE_KEY=~/vyrion-release.key contrib/e2e-update.sh

# 3. Build all platforms (dist/vyrion-X.Y.Z-<os>-<arch>.zip / .tar.gz)
make release VERSION=0.8.1

# 4. Write and sign the manifest
#    Optional update:
go run ./cmd/vyrion-release manifest -dist dist -version 0.8.1 -key ~/vyrion-release.key \
   -notes "Faster sync for new nodes. Wallet fixes."
#    Required (consensus) update: give each network's activation height.
#    Leave enough time for people to update: at 60-second blocks, 10080 blocks ≈ 1 week.
go run ./cmd/vyrion-release manifest -dist dist -version 0.9.0 -key ~/vyrion-release.key \
   -consensus 4 -required test=152000,main=60000 -notes "AgentSpace v2 rules from block …"

go run ./cmd/vyrion-release verify dist/vyrion-update.json   # double-check

# 5. Tag and publish (assets + manifest + signature on the same release)
git tag -a v0.8.1 -m "Vyrion 0.8.1" && git push origin v0.8.1
gh release create v0.8.1 --repo polymathdanny/Vyrion --title "Vyrion 0.8.1" --notes-file notes.md \
   dist/vyrion-0.8.1-*.zip dist/vyrion-0.8.1-*.tar.gz dist/vyrion-update.json dist/vyrion-update.json.sig
```

Or upload the same files through GitHub's web page (Releases → Draft a new release → tag `v0.8.1`). The release must be marked **latest** (not a draft or pre-release), and it must include `vyrion-update.json` and `vyrion-update.json.sig`.

## Rules for required updates

1. The new code must enforce the new rules only from the activation height; before it, it must accept the same blocks as the old version.
2. Publish the release well before the height (at least a week on mainnet), and announce it.
3. Only list networks that get the new rules in `-required` (e.g. testnet first, mainnet in a later release).
4. Check with `vyrion-cli getupdateinfo` on a test node that it shows `required` with the right height.

## Testing a release before publishing

Serve `dist/` locally and point a test node at it:

```bash
cd dist && python3 -m http.server 8000
vyriond -network regtest -updateurl http://127.0.0.1:8000/vyrion-update.json
vyrion-cli -network regtest checkupdate
vyrion-cli -network regtest installupdate
```
