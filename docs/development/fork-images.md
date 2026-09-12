# Maintaining and publishing this fork

The `release` branch stays at upstream. Work is split into two branches:

- `codex/gemini-history-cache`: the Gemini cache feature submitted upstream in [Luker PR #33](https://github.com/funnycups/Luker/pull/33).
- `codex/gemini-cache-images`: the feature plus this fork's manual Docker publishing support. Build images from this branch while the upstream PR is under review.

The image branch changes only the publishing gate in the existing workflow. It retains upstream's Dockerfile, native amd64/arm64 builders, manifest assembly and image tags. Ordinary pushes in a fork do not publish an image. Manual runs use the current repository's GHCR namespace and its `GITHUB_TOKEN`; no personal registry token is needed.

## Publish an image

Enable Actions in the fork's **Actions** tab if GitHub displays the first-run fork notice. Open **Create Docker Image (Release and Staging)**, click **Run workflow**, and select `codex/gemini-cache-images`.

The equivalent CLI command is:

```sh
gh workflow run docker-publish.yml --repo liushuangls/Luker --ref codex/gemini-cache-images
gh run list --repo liushuangls/Luker --workflow docker-publish.yml --branch codex/gemini-cache-images
```

A successful run publishes `ghcr.io/liushuangls/luker:manual`, `:dev`, and a `:sha-<commit>` tag containing both `linux/amd64` and `linux/arm64`. The workflow metadata shows the exact SHA tag. Use the SHA tag or image digest for deployment and rollback; `manual` and `dev` move with later builds. GitHub may initially create the GHCR package as private: authenticate the deployment host to GHCR or change package visibility deliberately if public anonymous pulls are wanted.

Publishing does not update an existing server. When choosing to deploy, change only the compose service's image reference, preserving its config/data/extensions mounts. The feature is off by default; enable **Cache stable chat history** in the OpenRouter connection settings. See [Gemini caching](../basics/connections.md#gemini-history-caching-on-openrouter).

## Bring in upstream updates

Keep the upstream PR branch focused. Integrate upstream changes into the image branch, review conflicts and rerun tests before publishing:

```sh
git fetch upstream
git switch codex/gemini-cache-images
git merge upstream/release
git push origin codex/gemini-cache-images
```

If the upstream remote is not configured, add `https://github.com/funnycups/Luker.git` as `upstream` first. After the cache feature is merged upstream, reconcile any equivalent changes and keep only the small publishing customization. No force-push of `release` is needed.
