# Repository Instructions

- Use Conventional Commits for all commit messages and PR titles.
- Format: `<type>(<scope>): <subject>` (e.g. `feat: add wait-for-workflow action`).
- Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- `feat` triggers a minor release, `fix` and `perf` trigger a patch release, other types trigger no release.
- Add a `BREAKING CHANGE:` footer for breaking changes (triggers a major release).
- Do not use the `!` marker (e.g. `feat!:`); the default Angular preset does not recognize it.
- Squash merge PRs so the PR title becomes the release commit message.
