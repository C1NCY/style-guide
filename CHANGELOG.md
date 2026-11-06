# Changelog

All notable changes to the style guide and its config files are listed here, newest first.
The version follows the rules in the [README](README.md#versioning): a major version means
adopting it will reformat or rename existing code.

## [1.0.0] - 2026-11-06

First release. Adopted the google C++ standards, with a few modifications. 

### Added

- `STYLE_GUIDE.md`: naming, files/headers/scoping, classes, language features, comments, PROS and robot
  rules, formatting and tooling, review checklist, and a known-deviations process.
- `configs/.clang-format`: Google preset with a 100-column limit, one argument per line when
  wrapping, regrouped includes (related header, C system, C++ standard, libraries, project), and
  LF line endings.
- `configs/.clang-tidy`: naming checks for the table in section 1, braces on multi-line bodies,
  `nullptr`, `override`, `using namespace`, and the `// TODO(name):` format.
- `configs/.clangd`: turns off the unused-include warning (noisy with PROS umbrella headers),
  checks doc comments with `-Wdocumentation`, and formats Doxygen comments in hover.
- Comment rules (section 5): Doxygen `/** @brief ... */` comments on header declarations.
- Naming rules for booleans, reference frames, reserved names, hardware-object globals, and
  `constants.h` contents.
- PROS rules for fixed-rate loops (`delay_until`), heap use, tasks and mutexes, and debug output.

### Decisions worth knowing

- Doc comments use the `/** @brief ... */` block format with `@` tags, on header declarations only.
  Doxygen formatting in hover needs a recent clangd; older versions show the tags as plain text.
- Every program-wide constant is `kPascalCase`, whether it is `const` or `constexpr` and whatever
  its type. Port lists may be `const std::vector<std::int8_t>`.