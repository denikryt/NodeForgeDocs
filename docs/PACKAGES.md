# Packages

Packages add reusable functions, examples, and library systems to NodeForge. A package is a directory containing `nodeforge_package.json` and one or more content directories.

```text
example.library/
├── nodeforge_package.json
├── functions/
├── examples/
└── systems/
```

Only include directories used by the package.

## Manifest

The manifest defines the package metadata and content directories:

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

| Field | Description |
| --- | --- |
| `id` | Unique package identifier. |
| `name` | Display name shown in NodeForge. |
| `version` | Package version. |
| `nodeforge_min_version` | Oldest supported NodeForge version. |
| `nodeforge_max_version` | Newest supported NodeForge version, or `null`. |
| `contents` | Package-relative content directories. |
| `permissions.python` | Allows Python files inside declared content directories. |

## Content directories

### Functions

The `functions` directory contains reusable `.nf` functions imported with `from functions import ...`.

See [Imports](SYNTAX.md#imports) for using functions from installed packages.

### Examples

The `examples` directory contains complete scripts that can be added from the **Examples** catalog.

### Systems

The `systems` directory contains library constructors implemented by a package. Each system is stored in its own directory under the declared content root.

## Installing a package

1. Open the NodeForge **Library** panel.
2. Open **Packages**.
3. Choose the package archive or directory.
4. Review the package information and requested permissions.
5. Confirm the installation.
6. Refresh the affected library catalogs.

Packages containing Python require installation consent.

## Removing a package

1. Open **Library → Packages**.
2. Select the installed package.
3. Remove the package.
4. Refresh the library catalogs.

Duplicate public function, example, or constructor names are reported as package errors.

