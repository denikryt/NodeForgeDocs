# Python Extension API v2

Extension API v2 is the supported Python interface for package-defined NodeForge callables. It separates the public source-language contract from frontend semantic processing and Blender-side realization.

Use Python extensions only for behavior that cannot be expressed cleanly as a source-backed `.nf` function. For ordinary reusable DSL code, start with [source-backed package functions](PACKAGES.md#source-backed-functions).

## Owner layout

A Python extension owner is a directory containing `interface.py` and, when physical Blender realization is required, one or more implementation modules.

```text
systems/
└── scale_tools/
    ├── interface.py
    ├── semantic.py      # optional
    └── backend.py
```

`interface.py` is the canonical declaration module. It defines the public callables and type contracts. `semantic.py` is optional frontend logic. `backend.py` is only a conventional name; an implementation reference may point to another owner-local module.

Do not import owner-local modules from `interface.py`. Keep relative imports such as `from .helpers import ...` in `semantic.py`, `backend.py`, or modules imported by those files. `interface.py` may import the Python standard library, the public `NodeForge` API, and other dependencies allowed by the package.

A single owner cannot contain both `source.nf` and `interface.py`.

## Minimal backend-only extension

The smallest useful extension has a declaration and a physical implementation.

```python
# systems/scale_tools/interface.py
from typing import Annotated
from NodeForge import EvaluationMode, Float

EXTENSION_API = 2

EXTENSIONS = {
    'scale_value': '.backend:scale_value',
}


def scale_value(
    value: Annotated[Float, EvaluationMode.RUNTIME_ONLY],
    *,
    factor: Annotated[Float, EvaluationMode.COMPILE_TIME_OR_RUNTIME] = 2.0,
) -> Float:
    ...
```

```python
# systems/scale_tools/backend.py
from NodeForge.extension_api import ExtensionBackendContext, ExtensionBackendValue, NFType


def scale_value(context: ExtensionBackendContext, value, *, factor=2.0):
    node = context.group.nodes.new('ShaderNodeMath')
    node.operation = 'MULTIPLY'
    node.location = context.location

    context.group.links.new(value.socket, node.inputs[0])

    if isinstance(factor, ExtensionBackendValue):
        context.group.links.new(factor.socket, node.inputs[1])
    else:
        node.inputs[1].default_value = float(factor)

    return context.value(node.outputs[0], NFType.FLOAT)
```

If the package manifest has `"import_name": "tools"`, source code calls it as:

```python
from packages import tools

value = input_float('Value', default=1.0)
result = tools.scale_value(value, factor=3.0)
output('Result', result)
```

## `interface.py` protocol

Every v2 owner must define:

```python
EXTENSION_API = 2
EXTENSIONS = {...}
```

`EXTENSIONS` is the complete public callable inventory for that owner. Each key names a function declared directly in `interface.py`.

A physical implementation reference uses this form:

```python
EXTENSIONS = {
    'name': '.module:function',
}
```

The leading dot is required. The reference is owner-relative and must point to a function in a module other than `interface.py` or `semantic.py`.

Use `None` when a callable is semantic-only and has no physical implementation:

```python
EXTENSIONS = {
    'make_part': None,
}
```

A native-only `functions/<name>/interface.py` or `examples/<name>/interface.py` owner must declare exactly one `EXTENSIONS` key, and that key must equal the owner directory name. A `systems/<owner>/interface.py` owner may declare multiple public callables.

Legacy Extension API v1 layouts such as `system.py`, `CONSTRUCTORS`, or native `BACKEND_BUILTINS` are not supported.

## Direct parameter types

Direct NodeForge values use annotation markers exported from `NodeForge`:

```text
Float
Int
Bool
Vector
Geometry
Material
Object
String
Bundle
Rotation
```

For a direct parameter, wrap one marker or a finite union of markers in `typing.Annotated` with exactly one `EvaluationMode`:

```python
from typing import Annotated
from NodeForge import EvaluationMode, Float


def amount(
    value: Annotated[Float, EvaluationMode.RUNTIME_ONLY],
) -> Float:
    ...
```

The marker classes are annotation tokens; they are not values to instantiate at runtime. A union such as `Float | Int` accepts either declared NodeForge type; the selected runtime value keeps its actual `NFType`.

## Evaluation modes

`EvaluationMode` defines which representation the extension accepts for a direct parameter.

| Mode | Meaning |
| --- | --- |
| `EvaluationMode.COMPILE_TIME_ONLY` | The argument must be available during compilation. The extension receives detached compile-time data. |
| `EvaluationMode.RUNTIME_ONLY` | The argument must be a runtime NodeForge value. The backend receives an `ExtensionBackendValue`. |
| `EvaluationMode.COMPILE_TIME_OR_RUNTIME` | A compile-time argument stays detached; otherwise the normal runtime representation is used. |

A `RUNTIME_ONLY` parameter cannot have a default. Compile-time and mixed parameters may use detached `Bool`, signed-32 `Int`, `Float`, `String`, or 3-component `Vector` defaults. Vector defaults may be written as a Python list or tuple in `interface.py`; NodeForge canonicalizes them before use.

Use compile-time mode for configuration that changes the shape or properties of the generated graph, and runtime mode for data that should remain connected as a Geometry Nodes socket or field.

## Result types

A backend-only callable can return one direct NodeForge type:

```python
def length(value: Annotated[Vector, EvaluationMode.RUNTIME_ONLY]) -> Float:
    ...
```

or a fixed tuple of direct result types:

```python
def split(value: Annotated[Float, EvaluationMode.RUNTIME_ONLY]) -> tuple[Float, Int]:
    ...
```

Variadic result tuples such as `tuple[Float, ...]` and union results such as `Float | Int` are not direct executable result contracts.

For package-defined semantic records and semantic lists, see [Semantic values](#semantic-values).

## Backend arguments and results

A physical implementation is called as:

```python
implementation(context, *bound_args, **bound_kwargs)
```

`context` is an `ExtensionBackendContext`. Runtime arguments are `ExtensionBackendValue` instances. Compile-time arguments are detached Python data selected by the parameter's evaluation mode.

The public backend API is:

```python
from NodeForge.extension_api import ExtensionBackendContext, ExtensionBackendValue, NFType
```

### `ExtensionBackendValue`

A runtime argument exposes:

- `value.socket` — the Blender output socket carrying the runtime value;
- `value.typ` — its canonical `NFType`.

Do not construct compiler values or reach into NodeForge compiler internals. Link the provided socket to nodes created in `context.group`.

### `context.group`

The current candidate Geometry Nodes group. Create and link Blender nodes in this group.

### `context.location`

The `(x, y)` placement anchor for the extension call. Use it as the starting location for generated nodes.

### `context.value(socket, typ)`

Wraps and validates a current-group output socket as the declared NodeForge runtime type.

```python
return context.value(node.outputs['Value'], NFType.FLOAT)
```

The socket must be a valid output socket from the current group and must match the declared `NFType`.

A backend may also return an existing compatible `ExtensionBackendValue` when no additional node is needed.

## Overloads

Finite direct-value overloads use `typing.overload`. Alternatives are tried in source order.

```python
from typing import Annotated, overload
from NodeForge import Bool, EvaluationMode, Float, Int

EXTENSION_API = 2
EXTENSIONS = {'select_value': '.backend:select_value'}

@overload
def select_value(
    cond: Annotated[Bool, EvaluationMode.RUNTIME_ONLY],
    a: Annotated[Int, EvaluationMode.RUNTIME_ONLY],
    b: Annotated[Int, EvaluationMode.RUNTIME_ONLY],
) -> Int:
    ...


@overload
def select_value(
    cond: Annotated[Bool, EvaluationMode.RUNTIME_ONLY],
    a: Annotated[Float, EvaluationMode.RUNTIME_ONLY],
    b: Annotated[Float, EvaluationMode.RUNTIME_ONLY],
) -> Float:
    ...


def select_value(*args, **kwargs):
    ...
```

Overloads are for direct executable contracts. Package-defined semantic record/list contracts are not overload alternatives.

## Semantic values

A package can define frontend-only structured values as frozen dataclasses declared directly in `interface.py`. These values are useful when a group of runtime references and compile-time options should travel together without becoming a Geometry Nodes socket type.

```python
# interface.py
from dataclasses import dataclass
from typing import Annotated
from NodeForge import EvaluationMode, Float

EXTENSION_API = 2


@dataclass(frozen=True)
class Part:
    value: Float


EXTENSIONS = {
    'make_part': None,
    'consume_part': '.backend:consume_part',
}


def make_part(
    value: Annotated[Float, EvaluationMode.RUNTIME_ONLY],
) -> Part:
    ...


def consume_part(part: Part) -> Float:
    ...
```

A public record name may appear in a public parameter or result contract. A leading underscore creates an owner-private semantic record, which is useful for internal normalized state passed from `semantic.py` to a backend.

Semantic records use same-owner nominal identity. Define them directly under their own class name in `interface.py`; do not replace them with lookalike classes imported from another module.

### Record field forms

Record fields and private semantic state support this recursive schema:

```text
one NodeForge marker, or a finite union such as Float | Int
exact Python bool | int | float | str
another registered record
list[T]
tuple[T1, T2, ...]
tuple[T, ...]
dict[str, T]
T | None
```

NodeForge copies Python containers into detached compiler-owned state. Package mutation after a semantic call cannot mutate the compiler's stored semantic value.

### `RuntimeRef`

When `semantic.py` receives a direct runtime argument, it sees a compiler-issued `RuntimeRef` rather than a Blender socket.

```python
from NodeForge import RuntimeRef


def make_part(value) -> Part:
    assert isinstance(value, RuntimeRef)
    return Part(value)
```

`RuntimeRef.typ` reports the actual canonical `NFType`. A `RuntimeRef` belongs only to the active semantic invocation; do not cache or fabricate it.

## `semantic.py`

A callable gets frontend semantic behavior when `semantic.py` defines an ordinary same-name function with a return annotation.

### Semantic-only callable

For a semantic-only callable, use `None` in `EXTENSIONS` and return the public semantic result from `semantic.py`.

```python
# semantic.py
from .interface import Part


def make_part(value) -> Part:
    return Part(value)
```

The result can be assigned and consumed later:

```python
from packages import tools

value = input_float('Value')
part = tools.make_part(value)
result = tools.consume_part(part)
output('Result', result)
```

Semantic records and semantic lists persist in source bindings together with the runtime dependencies they contain. They are frontend semantic values, not ordinary Geometry Nodes socket values: they cannot be exposed directly with `output(...)` and are not Repeat Zone carried state.

### Semantic-then-backend callable

A callable may use `semantic.py` to normalize its public arguments into owner-private state before physical realization.

```python
# interface.py
from dataclasses import dataclass
from NodeForge import Float


@dataclass(frozen=True)
class Part:
    value: Float


@dataclass(frozen=True)
class _BuildState:
    part: Part


EXTENSION_API = 2
EXTENSIONS = {'consume_part': '.backend:consume_part'}


def consume_part(part: Part) -> Float:
    ...
```

```python
# semantic.py
from .interface import _BuildState


def consume_part(part) -> _BuildState:
    return _BuildState(part)
```

```python
# backend.py
from NodeForge.extension_api import NFType


def consume_part(context, state):
    runtime_value = state.part.value
    # Runtime leaves inside semantic state are reconstructed as
    # ExtensionBackendValue objects here.
    return context.value(runtime_value.socket, NFType.FLOAT)
```

The backend receives the current-session record classes. Runtime leaves inside the semantic state are reconstructed as `ExtensionBackendValue` objects; static fields remain detached Python data.

## Semantic lists and `*` expansion

A semantic callable may return or consume a declared `list[Record]`. Semantic lists can persist in source variables and can be expanded into another extension call with caller-side `*` syntax when the receiving contract accepts those elements.

Semantic lists are distinct from ordinary mutable compile-time lists. In particular, they do not gain structural-array mutation methods such as `append(...)`.

## Generated Blender resources

If a backend creates persistent Blender IDs, create transaction-owned resources through `ExtensionBackendContext`:

```python
mesh = context.new_generated_mesh(role='surface', name_hint='Terrain')
curve = context.new_generated_curve(role='path', name_hint='Guide')
obj = context.new_generated_object(mesh, role='preview', name_hint='Terrain')
```

Available helpers are:

- `new_generated_mesh(*, role, name_hint='')`;
- `new_generated_curve(*, role, name_hint='')`;
- `new_generated_object(data=None, *, role, name_hint='')`.

Resources created through these helpers participate in the active NodeForge transaction and can be rolled back when compilation fails. Semantic code must not create Blender resources.

## Snapshot and reload behavior

NodeForge captures `interface.py`, `semantic.py`, implementation modules, and owner-local helpers as one extension-owner snapshot. Source bytes determine extension freshness; filesystem timestamp changes alone do not.

`semantic.py` and physical implementation modules are loaded lazily for the phase that needs them. Do not rely on mutable module state being shared between frontend semantic execution and backend realization.

When an installed package is edited and a group is recompiled or reloaded, NodeForge resolves a new environment snapshot. A single compilation uses one consistent snapshot for the root and nested calls.

## Validation rules

Keep these constraints in mind when authoring an extension:

- `EXTENSION_API` must be exactly `2`;
- `EXTENSIONS` must be a non-empty mapping;
- every exported declaration is defined directly in `interface.py`;
- direct parameters use a NodeForge marker or finite marker union plus exactly one `EvaluationMode`;
- `RUNTIME_ONLY` parameters have no defaults;
- `__unique__` is compiler-reserved and cannot be an extension parameter;
- `**kwargs` is not part of the public extension ABI;
- physical implementation references are owner-relative `'.module:function'` strings;
- `interface.py` does not import owner-local child modules;
- direct executable results are one exact NodeForge type or a fixed tuple of exact types;
- native-only `functions/<name>` and `examples/<name>` owners export exactly `<name>`;
- `source.nf` and `interface.py` are never mixed in the same owner.

Package installation validates these declarations before the candidate package becomes active. Prefer controlled validation failures over compensating for malformed declarations in backend code.
