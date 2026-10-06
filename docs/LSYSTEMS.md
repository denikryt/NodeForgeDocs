# L-System Reference

The `nodeforge.lsystem` package adds L-system constructors to NodeForge. Install the package through **Library → Packages**, enable **Allow executable Python** before importing it, and import its namespace before using these constructors.

```python
from packages import lsystem as ls
```

The reference examples below use the alias `ls`. An L-system starts from an axiom, applies rewrite rules for a number of iterations, then interprets the resulting modules with a 3D turtle. `ls.system(...)` returns normal `Geometry`, so the result can be transformed, joined, assigned materials, and connected to `output(...)` like other NodeForge geometry.

For a step-by-step guide, see [Writing L-System Scripts](LSYSTEMS_SCRIPTING.md).

## Constructors

### `ls.system(part, ...)`

Builds an L-system from constructor parts and returns `Geometry`.

| Part | Required | Multiplicity | Description |
| --- | --- | --- | --- |
| `ls.axiom(...)` | yes | one | Initial module stream. |
| `ls.iterations(...)` | yes | one | Static or runtime rewrite count. |
| `ls.angle(...)` | yes | one | Default turtle rotation angle in degrees. |
| `ls.step(...)` | yes | one | Default forward distance. |
| `ls.rule(...)` | no | many | One-symbol rewrite rules. |
| `ls.param(...)` | no | many | Named numeric values used by parameterized modules. |
| `ls.marker(...)` | no | many | Marker modules that emit points with orientation and parameter data. |

All parts are positional. Their order inside `ls.system(...)` does not affect declaration resolution: rules and the axiom can reference parameters and markers declared later in the same call.

Duplicate singleton parts, duplicate rule predecessors, duplicate parameter names, and duplicate marker names raise `CompileError`.

```python
from packages import lsystem as ls

iterations = input_int("Iterations", default=4)
angle = input_float("Angle", default=25.0)
step = input_float("Step", default=0.08)

plant = ls.system(
    ls.axiom("X"),
    ls.rule("X", "F[+X][-X]FX"),
    ls.rule("F", "FF"),
    ls.iterations(iterations),
    ls.angle(angle),
    ls.step(step),
)

output("Geometry", plant)
```

### `ls.axiom(value)`

Defines the module stream used before the first rewrite iteration.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time `String` | Initial L-system stream. |

The string may contain turtle commands, one-character grammar symbols, declared marker modules, and parameterized modules.

```python
geo = ls.system(
    ls.axiom("F+F+F+F"),
    ls.iterations(0),
    ls.angle(90),
    ls.step(1),
)

output("Geometry", geo)
```

### `ls.rule(symbol, replacement)`

Defines one parallel rewrite rule.

| Parameter | Type | Description |
| --- | --- | --- |
| `symbol` | compile-time `String` | Exactly one turtle command or one ASCII letter, digit, or `_`. |
| `replacement` | compile-time `String` | Module stream that replaces `symbol`. May be empty. |

During one iteration, every module is rewritten from the same input generation. Symbols without a rule pass through unchanged. An empty replacement deletes the predecessor.

Rule strings may be assembled from compile-time string fragments with f-strings. Every interpolated value must itself be a compile-time string.

```python
left_branch = "[-F]"
right_branch = "[+F]"
rule = f"F{left_branch}F{right_branch}F"

geo = ls.system(
    ls.axiom("F"),
    ls.rule("F", rule),
    ls.iterations(3),
    ls.angle(25),
    ls.step(0.08),
)

output("Geometry", geo)
```

### `ls.iterations(value)`

Sets the number of rewrite iterations.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time `Integer` or runtime `Int` | Rewrite iteration count. |

A compile-time integer must be non-negative. The complete L-string is expanded while the node group is compiled.

A runtime `Int`, such as `input_int(...)`, performs rewriting inside Geometry Nodes. Changing the value can therefore change the number of segments, branches, and markers without recompiling the script. Runtime values below zero are treated as `0`.

```python
iterations = input_int("Iterations", default=3)

geo = ls.system(
    ls.axiom("A"),
    ls.rule("A", "F[+A][-A]"),
    ls.iterations(iterations),
    ls.angle(30),
    ls.step(0.2),
)

output("Geometry", geo)
```

When ls.iterations() uses a runtime Int, every branch must be self-contained within the axiom or within a single rule replacement. Each axiom and replacement must therefore have matching [ and ]. The [ and ] symbols themselves cannot have rewrite rules. This guarantees that the branch stack remains valid for any runtime iteration count.

### `ls.angle(value)`

Sets the default turtle rotation angle in degrees.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time number or runtime numeric `Value` | Default angle used by unparameterized `+`, `-`, `^`, `&`, `/`, and `\`. |

An explicit command argument such as `&(35)` or `/(roll)` overrides `ls.angle(...)` for that command.

```python
angle = input_float("Angle", default=30.0)

geo = ls.system(
    ls.axiom("F[+F][-F][^F][&F]"),
    ls.iterations(0),
    ls.angle(angle),
    ls.step(1),
)

output("Geometry", geo)
```

### `ls.step(value)`

Sets the default forward distance.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time number or runtime numeric `Value` | Default distance used by unparameterized `F` and `f`. |

An explicit command argument such as `F(0.5)` or `F(length)` overrides `ls.step(...)` for that command.

```python
step = input_float("Step", default=0.25)

geo = ls.system(
    ls.axiom("F+F+F+F"),
    ls.iterations(0),
    ls.angle(90),
    ls.step(step),
)

output("Geometry", geo)
```

### `ls.param(name, value)`

Declares a named numeric value for parameterized turtle commands and marker modules.

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | compile-time `String` | Identifier matching `[A-Za-z_][A-Za-z0-9_]*`. |
| `value` | compile-time number or runtime numeric `Value` | Numeric value bound to the name. |

Module arguments may contain either a numeric literal or one declared parameter name. They are not NodeForge expressions.

```python
length = input_float("Length", default=1.0)
turn = input_float("Turn", default=45.0)

geo = ls.system(
    ls.axiom("F(length)+(turn)F(length)"),
    ls.iterations(0),
    ls.angle(90),
    ls.step(0.25),
    ls.param("length", length),
    ls.param("turn", turn),
)

output("Geometry", geo)
```

### `ls.marker(name, *parameter_names)`

Declares a marker module. A marker emits a point at the current turtle position without moving or rotating the turtle.

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | compile-time `String` | Marker identifier. It cannot be a built-in turtle command. |
| `parameter_names` | compile-time `String` values | Optional point-attribute names matched to marker arguments. |

Marker names and parameter names use identifier syntax. Parameter names must be unique inside one marker and cannot start with the reserved `nf_lsys_` prefix.

Marker points carry the turtle orientation in the `nf_lsys_marker_tangent` and `nf_lsys_marker_up` point attributes. Each declared marker parameter is also stored as a point attribute with the declared name.

```python
size = input_float("Leaf Size", default=0.25)

plant = ls.system(
    ls.axiom("FLeaf(size)+(45)FLeaf(size)"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
    ls.param("size", size),
    ls.marker("Leaf", "size"),
)

leaf_points = ls.points(plant, marker="Leaf")
leaves = instance_on_points(cube(0.15), leaf_points)

output("Geometry", join(plant, leaves))
```

### `ls.points(geometry, marker="Name")`

Extracts the points emitted by one declared marker.

| Parameter | Type | Description |
| --- | --- | --- |
| `geometry` | `Geometry` | Geometry containing L-system marker attributes. |
| `marker` | compile-time `String` | Required keyword selecting the marker name. |

Returns: `Geometry` containing only points for the requested marker.

Marker identity is stored in numeric point attributes, so marker filtering continues to work after normal geometry operations such as assignment, `transform(...)`, and `join(...)` when Blender preserves those attributes.

```python
plant = ls.system(
    ls.axiom("FLeaf+(60)FBud"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
    ls.marker("Leaf"),
    ls.marker("Bud"),
)

leaf_points = ls.points(plant, marker="Leaf")
bud_points = ls.points(plant, marker="Bud")
leaves = instance_on_points(cube(0.18), leaf_points)
buds = instance_on_points(cube(0.10), bud_points)

output("Geometry", join(plant, leaves, buds))
```

## Turtle Coordinate Frame

The turtle starts at the world origin with this local frame:

| Axis | Initial direction | Meaning |
| --- | --- | --- |
| Heading | `+X` | Forward direction used by `F` and `f`. |
| Left | `+Y` | Local pitch axis. |
| Up | `+Z` | Local yaw axis. |

Yaw, pitch, and roll update this local frame. Rotations are therefore order-dependent: `+(90)^(90)` and `^(90)+(90)` produce different orientations.

`[` saves the complete turtle state, including position and all three orientation axes. `]` restores that state.

## Turtle Commands

All rotation arguments are measured in degrees.

| Module | Meaning |
| --- | --- |
| `F` | Move forward by `ls.step(...)` and draw a segment. |
| `F(length)` | Move forward by `length` and draw a segment. |
| `f` | Move forward by `ls.step(...)` without drawing. |
| `f(length)` | Move forward by `length` without drawing. |
| `+` | Yaw left around local Up by `ls.angle(...)`. |
| `+(angle)` | Yaw left by `angle`. |
| `-` | Yaw right around local Up by `ls.angle(...)`. |
| `-(angle)` | Yaw right by `angle`. |
| `^` | Pitch up around local Left by `ls.angle(...)`. |
| `^(angle)` | Pitch up by `angle`. |
| `&` | Pitch down around local Left by `ls.angle(...)`. |
| `&(angle)` | Pitch down by `angle`. |
| `/` | Roll around local Heading by the positive rotation angle. |
| `/(angle)` | Roll around local Heading by positive `angle`. |
| `\` | Roll around local Heading by the negative rotation angle. |
| `\(angle)` | Roll around local Heading by negative `angle`. |
| `[` | Save position and orientation, then begin a branch. |
| `]` | Restore the most recently saved position and orientation. |

A parameterized command accepts exactly one numeric literal or one name declared with `ls.param(...)`.

Other ASCII letters, digits, and `_` are one-character grammar symbols. They participate in rewriting and are ignored by turtle drawing if they remain in the final stream. Declared multi-character marker names are recognized as marker modules; other grammar symbols remain one character each.

Whitespace, unsupported punctuation, Unicode grammar symbols, undeclared parameter names, malformed argument lists, unmatched `]`, and unclosed `[` raise `CompileError`.

## Runtime Controls

The following values can be changed through node-group inputs without recompiling when supplied as runtime values:

| Constructor | Runtime type | Effect |
| --- | --- | --- |
| `ls.iterations(...)` | `Int` | Rewrites the L-string at evaluation time and can change topology. |
| `ls.angle(...)` | numeric | Changes unparameterized yaw, pitch, and roll angles. |
| `ls.step(...)` | numeric | Changes unparameterized forward distances. |
| `ls.param(...)` | numeric | Changes any command or marker argument that references the parameter. |

The axiom, rule strings, parameter names, marker declarations, and rule set remain compile-time declarations.

```python
iterations = input_int("Iterations", default=4)
pitch = input_float("Pitch", default=35.0)
roll = input_float("Roll", default=137.5)
step = input_float("Step", default=0.2)

plant = ls.system(
    ls.axiom("A"),
    ls.rule("A", "F[&(pitch)B]/(roll)A"),
    ls.rule("B", "FLeaf"),
    ls.iterations(iterations),
    ls.angle(25),
    ls.step(step),
    ls.param("pitch", pitch),
    ls.param("roll", roll),
    ls.marker("Leaf"),
)

output("Geometry", plant)
output("Leaf Points", ls.points(plant, marker="Leaf"))
```

## Limits

L-system size commonly grows exponentially with the iteration count. Use the smallest iteration range needed for the model.

| Limit | Applies to | Behavior |
| --- | --- | --- |
| `200000` expanded modules | Compile-time `ls.iterations(...)` | Compilation raises `CompileError` when expansion exceeds the limit. |
| Branch depth `32` | Compile-time-expanded planar branched systems using runtime angle or step values | Compilation raises `CompileError` when branch nesting exceeds the backend limit. |

Runtime `Int` iterations are expanded during Geometry Nodes evaluation rather than by the compile-time `200000`-module guard. Large runtime values can therefore create very large evaluated geometry and should be constrained at the input level.

A compiled system must have a possible drawable segment or marker for the selected static/runtime backend. Systems that resolve to no drawable content in a compile-time-expanded path raise `CompileError`.
