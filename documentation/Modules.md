# Modules

PyCPP is modularized, and contains a set of core modules and optional modules. The core modules are divided into a hierarchy, so a module should only include headers from a module with less dependencies than its own. Optional modules may include other optional modules, as long as the appropriate `ifdef`s are set.

## Module Hierarchy

In order of ascending dependencies:

- preprocessor
- system
- stl
