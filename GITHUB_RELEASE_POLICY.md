# GitHub Release Policy

This repository is the desired state of the k8s-flightsim cluster. Argo CD
watches `main` and applies every change.

## Rules

- `main` is the only branch. Changes are committed directly to `main`.
  Every commit is a deployment step and must make sense on its own.
- History is never rewritten. No force pushes, no amended or rebased
  commits on `main`. To undo a change, use `git revert`. The history is the
  deployment log and a recovery tool.
- No secrets, not even encrypted ones. Manifests reference Secrets by name.
  The lab repository creates them in the cluster at runtime.
- Every version is pinned: Helm chart versions, image tags and URLs of
  remote manifests.
- Only declarative manifests. Scripts and documentation live in the lab
  repository.

## Commit messages

- Title: one line, sentence case, imperative mood, no prefix such as
  `feat:`, no trailing period, at most 72 characters.
- Body: optional, what changed and why, in plain English.
- Commits that break the cluster on purpose for a training scenario start
  with `Scenario` and are undone with `git revert`.
