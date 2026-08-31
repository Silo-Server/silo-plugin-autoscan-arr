# Agent instructions

Read [README.md](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md) before working here; they define this repository's ownership, build, and validation rules.

## Writing

Give human-facing prose a final readability pass. Lead with the outcome, use concrete plain language and active voice, and cut filler, stock framing, repetition, and promotional claims. Preserve meaning, evidence, citations, uncertainty, and established terminology. Never rewrite exact quotations, commands, logs, identifiers, API names, or contractual language. Match the audience and use restrained formatting.

## Pull requests

- Never create a pull request unless the developer explicitly asks.
- Use a plain-language Conventional Commit title. In the body, explain the problem before the solution and end with the required AI disclosure, naming the exact model, harness, and tooling used.
- For UI changes, include before-and-after evidence and a video for motion or timing behavior. Upload pull-request evidence to GitHub; never commit PR-only assets such as `.github/pr-assets`.
- Keep one concern per pull request. If the description needs the word "also" for another change, split it.
- When babysitting a pull request, poll for checks and comments newer than the last push. Verify bot findings against the source, fix real issues, and dismiss false positives with a written reason. Stay quiet when nothing is new, and stop when the latest commit is green.
