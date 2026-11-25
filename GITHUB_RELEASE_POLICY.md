# GitHub Release Policy

This repository is the desired state of the k8s-flightsim cluster. Argo CD
watches `main` and applies every change.

## Rules

- Trunk-based development with squash merges. `main` is the only
  long-lived branch and changes only through pull requests. Direct pushes
  are blocked.
- Every pull request is merged with **Squash and Merge**: one pull request
  becomes one commit on `main`, one deployment step that makes sense on its
  own. Branches are short-lived and named `release/YYYY-MM-DD-topic`.
- A pull request merges only after the `validate` check passes: every
  Kustomize directory renders, and every object matches the Kubernetes
  schema or the schema of its CRD (kubeconform).
- No approving review is required. The repository has one maintainer, and
  GitHub does not let an author approve their own pull request. With a
  team, require one review from CODEOWNERS.
- History is never rewritten. No force pushes, no amended or rebased
  commits on `main`. To undo a change, open a pull request with
  `git revert`. The history is the deployment log and a recovery tool.
- No secrets, not even encrypted ones. Manifests reference Secrets by name.
  The lab repository creates them in the cluster at runtime.
- Every version is pinned: Helm chart versions, image tags and URLs of
  remote manifests.
- Only declarative manifests. Scripts and documentation live in the lab
  repository.

## Pull request text

The pull request title and description become the commit message on `main`.

- Title: one line, sentence case, imperative mood, no prefix such as
  `feat:`, no trailing period, at most 72 characters.
- Description: optional, what changed and why, in plain English.
- Pull requests that break the cluster on purpose for a training scenario
  start with `Scenario` and are undone with a revert pull request.

## Emergencies

If the cluster needs a change while GitHub or the check is unavailable, an
administrator may disable the ruleset, push, and enable it again. Every
such push is noted in the next pull request.
