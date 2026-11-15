# Writing Functions

Declare reusable functions with top-level `def` in a `.nf` module. Each parameter has a type annotation, and each function ends with a `return`. A file can declare several functions; [`.nf` Modules](MODULES.md#compose-the-base-tree) explains when their Group Nodes are placed in the base tree.

## Parameters and sockets

```python
def move(
    geometry: Geometry,
    offset: Vector["Translation"] = vector(0, 0, 1)
) -> Geometry["Moved Geometry"]:
    return transform(geometry, translation=offset)
```

The input sockets appear in parameter order. `geometry` is required in an explicit call; `offset` has a default. The parameter identifier `offset` is also its keyword name. `Vector["Translation"]` changes only the Blender socket label. Every parameter needs an explicit type annotation, including parameters with defaults.

Function parameters support `Geometry`, `Float`, `Int`, `Bool`, `Vector`, `Material`, `Object`, `String`, and `Bundle`. You can declare defaults for `Float`, `Int`, `Bool`, `Vector`, and `String`. Parameters of the other four types are required.

```python
result = move(cube(1.0), offset=vector(0, 0, 2))
```

Defaults must be known at compile time and can use constants from the defining module. Use `panel([geometry, offset], name="Transform", collapsed=True)` at the start of a function body to group its inputs in Blender's interface.

## Return values

Return one runtime value or a flat, nonempty tuple of values. A single `Type["Label"]` return annotation sets its output socket label. Otherwise a returned variable supplies the label, or an expression uses `Value`.

```python
def measure(geometry: Geometry, scale: Float = 2.0):
    resized = transform(geometry, scale=vector(scale, scale, scale))
    doubled = scale * 2.0
    return resized, doubled

result, factor = measure(cube(1.0))
```

The caller can unpack multiple outputs or select one with a compile-time integer index. A function can call other functions in its module. Recursive call cycles are not supported.

Save the module in Local to reuse its functions in other scripts. See [Import functions](MODULES.md#import-functions).

## Refresh a compiled function

After editing the Local source, select a source-backed Group Node and click **Reload Function from Source** to rebuild it. For the base tree, see [Update a script](GET_STARTED.md#update-a-script).
