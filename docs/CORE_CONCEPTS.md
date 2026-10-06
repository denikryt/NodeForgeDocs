# Core Concepts

NodeForge is a domain-specific language (DSL) for authoring Blender Geometry Nodes groups. A source file describes one group: its interface, the operations inside it, and the data flow between those operations. The compiler turns that description into a native `GeometryNodeTree`.

## One script describes one node group

The top-level script owns the generated group interface and body.

```python
size = input_float('Size', default=1.0)
scaled = size * 2.0
geometry = cube(size=scaled)
output('Geometry', geometry)
```

This source describes the following parts of the group:

| Source | Node-group result |
| --- | --- |
| `input_float('Size', ...)` | A Float input socket named **Size**. |
| `size * 2.0` | A math operation linked to the **Size** input. |
| `cube(size=scaled)` | A Cube operation whose size receives the math result. |
| `output('Geometry', geometry)` | A Geometry output socket linked to the cube result. |

The generated data flow is approximately:

```text
Size group input → Math (Multiply) → Cube → Geometry group output
```

The variable `size` is a compiler handle for a typed Geometry Nodes value. It is not the current numeric value of the socket. Changing **Size** on a group-node instance makes Blender evaluate the existing graph with the new value.

Assignments give readable names to values in this data flow. Arithmetic, comparisons, function calls, and supported control flow either produce graph operations or guide the compiler while it builds the graph.

## DSL compilation

NodeForge uses familiar Python syntax as the surface language and gives that syntax Geometry Nodes semantics. The compiler processes a script through these stages:

```text
NodeForge source
    ↓
Python AST parsing
    ↓
DSL validation and compile-time preprocessing
    ↓
runtime type and group-interface inference
    ↓
Geometry Nodes materialization
    ↓
native GeometryNodeTree
```

The compiler tracks runtime types such as `Geometry`, `Float`, `Int`, `Bool`, `Vector`, `Material`, `Object`, `String`, and `Bundle`. These types determine which sockets and operations can be connected. Compile-time values use compiler-owned representations such as numbers, strings, vectors, lists, and tuples.

The documented language defines which Python syntax has NodeForge meaning. See [DSL Syntax and Semantics](SYNTAX.md) for the supported statements, expressions, types, and restrictions.

## Compile time

Compile-time values are available while NodeForge is deciding what graph to create. They can control socket names, defaults, options, list structure, imports, and authoring loops.

```python
count = 3
spacing = input_float('Spacing', default=1.5)

parts = []
for index in range(count):
    offset = vector(index * spacing, 0, 0)
    part = transform(cube(size=1.0), translation=offset)
    parts.append(part)

output('Geometry', join(parts))
```

Here `count`, `index`, and the list structure are compile-time data. The compiler unrolls `range(3)` and creates three cube-and-transform paths. `spacing` remains a runtime socket value, so each generated offset path stays connected to the adjustable **Spacing** input.

Compile-time arithmetic can be resolved before nodes are created when the receiving operation permits it. Ordinary statement `if` is deliberately different: it always represents runtime control flow, even when its condition is a literal or another compile-time-known `Bool`.

## Runtime

Runtime values exist as sockets, fields, and state in the generated Geometry Nodes graph. Values returned by `input_*()` are group inputs, so they can be changed on each group-node instance without recompiling the script.

```python
steps = input_int('Steps', default=4)
distance = input_float('Distance', default=0.5)
geometry = cube(size=1.0)

for index in repeat_range(steps):
    geometry = transform(
        geometry,
        translation=vector(distance, 0, 0),
    )

output('Geometry', geometry)
```

`repeat_range(...)` creates a native Blender Repeat Zone. The compiler materializes the loop body inside the zone, carries changed values as repeat state, and connects **Steps** to the zone's iteration count. Changing **Steps** changes the evaluated iteration count without rebuilding the group.

The two loop forms serve different authoring needs:

| Form | Compilation result |
| --- | --- |
| `for ... in range(...)` | The compiler unrolls the body and generates one graph fragment per iteration. The range arguments must be known at compile time. |
| `for ... in repeat_range(...)` | The compiler creates a Repeat Zone and materializes the body once inside it. The iteration count may be a runtime `Int`. |

Every ordinary `if` statement is runtime control flow. NodeForge validates both branches and materializes compatible branch results through Switch nodes, even for a condition such as `True` or `False`. A top-level `if` therefore requires an `else` branch and both branches must assign at least one compatible common runtime value.

## Group inputs are runtime parameters

An `input_*()` declaration creates an exposed node-group socket. The source supplies its label, type, and default value.

```python
radius = input_float('Radius', default=1.0)
segments = input_int('Segments', default=32)
enabled = input_bool('Enabled', default=True)
```

Every instance of the generated group can have its own input values or incoming links. These parameters participate in the surrounding Geometry Nodes graph and update through normal Blender evaluation.

An `output(...)` declaration creates a group output and connects the selected runtime value to it. The script therefore defines both the public interface and the internal implementation of the group.

## Generated groups remain native Blender data

The compiler creates ordinary Blender nodes, sockets, links, interface panels, and zones. The resulting group can be opened and inspected in the Geometry Node Editor, connected from another graph, copied, and saved in a `.blend` file. Compiled groups continue evaluating when the NodeForge add-on is disabled or uninstalled.

## Embedded source and updates

Every generated NodeForge group stores the source that created it. Select a generated group node and use **Load Script From Selected NodeGroup** to copy the embedded source into a Blender Text datablock.

After editing the source, use **Update Selected NodeGroup** to rebuild the selected group in place. NodeForge compiles a replacement first and then updates the existing group datablock. Explicit `input_*()` declarations carry compiler-owned declaration identities, so compatible incoming links and user-overridden input values follow the declaration rather than its displayed socket label. Renaming a label, reordering declarations, or adding another input with the same label does not by itself retarget preserved state. Outgoing links are restored when the rebuilt public output remains compatible.

This workflow keeps the generated group connected to the surrounding graph while its implementation evolves:

1. Load or open the source.
2. Edit and test the DSL script.
3. Select the generated group node.
4. Run **Update Selected NodeGroup**.
5. Inspect the rebuilt internals and the preserved external connections.

Keep reusable source in `.nf` files or another version-controlled source location. The embedded copy makes each generated group self-describing inside the `.blend` file, while the source file provides a practical history for reviewing and reproducing changes.
