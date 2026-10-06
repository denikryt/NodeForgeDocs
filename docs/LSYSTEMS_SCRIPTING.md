# Writing L-System Scripts

Install the `nodeforge.lsystem` package through **Library → Packages** and enable **Allow executable Python** before importing it. The examples use `from packages import lsystem as ls`; package qualification is required for `ls.points(...)` because Core DSL also has a `points(...)` callable.

An L-system script has two stages:

1. The grammar starts from an axiom and applies rewrite rules for the selected number of iterations.
2. A 3D turtle reads the resulting modules and creates curves and marker points.

Use [L-System Reference](LSYSTEMS.md) for constructor signatures and the complete command table.

## Minimal L-System

This script draws a branching curve with runtime controls for angle and segment length.

```python
from packages import lsystem as ls

angle = input_float("Angle", default=25.0)
step = input_float("Step", default=0.1)

geo = ls.system(
    ls.axiom("F"),
    ls.rule("F", "F[+F]F[-F]F"),
    ls.iterations(2),
    ls.angle(angle),
    ls.step(step),
)

output("Geometry", geo)
```

The grammar is evaluated in this order:

1. Start with the axiom `F`.
2. Replace every `F` with `F[+F]F[-F]F`.
3. Repeat the rewrite for the requested iteration count.
4. Interpret `F`, `+`, `-`, `[`, and `]` as turtle commands.

Rules are parallel. Each iteration reads one complete generation and produces the next generation; replacements created during that iteration are not rewritten again until the next iteration.

## Axiom and Grammar Symbols

`ls.axiom(...)` defines the initial stream. A visible curve can start directly with `F`:

```python
geo = ls.system(
    ls.axiom("F+F+F"),
    ls.iterations(0),
    ls.angle(120),
    ls.step(1),
)

output("Geometry", geo)
```

Grammar symbols such as `A`, `B`, or `X` can control growth without drawing anything themselves. They become visible only when rewrite rules eventually produce drawing commands or markers.

```python
geo = ls.system(
    ls.axiom("X"),
    ls.rule("X", "F+X"),
    ls.iterations(5),
    ls.angle(60),
    ls.step(0.15),
)

output("Geometry", geo)
```

Symbols without a rule stay unchanged from one generation to the next. If they reach the final stream and are not turtle commands or declared markers, the turtle ignores them.

A rule may also delete a symbol with an empty replacement. This is useful for temporary grammar symbols.

```python
geo = ls.system(
    ls.axiom("FAF"),
    ls.rule("A", ""),
    ls.iterations(1),
    ls.angle(90),
    ls.step(0.5),
)

output("Geometry", geo)
```

## Branches

`[` saves the current position and orientation. `]` restores the saved state.

```python
geo = ls.system(
    ls.axiom("F[+F]F[-F]F"),
    ls.iterations(0),
    ls.angle(35),
    ls.step(0.4),
)

output("Geometry", geo)
```

Read the stream as:

```text
F     draw the trunk forward
[+F]  save state, turn left, draw a side branch, restore state
F     continue the trunk
[-F]  save state, turn right, draw a side branch, restore state
F     continue the trunk
```

The saved state includes the full 3D orientation, not only position. A roll or pitch made inside a branch therefore does not change the parent branch after `]`.

## 3D Turtle Orientation

The turtle begins at the origin, facing `+X`, with local Left along `+Y` and local Up along `+Z`.

The six rotation commands are:

| Command | Rotation |
| --- | --- |
| `+` | yaw left around local Up |
| `-` | yaw right around local Up |
| `^` | pitch up around local Left |
| `&` | pitch down around local Left |
| `/` | positive roll around local Heading |
| `\` | negative roll around local Heading |

Every unparameterized rotation uses `ls.angle(...)`. Add one argument to give a command its own angle.

```python
pitch = input_float("Pitch", default=35.0)
roll = input_float("Roll", default=60.0)

geo = ls.system(
    ls.axiom("F/(roll)[&(pitch)F]F"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
    ls.param("pitch", pitch),
    ls.param("roll", roll),
)

output("Geometry", geo)
```

Roll changes the local Left and Up axes. It does not move the turtle by itself, but it changes the plane used by later pitch or yaw operations. Rotation order therefore matters.

For example, this script exposes two curves that use the same two rotations in different orders:

```python
first = ls.system(
    ls.axiom("+(90)^(90)F"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
)

second = ls.system(
    ls.axiom("^(90)+(90)F"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
)
second = transform(second, translation=vector(2, 0, 0))

output("Geometry", join(first, second))
```

## Parameterized Commands

Use command arguments when different modules need different distances or angles.

The movement commands accept one distance:

```text
F(0.5)
f(2.0)
```

The rotation commands accept one angle in degrees:

```text
+(30)
-(45)
^(20)
&(20)
/(90)
\(90)
```

A module argument can also reference a value declared with `ls.param(...)`. The declared value may be a runtime input.

```python
long_step = input_float("Long Step", default=1.5)
short_step = input_float("Short Step", default=0.5)
turn = input_float("Turn", default=70.0)

geo = ls.system(
    ls.axiom("F(long_step)+(turn)F(short_step)-(turn)F(long_step)"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
    ls.param("long_step", long_step),
    ls.param("short_step", short_step),
    ls.param("turn", turn),
)

output("Geometry", geo)
```

Arguments inside L-system strings are data references, not NodeForge expressions. Use a numeric literal or a single `ls.param(...)` name. Compute a value in the NodeForge script first, then bind the result with `ls.param(...)` when a more complex expression is needed.

```python
base = input_float("Base Length", default=0.4)
length = base * 2.0

geo = ls.system(
    ls.axiom("F(length)+(90)F(length)"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(1),
    ls.param("length", length),
)

output("Geometry", geo)
```

## Runtime Growth

Pass a runtime `Int` to `ls.iterations(...)` when the number of rewrite generations should be adjustable from the node-group interface.

```python
iterations = input_int("Iterations", default=4)

geo = ls.system(
    ls.axiom("A"),
    ls.rule("A", "FA"),
    ls.iterations(iterations),
    ls.angle(25),
    ls.step(0.25),
)

output("Geometry", geo)
```

Changing **Iterations** changes topology without recompiling the script. A runtime value of `0` uses the axiom directly. Runtime values below `0` are treated as `0`.

Runtime rewriting still uses compile-time grammar declarations. The axiom, rules, marker declarations, and parameter names do not become runtime strings.

For runtime iterations, keep every branch local to one declared stream: the axiom and every rule replacement must each have balanced brackets. This form is valid:

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

A replacement that opens a branch while another replacement closes it is not valid in runtime iteration mode. Rules that rewrite `[` or `]` are also reserved from this mode.

## Marker Points

Markers place points in the turtle stream without drawing or moving the turtle. Declare the marker with `ls.marker(...)`, use its name in the axiom or a rule, and extract it with `ls.points(...)`.

```python
plant = ls.system(
    ls.axiom("F[+FLeaf][-FLeaf]FLeaf"),
    ls.iterations(0),
    ls.angle(35),
    ls.step(0.5),
    ls.marker("Leaf"),
)

leaf_points = ls.points(plant, marker="Leaf")
leaves = instance_on_points(cube(0.12), leaf_points)

output("Geometry", join(plant, leaves))
```

A marker can carry numeric parameters. Each parameter becomes a point attribute with the name declared in `ls.marker(...)`.

```python
size = input_float("Leaf Size", default=0.25)

plant = ls.system(
    ls.axiom("FLeaf(size)+(45)FLeaf(size)"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(0.7),
    ls.param("size", size),
    ls.marker("Leaf", "size"),
)

leaf_points = ls.points(plant, marker="Leaf")
leaves = instance_on_points(cube(0.12), leaf_points)

output("Geometry", join(plant, leaves))
```

Marker points also carry two orientation vectors:

- `nf_lsys_marker_tangent` is the turtle Heading at the marker.
- `nf_lsys_marker_up` is the turtle Up vector at the marker.

These vectors follow yaw, pitch, roll, and branch restoration, so marker orientation remains meaningful in spatial L-systems.

## Spatial Plant with Runtime Growth

This example combines runtime iterations, pitch, roll, a runtime step length, and leaf markers. It is a compact starting point for procedural 3D plants.

```python
growth_iterations = input_int("Growth Iterations", default=5)
branch_angle = input_float("Branch Angle", default=35.0)
node_rotation = input_float("Node Rotation", default=137.507764)
step = input_float("Step", default=0.2)

plant = ls.system(
    ls.axiom("A"),
    ls.rule("A", "F[&(branch_angle)B]/(node_rotation)A"),
    ls.rule("B", "FLeaf"),
    ls.iterations(growth_iterations),
    ls.angle(25),
    ls.step(step),
    ls.param("branch_angle", branch_angle),
    ls.param("node_rotation", node_rotation),
    ls.marker("Leaf"),
)

leaf_points = ls.points(plant, marker="Leaf")
leaves = instance_on_points(cube(0.08), leaf_points)

output("Geometry", join(plant, leaves))
```

The roll angle separates successive branches around the trunk. The pitch angle moves each branch away from the current heading. Because both values are runtime parameters, their shape can be adjusted without recompiling.

## Compile-Time String Composition

Use f-strings when the grammar itself is assembled from compile-time string fragments.

```python
left = "[-F]"
right = "[+F]"
rule = f"F{left}F{right}F"

geo = ls.system(
    ls.axiom("F"),
    ls.rule("F", rule),
    ls.iterations(3),
    ls.angle(25),
    ls.step(0.1),
)

output("Geometry", geo)
```

F-string interpolation changes the compile-time grammar. Runtime numeric controls belong in `ls.param(...)` instead.

## Transforming and Joining Results

`ls.system(...)` returns ordinary geometry. Apply NodeForge geometry operations after the L-system is built.

```python
left = ls.system(
    ls.axiom("F[+F][-F]F"),
    ls.iterations(2),
    ls.angle(30),
    ls.step(0.15),
)

right = ls.system(
    ls.axiom("F[+F][-F]F"),
    ls.iterations(3),
    ls.angle(22),
    ls.step(0.12),
)
right = transform(right, translation=vector(1.5, 0, 0))

output("Geometry", join(left, right))
```

Use `translation=` for positional transforms.

## Common Patterns

### Koch Curve

```python
geo = ls.system(
    ls.axiom("F"),
    ls.rule("F", "F+F--F+F"),
    ls.iterations(4),
    ls.angle(60),
    ls.step(0.08),
)

output("Geometry", geo)
```

### Classic Fractal Plant

```python
angle = input_float("Angle", default=25.0)
step = input_float("Step", default=0.04)

geo = ls.system(
    ls.axiom("X"),
    ls.rule("X", "F+[[X]-X]-F[-FX]+X"),
    ls.rule("F", "FF"),
    ls.iterations(5),
    ls.angle(angle),
    ls.step(step),
)
geo = transform(geo, rotation=vector(0, 0, pi / 2))

output("Geometry", geo)
```

### 3D Branch Whorl

```python
pitch = input_float("Pitch", default=35.0)
roll = input_float("Roll", default=120.0)

geo = ls.system(
    ls.axiom("F[&(pitch)F]/(roll)[&(pitch)F]/(roll)[&(pitch)F]"),
    ls.iterations(0),
    ls.angle(25),
    ls.step(0.8),
    ls.param("pitch", pitch),
    ls.param("roll", roll),
)

output("Geometry", geo)
```

## Grammar Requirements

Use these rules when writing axioms and replacements:

- Keep whitespace out of L-system strings.
- Use ASCII letters, digits, and `_` for ordinary grammar symbols.
- Use only declared marker names for multi-character modules.
- Give parameterized built-ins exactly one numeric literal or declared parameter name.
- Declare every named module argument with `ls.param(...)`.
- Match every `[` with a later `]` in the interpreted stream.
- For runtime iterations, balance brackets separately in the axiom and in every replacement.

The compiler reports invalid streams as `CompileError` before producing the node group whenever the invalid structure is known at compile time.

## Performance

L-system growth can be exponential. A rule that produces two recursive symbols may approximately double the active grammar each iteration; rules that produce three or more recursive symbols grow faster.

Compile-time iteration mode stops expansion above `200000` modules. Runtime iteration mode performs the expansion during Geometry Nodes evaluation, so choose practical input ranges for **Iterations** instead of exposing an unrestricted large value.
