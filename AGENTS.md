# Agent Instructions

**Verbosity:** concise
**Chattiness:** low

## Language Policy

**Always** write code, code comments, commit messages and command output in
English. Communicate with the user in their preferred language, but **never**
write code or comments in the user's language. If the user's language
distinguishes between formal and informal pronouns, always use the informal
form.

## Language Usage

Role model: News language.

- Information first.
- Short sentences, strong verbs.
- Be transparent, accountable, reliable and authentic.
- State facts in the indicative mood and uncertainties in the subjunctive mood.
  - *Correct (Fact):* "The node `w2` is Ready again."
  - *Correct (Uncertainty):* "This could cause a split brain if..."
  - *Incorrect:* "I think I have successfully fixed the node." (Do not guess about facts).
- **Avoid all-caps emphasis:** Do not use capitalization to emphasize rules, conditions, or importance (e.g., instead of "ALWAYS", use "**always**"). Use Markdown formatting for emphasis instead.
- **No dashes for clauses:** Do not use em-dashes (—) or en-dashes (–) to connect clauses or thoughts. Use commas, colons, or parentheses instead, or write shorter sentences.

## Admitting Mistakes

Admit mistakes immediately and without sugarcoating.
Do not call a mistake "confusion".

## Project

All files in this repository are in English, because it is published.

The desired state of the k8s-flightsim cluster. Argo CD watches `main` and
applies every change. Scripts, docs and scenarios live in the lab repository,
not here.

- Only declarative manifests: Kustomize, Helm values, Argo CD Applications.
- `applications/` holds one Argo CD Application per component. Order them
  with `argocd.argoproj.io/sync-wave` (platform below 0, apps from 0).
- Render every changed Kustomization with `kubectl kustomize <dir>` before
  committing.
- Manifest comments explain why, not what.

## Understand Before Improving

- **Blast radius:** If a change touches more than 3 files or changes the
  architecture (node layout, network, storage, backup path, repository
  layout), present a brief plan and wait for approval before modifying files.
- **Dependencies:** Do not introduce new tools, Helm charts, operators,
  Ansible collections or Terraform providers without explicit permission.
  Solve problems with the existing stack. Suggest a new dependency only if
  its value is exceptionally high.
- **Existing patterns first:** Extend what exists instead of adding new
  layers. Build only what is requested now (YAGNI), but make it fit the
  existing structure.
- **Pin versions** of everything that is downloaded (images, charts,
  providers, actions, remote manifests). Pick versions that are at least two
  weeks old.

## Git Commits

This repository has **direct commits on `main`**, no pull requests. Every
commit is a deployment step: Argo CD applies it after the push.

- Commit locally under the user's own credentials. **Never** push. The user
  pushes.
- **Never** rewrite history: no amend of pushed commits, no rebase, no force
  push. Undo a change with `git revert`.
- Commit messages describe the change, not the tools used to make it: no
  `Co-Authored-By` trailer, no "Generated with" line, no tool names. The use
  of AI is disclosed openly in the README section "AI Disclosure", which
  applies to every commit. Repeating it in each commit would add no
  information about the change.
- Commits that break the cluster on purpose for a training scenario start
  with `Scenario`.

Before staging, check the diff for secrets and host-specific paths. If you
find one, name what and where, never the value, and stop.

The message:

- **Sentence case.** First letter capitalized, standard English
  capitalization, no trailing period.
- **Imperative mood.** "Add etcd quorum loss scenario", not "Adds" or "Added".
- **No type or topic prefix.** No `feat:`, `fix:`, `refactor:`, and no
  colon-separated prefix of any kind. Plain sentences only.
- **No emojis and no colons.**
- **Plain language.** Clear, specific wording without filler or
  generated-sounding phrases.
- **Short.** One line preferred, at most three, each at most 72 characters.
  Add lines only when the subject cannot carry the change: the subject line,
  a blank line, then one or two short lines of plain text. No bullet points.

Commit what the working tree holds. If it carries several unrelated changes,
say so and propose separate commits rather than inventing one message that
covers everything.

The rules for this repository are in `GITHUB_RELEASE_POLICY.md`. Read it
before the first commit of a session.

## Secrets

Credentials, tokens, keys, passwords, connection strings, kubeconfigs,
personal data: never printed, not in a response, log, commit message, error,
or comment. Refer to them by name or path.

Do not read `.env` files, keyrings, SSH private keys, kubeconfigs, or cloud
config unless that is exactly what was asked. Do not write a secret outside a
designated secret store, never into version control, never into an outward
call such as an API request, web search, or subagent prompt. Check the diff
before staging.

This project keeps **no secrets in git, not even encrypted ones** (no SOPS).
Kubernetes Secrets are created in the cluster at runtime and referenced by
name.

Found one incidentally: say what and where, never the value. Ambiguous case:
treat it as a secret.
