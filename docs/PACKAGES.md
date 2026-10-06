# Creating Packages

A NodeForge package is an installable library that can provide reusable DSL functions, examples, and Python extensions. Packages are installed explicitly through **Library → Packages** and are validated before they become active.

Use a package when reusable content should be distributed as one versioned unit. For scripts used only on one machine or project, the [Local catalog](GET_STARTED.md#save-your-own-script-to-local) is usually simpler.

## Package layout

Every package has a `nodeforge_package.json` manifest at its root and one or more content directories declared by that manifest.

A source-only package can be as small as:

```text
vendor.shapes/
├── nodeforge_package.json
└── functions/
    └── ring.nf
```

A package that uses all supported content roots can look like this:

```text
vendor.toolkit/
├── nodeforge_package.json
├── functions/
│   ├── ring.nf
│   ├── scatter/
│   │   └── source.nf
│   └── native_value/
│       ├── interface.py
│       └── backend.py
├── examples/
│   └── demo_scene.nf
└── systems/
    └── procedural/
        ├── interface.py
        ├── semantic.py
        └── backend.py
```

Only declare content roots that the package actually contains.

## Manifest

`nodeforge_package.json` uses schema version `1`.

```json
{
  "schema_version": 1,
  "id": "vendor.shapes",
  "import_name": "shapes",
  "name": "Vendor Shapes",
  "version": "1.0.0",
  "author": "Vendor",
  "description": "Reusable shape functions for NodeForge.",
  "nodeforge_min_version": "0.65.0",
  "nodeforge_max_version": null,
  "contents": {
    "functions": "functions"
  },
  "permissions": {
    "python": false
  }
}
```

| Field | Requirement |
| --- | --- |
| `schema_version` | Must be integer `1`. |
| `id` | Canonical package identity. Use a dotted lowercase identifier such as `vendor.shapes`. |
| `import_name` | Optional DSL package name. Defaults to the last component of `id`; for `vendor.shapes` the default is `shapes`. |
| `name` | Display name shown to the user. |
| `version` | Dotted numeric package version such as `1.2.0`. |
| `author` | Optional package author text. Defaults to an empty string when omitted. |
| `description` | Optional short description. Defaults to an empty string when omitted. |
| `nodeforge_min_version` | Minimum compatible NodeForge version as a dotted numeric string. |
| `nodeforge_max_version` | Optional maximum compatible NodeForge version. Omit it or use `null` for no upper bound. |
| `contents` | Non-empty object declaring any of `functions`, `examples`, and `systems`. |
| `permissions.python` | Defaults to `false`. Must be `true` when declared package content contains Python or declares `systems`. |

`id` is the stable package identity used for installation and ownership. `import_name` is the spelling used by NodeForge source code. It must be a public Python-style identifier, cannot begin with `_`, and cannot be a Python keyword. Active packages cannot share the same `import_name`.

Content paths are package-relative POSIX paths. They cannot be absolute and cannot contain empty, `.` or `..` path segments.

Version fields are compared numerically by component. For example, `0.65` and `0.65.0` compare as the same version.

## Package namespaces

Installed package callables are imported through the `packages` namespace.

```python
from packages import shapes

geo = shapes.ring(2.0, 0.25)
output('Geometry', geo)
```

Aliases are supported:

```python
from packages import shapes as s

geo = s.ring(2.0, 0.25)
output('Geometry', geo)
```

Use qualified calls such as `shapes.ring(...)` in reusable code. Different packages may export the same member name, and a package member may use the same spelling as a Core DSL callable. Qualification identifies the exact owner.

An unqualified member can also resolve when exactly one imported package provides that name and no source binding owns the spelling:

```python
from packages import shapes

geo = ring(2.0, 0.25)
```

The qualified form is more robust when another package or local variable is added later.

`from packages import *` is not supported. Package names are namespace qualifiers, not runtime values, so this is also invalid:

```python
from packages import shapes
value = shapes  # invalid: a package namespace is not a DSL value
```

The former package-source form `from functions import ...` is not supported. `from examples import ...` and `from local import ...` remain separate catalogs.

## Source-backed functions

The `functions` root contains reusable NodeForge DSL functions. An entry can be either a top-level `.nf` or `.nodeforge` source file:

```text
functions/
└── ring.nf
```

or a directory containing `source.nf`:

```text
functions/
└── ring/
    └── source.nf
```

The entry name becomes the public package member. For example, `functions/ring.nf` is called as `shapes.ring(...)` after `from packages import shapes`.

A source-backed function uses the same interface rules as any reusable NodeForge script. Explicit `input_*()` declarations become parameters and `output(...)` declarations become results.

```python
# functions/scale_cube.nf
size = input_float('Size', default=1.0)
factor = input_float('Factor', default=2.0)
geo = cube(size=size * factor)
output('Geometry', geo)
```

```python
from packages import shapes

geo = shapes.scale_cube(1.5, factor=3.0)
output('Geometry', geo)
```

Input labels are display text, not parameter identity. Each `input_*()` call declares a distinct input even when two inputs have the same label. Keyword matching uses the normalized label described in [Writing Functions](WRITING_FUNCTIONS.md#write-the-function); if two labels normalize to the same keyword, use positional arguments for those inputs.

Editable source-backed calls support the normal reusable-function behavior described in [Reusable Functions](SYNTAX.md#reusable-functions), including `__unique__=True` where supported.

## Examples

The `examples` root contains complete scripts shown in **Library → Examples**. Examples remain in the explicit `examples` catalog rather than the package callable namespace.

A source example can be a top-level `.nf` or `.nodeforge` file:

```text
examples/
└── demo_scene.nf
```

or a directory with `source.nf`:

```text
examples/
└── demo_scene/
    └── source.nf
```

It can be added from the Library UI or imported explicitly:

```python
from examples import demo_scene
```

Python-backed examples use Extension API v2: the owner directory contains `interface.py` and an implementation module such as `backend.py`. A single owner cannot contain both `source.nf` and `interface.py`; source/Python hybrid owners are rejected.

## Python extensions

Python extensions use the declarative Extension API v2. They are appropriate when a package needs behavior that cannot be expressed with the Core DSL alone, such as a custom Blender-node realization or package-defined semantic values.

There are two common layouts.

A native-only entry under `functions` exports exactly one callable whose name matches the entry directory:

```text
functions/
└── native_value/
    ├── interface.py
    └── backend.py
```

A `systems` owner may declare multiple public callables:

```text
systems/
└── procedural/
    ├── interface.py
    ├── semantic.py      # optional
    └── backend.py
```

All public callables from both layouts are exported through the owning package namespace. They therefore use the same source syntax as source-backed functions:

```python
from packages import toolkit

value = toolkit.native_value(...)
geo = toolkit.build_procedural(...)
```

Any package containing Python must set:

```json
{
  "permissions": {
    "python": true
  }
}
```

The user must approve that permission when installing the package.

For the complete declaration, backend, semantic-value, overload, and generated-resource contracts, see [Python Extension API v2](EXTENSION_API.md).

## Public-name rules

Top-level files and owner directories under `functions` and `examples` use their filename or directory name as the public entry name. System directory names are also public identifiers. Public names must be valid Python-style identifiers, cannot begin with `_`, and cannot be Python keywords.

Within one package, a source/native function and a system-exported callable cannot publish the same member name. Different packages may publish the same member because package qualification keeps ownership explicit.

Files and directories beginning with `__` are not public entries.

## Packaging for installation

The Blender UI installs package ZIP archives. Create an archive with exactly one `nodeforge_package.json`, either directly at the archive root or inside one top-level directory. For example:

```text
shapes.zip
└── vendor.shapes/
    ├── nodeforge_package.json
    └── functions/
        └── ring.nf
```

Archives with ambiguous roots, path traversal, symbolic links, duplicate paths, Python cache artifacts, or invalid manifest content are rejected.

## Installing and replacing a package

1. Open **Library → Packages**.
2. If the package requires Python, enable **Allow executable Python**. Enable it only for a package whose Python code you trust.
3. Click **Import** and select the package ZIP.
4. When replacing an installed package with the same `id`, enable **Replace existing package** in the import options.
5. Confirm the import.

NodeForge copies the validated package into its managed package storage; the installed package is not a live link to the ZIP or authoring directory. To publish source changes to an existing installation, create a new ZIP and import it with **Replace existing package** enabled.

NodeForge validates the replacement candidate before publishing it as active state, so a failed replacement leaves the previous installation active. A new root compilation resolves one package environment snapshot and uses that snapshot consistently for nested calls.

## Removing a package

1. Open **Library → Packages**.
2. Select the installed package.
3. Click **Uninstall**.

Removal deletes the package from active package state. Existing generated Geometry Nodes groups remain Blender data, but new compilation can no longer resolve callables from the removed package.

## Package compatibility checklist

Before distributing a package, verify that:

- `nodeforge_package.json` declares the exact content roots that exist;
- `nodeforge_min_version` matches the newest NodeForge feature the package relies on;
- `import_name` is stable and does not conflict with another package you expect users to install;
- source-backed functions compile from a clean Blender file;
- every Python owner uses `interface.py` with `EXTENSION_API = 2`;
- no owner mixes `source.nf` with `interface.py`;
- `permissions.python` is `true` whenever any declared content contains `.py` files;
- examples are usable from **Library → Examples**;
- package calls use qualified `package.member(...)` syntax in published examples.
