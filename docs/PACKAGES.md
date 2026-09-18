# Packages

Packages add reusable DSL functions, examples, and optional library systems to NodeForge. Packages are not installed automatically. Each package must be installed explicitly from a directory or ZIP archive through **Library → Packages**.

A package directory contains `nodeforge_package.json` and only the content directories it uses:

```text
example.library/
├── nodeforge_package.json
├── functions/
├── examples/
└── systems/
```

## Manifest

```json
{
  "schema_version": 1,
  "id": "vendor.demo",
  "name": "Vendor Demo",
  "version": "1.0.0",
  "author": "Vendor",
  "description": "Demo NodeForge package.",
  "nodeforge_min_version": "0.49.47",
  "nodeforge_max_version": null,
  "contents": {
    "functions": "functions",
    "examples": "examples",
    "systems": "systems"
  },
  "permissions": {
    "python": false
  }
}
```

| Field | Requirement |
| --- | --- |
| `schema_version` | Integer `1`. |
| `id` | Unique lowercase dotted identifier such as `vendor.demo`. |
| `name` | Display name. |
| `version` | Dotted numeric version string. |
| `author` | Package author. |
| `description` | Package description. |
| `nodeforge_min_version` | Minimum compatible NodeForge version as a dotted numeric string. |
| `nodeforge_max_version` | Maximum compatible version, or `null`. |
| `contents` | Package-relative `functions`, `examples`, and `systems` directories that exist. |
| `permissions.python` | Whether executable Python is allowed for declared package content. |

Content paths must be relative and cannot contain empty, `.` or `..` segments.

## Functions

The declared `functions` directory contains reusable `.nf` scripts. Import them with `from functions import ...` after installation. Each public filename becomes an importable name. Public callable names must be unique across active packages.

See [Imports](SYNTAX.md#imports).

## Examples

The declared `examples` directory contains complete example entries shown in the **Examples** catalog. A simple entry is a top-level `.nf` or `.nodeforge` file. An entry that needs package-local Python support can use a directory containing `source.nf` and an optional `backend.py`.

A package-local `backend.py` can expose helper callables through `BACKEND_BUILTINS`; those helpers are available only while compiling that entry's `source.nf`. Packages that include executable Python must declare `permissions.python: true`, and the user must approve that permission during installation.

## Systems

A declared `systems` directory contains one subdirectory per system. A system directory must contain `system.py`. That module provides a `CONSTRUCTORS` mapping and may provide `load_handlers()`. Because systems execute Python, the package must declare `permissions.python: true`, and the user must approve Python when installing it.

## Installing a package

1. Open **Library → Packages**.
2. Select a package directory or ZIP archive.
3. Review its metadata and requested Python permission.
4. Confirm installation.

A ZIP may contain the package at its root or inside one top-level directory. Absolute paths, path traversal, symbolic links, duplicate paths, and ambiguous package roots are rejected. Installing an already installed package requires replacement; replacement activates the new installation only after validation succeeds.

## Removing a package

1. Open **Library → Packages**.
2. Select the installed package.
3. Remove it.

Removal deletes the package from active package state, so its functions, examples, and constructors are no longer available.
