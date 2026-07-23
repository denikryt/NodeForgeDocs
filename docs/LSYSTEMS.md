# LSystem Reference

These constructors are provided by the `nodeforge.lsystem` package. Install the package explicitly and approve its Python permission before using them.

The LSystem library generates geometry from rewrite rules. An `ls_system(...)` result is normal `Geometry`: you can `transform(...)`, `join(...)`, assign materials, and connect it to `output(...)`.

For a practical walkthrough, see [Writing L-System Scripts](LSYSTEMS_SCRIPTING.md).

## Constructors

### `ls_system(part, ...)`

Builds the L-system and returns `Geometry`.

A system requires:

| Part | Required | Notes |
| --- | --- | --- |
| `ls_axiom(...)` | yes | Initial symbol stream. |
| `ls_iterations(...)` | yes | Compile-time rewrite count. |
| `ls_angle(...)` | yes | Turn angle in degrees. Can be runtime input. |
| `ls_step(...)` | yes | Forward distance. Can be runtime input. |
| `ls_rule(...)` | optional, many | Rewrite rules. |
| `ls_param(...)` | optional, many | Named numeric values used by parameterized modules. |
| `ls_marker(...)` | optional, many | Declared marker modules that emit point-domain placement data. |

Duplicate singleton parts and duplicate rule predecessors raise `CompileError`.

```python
geo = ls_system(
    ls_axiom("F"),
    ls_rule("F", "F[+F]F[-F]F"),
    ls_iterations(2),
    ls_angle(25),
    ls_step(0.1),
)
output("Geometry", geo)
```

### `ls_axiom(value)`

Defines the starting symbols before any rewrite iteration runs.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time `String` | Initial L-system symbols. |

### `ls_rule(symbol, replacement)`

Defines one rewrite rule.

| Parameter | Type | Description |
| --- | --- | --- |
| `symbol` | compile-time `String` | Exactly one allowed command or grammar symbol. |
| `replacement` | compile-time `String` | Symbols emitted when `symbol` is rewritten. |

Rules rewrite one symbol at a time. Symbols without a rule pass through unchanged.

```python
ls_rule("X", "F+X")
```

Rule strings can be composed from named compile-time string fragments with f-strings. Every interpolated value must be a compile-time string.

```python
left = "[-F[+F]-F]"
right = "[+F+F+F]"
rule = f"F{left}FFF{right}FFF"

geo = ls_system(
    ls_axiom("F"),
    ls_rule("F", rule),
    ls_iterations(3),
    ls_angle(60),
    ls_step(0.1),
)
```


### `ls_iterations(value)`

Defines how many rewrite passes run before geometry is generated.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time `Integer` | Non-negative expansion iteration count. |

Changing this value changes the generated command data and requires recompilation.

### `ls_angle(value)`

Defines the turtle turn angle in degrees.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time number or runtime numeric `Value` | Angle used by `+` and `-`. |

Runtime values commonly come from `input_float(...)`.

### `ls_step(value)`

Defines the turtle forward distance.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time number or runtime numeric `Value` | Distance used by `F` and `f`. |

Runtime values commonly come from `input_float(...)`.

### `ls_param(name, value)`

Declares a named numeric value that can be used inside parameterized L-system modules such as `F(length)` or `+(angle)`. `name` is a compile-time identifier. `value` is a compile-time number or runtime numeric `Value`.

### `ls_marker(name, *parameter_names)`

Declares a multi-character marker module. Marker names use identifier syntax and cannot be one of the built-in turtle commands. Optional `parameter_names` define the point attribute names written by marker arguments.

```python
leaf_size = input_float("Leaf Size", default=0.8)
plant = ls_system(
    ls_axiom("FLeaf(leaf_size)"),
    ls_iterations(0),
    ls_angle(25),
    ls_step(0.1),
    ls_param("leaf_size", leaf_size),
    ls_marker("Leaf", "size"),
)
```

### `ls_points(geometry, marker="Name")`

Extracts marker points for the requested marker name from ordinary geometry attributes. Filtering uses four numeric identity attributes derived from the marker name, so extraction works after assignment, `transform(...)`, and `join(...)` when attributes are preserved by Blender geometry evaluation. The optional human-readable marker name attribute is not part of the filtering contract.

## Turtle commands

| Symbol | Meaning |
| --- | --- |
| `F` | Move forward by `ls_step(...)` and draw a segment. |
| `F(length)` | Move forward by the literal or `ls_param` value `length` and draw a segment. |
| `f` | Move forward by `ls_step(...)` without drawing. |
| `f(length)` | Move forward by the literal or `ls_param` value `length` without drawing. |
| `+` | Turn left by `ls_angle(...)`. |
| `+(angle)` | Turn left by the literal or `ls_param` value `angle`. |
| `-` | Turn right by `ls_angle(...)`. |
| `-(angle)` | Turn right by the literal or `ls_param` value `angle`. |
| `[` | Save current turtle state and begin a branch. |
| `]` | Restore saved turtle state and end a branch. |

ASCII letters, digits, and `_` can be grammar symbols. They participate in rewriting. If they remain in the final expanded stream and are not turtle commands or declared marker modules, they are ignored by drawing. Declared multi-character marker modules such as `Leaf(size)` emit marker points at the current turtle position and do not move or rotate the turtle.

Unsupported punctuation, whitespace, Unicode symbols, unmatched `]`, and unclosed `[` raise `CompileError`.

## Runtime values

`ls_angle(...)` and `ls_step(...)` may use runtime inputs:

```python
angle = input_float("Angle", default=25.0)
step = input_float("Step", default=0.1)

geo = ls_system(
    ls_axiom("F"),
    ls_rule("F", "F[+F]F[-F]F"),
    ls_iterations(2),
    ls_angle(angle),
    ls_step(step),
)
```

`ls_angle(...)` and `ls_step(...)` may receive runtime `Float` values. The axiom, rules, iteration count, parameter names, marker declarations, and branch structure are compile-time declarations and require recompilation when changed.

## Limits

L-systems expand their symbol stream before geometry is generated. Large iteration counts can produce very large systems.

| Limit | Value |
| --- | --- |
| Expanded symbols | `200000` |
| Runtime branch depth | `32` |

A system must produce at least one drawn `F` segment or marker point. Exceeding a limit raises `CompileError`.
