# Maintaining and publishing this fork

The `release` branch stays at upstream. Work is split into two branches:

- `codex/gemini-history-cache`: the Gemini cache feature submitted upstream in [Luker PR #33](https://github.com/funnycups/Luker/pull/33).
- `self`: the fork's integration and image branch, containing the feature plus Docker publishing support. This is the fixed source for `ghcr.io/liushuangls/luker:latest`.

The image branch adjusts the publishing gate and adds `latest` to fork builds in the existing workflow. It retains upstream's Dockerfile, native amd64/arm64 builders, manifest assembly and other image tags. Fork builds run only on `self`, on push or manual dispatch; other branches cannot publish through this workflow. New runs cancel older runs on the same branch to prevent an older build from replacing a newer `latest`. Builds use the current repository's GHCR namespace and its `GITHUB_TOKEN`; no personal registry token is needed. Upstream's `latest` policy remains limited to stable tag refs.

## Publish an image

Enable Actions in the fork's **Actions** tab if GitHub displays the first-run fork notice. Push updates to `self` to publish automatically. To rebuild manually, open **Create Docker Image (Release and Staging)**, click **Run workflow**, and select `self`.

The equivalent CLI command is:

```sh
gh workflow run docker-publish.yml --repo liushuangls/Luker --ref self
gh run list --repo liushuangls/Luker --workflow docker-publish.yml --branch self
```

A successful fork build from `self` publishes `ghcr.io/liushuangls/luker:latest`, `:dev`, and a `:sha-<commit>` tag containing both `linux/amd64` and `linux/arm64`. Manual runs also publish `:manual`. Use `latest` for deployment:

```yaml
image: ghcr.io/liushuangls/luker:latest
```

Pull the image and recreate the service to apply a later build; publishing a new `latest` does not change a running container. Keep the previous image digest or SHA tag for rollback. The workflow metadata shows the exact SHA tag. GitHub may initially create the GHCR package as private: authenticate the deployment host to GHCR or change package visibility deliberately if public anonymous pulls are wanted.

Publishing does not update an existing server. When choosing to deploy, change only the compose service's image reference, preserving its config/data/extensions mounts. The feature is off by default; enable **Cache stable chat history** in the OpenRouter connection settings. See [Gemini caching](../basics/connections.md#gemini-history-caching-on-openrouter).

## Bring in upstream updates

Keep the upstream PR branch focused. Integrate upstream changes into the image branch, review conflicts and rerun tests before publishing:

```sh
git fetch upstream
git switch self
git merge upstream/release
git push origin self
```

If the upstream remote is not configured, add `https://github.com/funnycups/Luker.git` as `upstream` first. After the cache feature is merged upstream, reconcile any equivalent changes and keep only the small publishing customization. No force-push of `release` is needed.
