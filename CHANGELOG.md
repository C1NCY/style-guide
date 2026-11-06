# Changelog

All notable changes to the style guide and its config files are listed here, newest first.
The version follows the rules in the [README](README.md#versioning): a major version means
adopting it will reformat or rename existing code.

## [1.0.0] - 2026-11-05

First release. Adopted the Google C++ Style Guide, with a few modifications. 

### Added

- `STYLE_GUIDE.md`: naming, files/headers/scoping, classes, language features, PROS and robot
  rules, formatting and tooling, review checklist, and a known-deviations process.
- `configs/.clang-format`: Google preset with a 100-column limit, one argument per line when
  wrapping, regrouped includes (related header, C system, C++ standard, libraries, project).
- `configs/.clang-tidy`: naming checks for the table in section 1, braces on multi-line bodies,
  `nullptr`, `override`, and `using namespace`.
- `configs/.clangd`: ignores the unused-include warning.
- Naming rules for booleans, reference frames, reserved names, hardware-object globals, and
  `constants.h` contents.
- PROS rules for fixed-rate loops (`delay_until`), heap use, tasks and mutexes, and debug output.

### Decisions worth knowing

- Every program-wide constant is `kPascalCase`, whether it is `const` or `constexpr` and whatever
  its type. Port lists may be `const std::vector<std::int8_t>`.