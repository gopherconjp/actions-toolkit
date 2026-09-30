# Repository Instructions

- Use Conventional Commits for all commit messages and PR titles.
- Format: `<type>(<scope>): <subject>` (e.g. `feat: add wait-for-workflow action`).
- Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- `feat` triggers a minor release, `fix` and `perf` trigger a patch release, other types trigger no release.
- Add `!` after the type or a `BREAKING CHANGE:` footer for breaking changes (triggers a major release).
- Squash merge PRs so the PR title becomes the release commit message.
