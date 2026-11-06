# Robotics Style Guide

## Contents

| Path | Purpose |
| --- | --- |
| [`STYLE_GUIDE.md`](STYLE_GUIDE.md) | The rules. |
| [`CHANGELOG.md`](CHANGELOG.md) | What changed in each version, and why. |
| `configs/.clang-format` | Automatic formatting (run on save). |
| `configs/.clang-tidy` | Naming and basic code-quality checks. |
| `configs/.clangd` | Quiets the unused-include warning for PROS umbrella headers. |


## Adopting it in a robot repo

The tools can only read config files that sit in the robot repo's root, so the guide is linked and
the configs are copied.

1. **Copy the three files** from `configs/` into the robot repo's root. Each starts with a comment
   such as `# robotics-style-guide v1.0.0` so you can tell which version the repo is on.
2. **Link the guide** from the robot repo's README, pinned to the tag you copied:
   `https://github.com/C1NCY/style-guide/blob/v1.0.0/STYLE_GUIDE.md`
3. **Turn on format on save in VS Code** (`"editor.formatOnSave": true`) and restart clangd
   (**clangd: Restart language server**).
4. Recommended: add a .gitattributes file to the robot repo containing `* text=auto eol=lf` so Windows and Linux checkouts agree on line endings.
5. **Add a "Known deviations" section** to the robot README (rule, reason, date), even if it only
   says "none". See section 9 of the guide.

Don't commit `compile_commands.json`; it contains machine-specific paths. Add it and clangd's
`.cache/` folder to the robot repo's `.gitignore`.

## Upgrading a robot repo

1. Read the [changelog](CHANGELOG.md) entries between your version and the new one.
2. Copy the new `configs/` files over the old ones and update the version link in the README.
3. Restart clangd, let it reformat, fix any new warnings, and commit separately from other work.

Stay on your current version during competition season unless a change is worth the churn.

## Changing the guide

- Propose changes with a pull request. Say what rule changes and why.
- Update `STYLE_GUIDE.md` and the matching file in `configs/` in the same pull request, so they
  never drift apart.
- Add a line to the changelog under a new version heading.
- After merging, tag the release (`git tag v1.1.0 && git push --tags`).

## Versioning

| Bump | When | Example |
| --- | --- | --- |
| Major (2.0.0) | Adopting it reformats or renames lots of existing code | New naming style, different line length |
| Minor (1.1.0) | New rules, sections or checks that don't reformat existing code | A new PROS rule |
| Patch (1.0.1) | Wording fixes and clarifications; no change to behavior of the configs | Fix a typo, add an example |

## Troubleshooting

- **No tidy warnings at all:** `compile_commands.json` is missing, or `.clang-tidy` isn't in the
  robot repo's root. Run `pros build-compile-commands`, then restart clangd.
- **`clang-format` or `clang-tidy` not found in the terminal:** restart the terminal after installing,
  and check that the install location is on your PATH (or use the `pip` install).
- **Hover shows `@brief` / `@param` as plain text:** Doxygen formatting needs a recent clangd (support
  was added in 2025) and the `Documentation: CommentFormat: Doxygen` setting from `.clangd`. Check
  your clangd version in the VS Code Output panel (choose "clangd"), then run **clangd: Check for
  language server update**. If you installed clangd yourself, point `"clangd.path"` at the new one.
  On older versions the comment is still readable as plain text.
- **Standard headers like `<cstdint>` not found:** add
  `"clangd.arguments": ["--query-driver=**/arm-none-eabi-*"]` to `.vscode/settings.json`.
