# Core DSL Built-ins Reference

Core built-ins are primitive operations registered by the NodeForge add-on and available without imports.

## Type Names

| Type | Meaning |
| --- | --- |
| `Geometry` | Blender geometry socket. |
| `Float` | Blender-compatible binary32 floating-point value or field. |
| `Int` | Signed 32-bit integer value or field. |
| `Bool` | Boolean value or field. |
| `Vector` | 3-component vector value or field. |
| `Material` | Blender material socket. |
| `Object` | Blender object socket with Object Info access. |
| `String` | Blender string socket. String literals can also be used as compile-time names and options. |
| `Bundle` | Blender bundle socket containing named runtime values. |
| `Float`, `Int`, `Bool`, `Vector`, `Geometry`, `Material`, `Object`, `String`, `Bundle` | Type tokens used by `node(..., typ=...)`, `node(..., outputs=...)`, local parameter annotations, and `bundle_get(..., typ=...)`. |

NodeForge values are typed socket wrappers. A value can be a constant lowered to a node, a linked runtime field, or geometry. Most built-ins accept either literal values or runtime values of the declared type.

## Compile-Time Constants

NodeForge exposes three numeric constants in every source file.

| Constant | Value |
| --- | --- |
| `pi` | π, approximately `3.141592653589793` |
| `tau` | 2π, approximately `6.283185307179586` |
| `e` | Euler’s number, approximately `2.718281828459045` |

```python
turn = tau
half_turn = pi
base = e
output('Turn', turn)
output('Half Turn', half_turn)
output('Base', base)
```

## Compile-Time Helper Calls

These helpers are evaluated during compilation. They do not create Geometry Nodes sockets and cannot operate on runtime values.

### `range(stop)`
### `range(start, stop)`
### `range(start, stop, step)`

Creates a compile-time list of integers. Use it mainly for unrolled `for` loops or compile-time indexing.

| Parameter | Type | Description |
| --- | --- | --- |
| `start` | compile-time `Int` | First integer. Defaults to `0`. |
| `stop` | compile-time `Int` | Exclusive upper bound. |
| `step` | compile-time `Int` | Step size. Defaults to `1`. |

Returns: compile-time `List[Int]`.

```python
BASE_COUNT = 4
EXTRA_COUNT = 2
COUNT = BASE_COUNT + EXTRA_COUNT

items = []
for i in range(COUNT):
    offset = vector(i * 1.25, 0, 0)
    geo = cube(size=1.0)
    moved = transform(geo, translation=offset)
    items.append(moved)
result = join(items)
output('Geometry', result)
```

### `len(value)`

Returns the length of a compile-time list, tuple, or string.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time `List`, `Tuple`, or `String` | Sequence whose length is known during compilation. |

Returns: compile-time `Int`.

```python
sizes = [0.5, 1.0, 1.5]
count = len(sizes)
step = 1.0 / count
value = input_float('Value', default=step)
output('Value', value)
```

### `sum(value)`

Returns the numeric sum of a compile-time list or tuple.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | compile-time numeric `List` or `Tuple` | Sequence whose elements can be added during compilation. |

Returns: compile-time `Int` when every element is `Int` (and for an empty sequence); otherwise compile-time `Float`. Reduction is left-to-right, using the same signed-32 `Int` and binary32 `Float` arithmetic as source expressions. `Bool` elements are rejected.

```python
sizes = [0.5, 1.0, 1.5]
total = sum(sizes)
count = len(sizes)
average = total / count
geo = cube(size=average)
output('Geometry', geo)
```

## Outputs

### `output(value)`
### `output(name, value)`
### `output(name="Name", value=value)`

Creates a group output socket and connects a runtime value to it.

| Parameter | Type | Description |
| --- | --- | --- |
| `name` | `String` | Optional output socket name. When omitted, the output socket is named `out`. Duplicate output names receive suffixes such as `_2`. |
| `value` | runtime value | `Float`, `Int`, `Bool`, `Vector`, `Geometry`, `Material`, `Object`, `String`, or `Bundle` value to expose on the node group. Arrays and multi-output results cannot be output directly; index or unpack them first. |

Returns: statement only.

```python
geo = cube(size=2.0)
height = input_float('Height', default=1.0)
vec_1 = vector(1, 1, height)
geo_2 = transform(geo, scale=vec_1)
output('Geometry', geo_2)
output(name='Height', value=height)
```

Automatic final outputs are part of the source language semantics. See [DSL Syntax and Semantics](SYNTAX.md#group-inputs-and-outputs).

## Active Geometry stream statements

These statement-only forms operate on an implicit Geometry input/output stream. When a script contains `store(...)` or statement-form `set_position(...)`, NodeForge creates a Geometry group input named `Geometry` and a Geometry group output named `Geometry`.

Use these forms when the node group is meant to modify geometry that is passed into it. Use expression-form `store_named_attribute(geometry, ...)` and `set_position(geometry, ...)` when you want to operate on an explicit Geometry value inside the script.

### `store(name, value, selection=True, domain="POINT", type=None)`

Stores a named attribute on the active Geometry stream.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | compile-time or runtime `String` | required | Attribute name. |
| `value` | `Float`, `Int`, `Bool`, or `Vector` | required | Attribute value field. Arrays are rejected. |
| `selection` | `Bool` | `True` | Optional field mask. |
| `domain` | compile-time `String` | `"POINT"` | `"POINT"`, `"EDGE"`, `"FACE"`, `"CORNER"`, `"CURVE"`, or `"INSTANCE"`. |
| `type` | compile-time `String` or `None` | `None` | Optional Blender data type override, for example `"FLOAT"`, `"INT"`, `"BOOLEAN"`, `"VECTOR"`, or `"COLOR"`. |

Returns: statement only.

```python
height = position().z
store('height', height, domain='POINT', type='FLOAT')
```

The attribute name can come from `input_string(...)` when it must be selected at runtime.

### `set_position(position, selection=True)`

Sets point positions on the active Geometry stream.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `position` | `Vector` | required | New position field. |
| `selection` | `Bool` | `True` | Optional field mask. |

Returns: statement only.

```python
pos = position()
height = pos.x * 0.5
offset = vector(0, 0, height)
new_pos = pos + offset
set_position(new_pos)
```

## Inputs

Input built-ins create group input sockets. The `name` argument must be a compile-time string. Defaults must be compile-time values.

Use each `input_*()` call as the complete right-hand side of a simple assignment. Every call creates a separate interface socket; `name` is its displayed label rather than the identity of the declaration. Two declarations may therefore use the same label and keep independent defaults.

### `input_geometry(name)`

Creates a Geometry input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |

Returns: `Geometry`.

```python
geo = input_geometry('Geometry')
vec_1 = vector(1.5, 1.5, 1.5)
geo_2 = transform(geo, scale=vec_1)
output('Geometry', geo_2)
```

### `input_float(name, default=0.0)`

Creates a Float input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |
| `default` | compile-time `Float` | `0.0` | Socket default value. |

Returns: `Float`.

```python
radius = input_float('Radius', default=2.0)
geo_1 = cube(size=radius)
output('Geometry', geo_1)
```

### `input_int(name, default=0)`

Creates an Int input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |
| `default` | compile-time `Int` | `0` | Socket default value. Numeric constants are converted to integer defaults. |

Returns: `Int`.

```python
count = input_int('Count', default=32)
geo = points(count)
output('Geometry', geo)
```

### `input_bool(name, default=False)`

Creates a Bool input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |
| `default` | compile-time `Bool` | `False` | Socket default value. |

Returns: `Bool`.

```python
enabled = input_bool('Enabled', default=True)
output('Enabled', enabled)
```

### `input_vector(name, default=vector(0, 0, 0))`

Creates a Vector input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |
| `default` | compile-time `Vector` or 3-number tuple/list | `vector(0, 0, 0)` | Socket default value. |

Returns: `Vector`.

```python
vec_1 = vector(0, 0, 2)
offset = input_vector('Offset', default=vec_1)
geo_2 = cube()
geo_3 = transform(geo_2, translation=offset)
output('Geometry', geo_3)
```

### `input_material(name)`

Creates a Material input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |

Returns: `Material`.

```python
material = input_material('Material')
geo = set_material(cube(size=1.0), material)
output('Geometry', geo)
```

### `input_object(name)`

Creates an Object input socket. Read the selected object's geometry and transforms through the Object properties described in [Object values](#object-values).

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |

Returns: `Object`.

```python
source = input_object('Source')
source.info(transform_space='RELATIVE', as_instance=False)
output('Geometry', source.geometry)
```

### `input_string(name, default="")`

Creates a String input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | compile-time `String` | required | Input socket name. |
| `default` | compile-time `String` | `""` | Socket default value. |

Returns: runtime `String`.

```python
attribute_name = input_string('Attribute', default='weight')
geo = store_named_attribute(cube(), attribute_name, position().z)
output('Geometry', geo)
```

### `input_bundle(name)`

Creates a Bundle input socket.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | `String` | required | Input socket name. |

Returns: `Bundle`.

```python
data = input_bundle('Data')
factor = bundle_get(data, 'factor', typ=Float)
output('Factor', factor)
```

## Interface Panels

### `panel([input_a, input_b, ...], name="Name", collapsed=False)`

Places existing group input sockets in a native Blender interface panel. Write `panel(...)` as a standalone top-level declaration after the referenced inputs.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `inputs` | literal list or tuple of input variable names | required | One or more group inputs in their panel order. |
| `name` | compile-time `String` | required | Non-empty panel name. Panel names must be unique. |
| `collapsed` | compile-time `Bool` | `False` | Whether the panel is closed by default. |

Returns: statement only.

Each group input can belong to one panel. Inputs omitted from `panel(...)` remain at the root of the group interface. The DSL creates root-level panels. Computed values and function results are not interface inputs and cannot be panel members.

```python
radius = input_float('Radius', default=1.0)
segments = input_int('Segments', default=32)
panel([radius, segments], name='Shape', collapsed=True)

geo = cube(size=radius)
output('Geometry', geo)
```

## Object Values

An `Object` value exposes the outputs of Blender's Object Info node as properties.

| Property | Returns | Description |
| --- | --- | --- |
| `.geometry` | `Geometry` | Geometry from the selected object. |
| `.location` | `Vector` | Object location in the configured transform space. |
| `.rotation` | `Vector` | Object rotation. |
| `.scale` | `Vector` | Object scale. |

### `object.info(transform_space="ORIGINAL", as_instance=True)`

Configures the Object Info node used by subsequent property access and returns the same `Object` value.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `transform_space` | compile-time `String` | `"ORIGINAL"` | `"ORIGINAL"` or `"RELATIVE"`. |
| `as_instance` | compile-time `Bool` | `True` | Return geometry as an instance when enabled. |

You may call `.info()` more than once to set the options separately. Configure it before reading any Object property; the configuration is fixed when the first property creates the Object Info node.

```python
source = input_object('Source')
source.info(transform_space='RELATIVE')
source.info(as_instance=False)

offset = source.location
geo = transform(source.geometry, translation=-offset)
output('Geometry', geo)
```

## Bundles

Bundles carry named runtime values through one Blender socket. Bundle items support `Float`, `Int`, `Bool`, `Vector`, `Geometry`, `Material`, `Object`, `String`, and nested `Bundle` values. Bundles can cross group and reusable-function sockets and can be carried through runtime `if` and `repeat_range(...)` state.

### `bundle(**items)`

Creates a Bundle from named keyword items. The keyword names become item names.

Returns: `Bundle`.

```python
weight = input_float('Weight', default=0.5)
data = bundle(
    geometry=cube(size=1.0),
    weight=weight,
)
output('Data', data)
```

### `bundle_get(bundle, path, typ=Type)`

Reads an item from a Bundle.

| Parameter | Type | Description |
| --- | --- | --- |
| `bundle` | `Bundle` | Bundle to read. |
| `path` | `String` | Runtime bundle item path. |
| `typ` | type token | Expected item type and output socket type. |

Returns: the runtime type selected by `typ`.

`typ=` is required because an incoming Bundle does not declare the type of each item to the script. The path can be a string literal or a runtime `String`.

```python
data = input_bundle('Data')
path = input_string('Path', default='weight')
weight = bundle_get(data, path, typ=Float)
output('Weight', weight)
```

### `bundle_set(bundle, path, value)`

Stores or replaces one item and returns the resulting Bundle.

| Parameter | Type | Description |
| --- | --- | --- |
| `bundle` | `Bundle` | Input Bundle. |
| `path` | `String` | Runtime bundle item path. |
| `value` | supported runtime value | Item value to store. |

Returns: `Bundle`.

The stored item type is inferred from `value`.

```python
data = input_bundle('Data')
path = input_string('Path', default='weight')
weight = input_float('Weight', default=1.0)
updated = bundle_set(data, path, weight)
output('Data', updated)
```

## Field inputs

Field built-ins read Blender Geometry Nodes context fields. They take no arguments and do not accept keywords.

| Function | Returns | Description |
| --- | --- | --- |
| `position()` | `Vector` | Current element position field. |
| `normal()` | `Vector` | Current element normal field. |
| `index()` | `Int` | Current element index field. |
| `id()` | `Int` | Current element ID field. |

### `position()`

Returns the current element position field. Use it when a formula should be relative to the existing point or vertex position.

```python
pts = points(32)
base_position = position()
offset = vector(0, 0, 1)
new_position = base_position + offset
pts = set_position(pts, new_position)
output('Geometry', pts)
```

### `normal()`

Returns the current element normal field. Use it for displacement or orientation formulas that depend on surface direction.

```python
geo = cube(size=2.0)
surface_normal = normal()
displacement = surface_normal * 0.25
base_position = position()
new_position = base_position + displacement
geo = set_position(geo, new_position)
output('Geometry', geo)
```

### `index()`

Returns the current element index field. Use it for per-element spacing, alternating patterns, and procedural ordering.

```python
pts = points(32)
i = index()
x = i * 0.1
z = i * 0.25
pos = vector(x, 0, z)
pts = set_position(pts, pos)
output('Geometry', pts)
```

### `id()`

Returns the current element ID field. Use it as a stable per-element random seed when available.

```python
pts = points(64)
element_id = id()
offset = (element_id % 10) * 0.1
base_position = position()
delta = vector(0, 0, offset)
new_position = base_position + delta
pts = set_position(pts, new_position)
output('Geometry', pts)
```

## Vector construction and vector math

### `vector(x, y, z)`

Creates a `Vector` from three numeric components. Positional arguments and `x=`, `y=`, `z=` keywords are supported.

| Parameter | Type | Description |
| --- | --- | --- |
| `x` | `Float` | X component. |
| `y` | `Float` | Y component. |
| `z` | `Float` | Z component. |

Returns: `Vector`.

```python
height = input_float('Height', default=2.0)
offset = vector(x=0.0, y=0.0, z=height)
geo_1 = cube()
geo_2 = transform(geo_1, translation=offset)
output('Geometry', geo_2)
```

Vector components can be read with `.x`, `.y`, and `.z`.

```python
vec_1 = vector(1, 2, 3)
v = input_vector('Vector', default=vec_1)
vec_2 = vector(v.x, v.y, 0)
value_3 = length(vec_2)
output('Length XY', value_3)
```

### Vector functions

| Function | Signature | Returns | Description |
| --- | --- | --- | --- |
| `length` | `length(v)` | `Float` | Vector length. |
| `distance` | `distance(a, b)` | `Float` | Distance between two vectors. |
| `dot` | `dot(a, b)` | `Float` | Dot product. |
| `normalize` | `normalize(v)` | `Vector` | Normalized vector. |
| `cross` | `cross(a, b)` | `Vector` | Cross product. |
| `reflect` | `reflect(a, b)` | `Vector` | Reflection vector. |
| `project` | `project(a, b)` | `Vector` | Projection vector. |

```python
vec_1 = vector(1, 1, 0)
n = normalize(vec_1)
vec_2 = vector(0, 0, 1)
side = cross(n, vec_2)
pos = n + side * 0.5
output('Position', pos)
```

## Geometry

### `points(count)`

Creates point geometry with `count` points.

| Parameter | Type | Description |
| --- | --- | --- |
| `count` | `Int` | Number of points. Compile-time constants and runtime Int values are supported. |

Returns: `Geometry`.

```python
count = input_int('Count', default=32)
pts = points(count)
value_1 = index()
vec_2 = vector(value_1 * 0.1, 0, 0)
pts = set_position(pts, vec_2)
output('Geometry', pts)
```

### `grid(width, height)`

Creates a planar mesh grid on the XY plane and stores the generated UV field for `grid_uv()` in the current script scope.

| Parameter | Type | Description |
| --- | --- | --- |
| `width` | `Int` | Number of vertices in X. |
| `height` | `Int` | Number of vertices in Y. |

Returns: `Geometry`.

```python
geo = grid(16, 16)
uv = grid_uv()
height = (uv.x + uv.y) * 0.5
position = vector(uv.x * 2.0, uv.y * 2.0, height)
geo = set_position(geo, position)
output('Geometry', geo)
```

### `grid_uv()`

Returns the UV coordinates produced by the most recent `grid(width, height)` call in the current script scope. `grid_uv()` requires a preceding `grid(...)` call.

Returns: `Vector` with `.x` and `.y` in the `0..1` range.

```python
geo = grid(8, 8)
uv = grid_uv()
height = (uv.x - 0.5) * (uv.y - 0.5)
base_position = position()
offset = vector(0, 0, height)
geo = set_position(geo, base_position + offset)
output('Geometry', geo)
```

### `set_position(geometry, position, selection=True)`

Sets point positions on geometry.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Input geometry. |
| `position` | `Vector` | required | New position field. |
| `selection` | `Bool` | `True` | Optional field mask. |

Returns: `Geometry`.

```python
pts = points(32)
i = index()
mask = (i % 2) == 0
pos = vector(i * 0.1, 0, i * 0.02)
pts = set_position(pts, pos, selection=mask)
output('Geometry', pts)
```

### `store_named_attribute(geometry, name, value, selection=True, domain="POINT", type=None)`

Stores a named attribute on geometry.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Input geometry. |
| `name` | compile-time or runtime `String` | required | Attribute name. |
| `value` | `Float`, `Int`, `Bool`, or `Vector` | required | Attribute value field. Arrays are rejected. |
| `selection` | `Bool` | `True` | Optional field mask. |
| `domain` | compile-time `String` | `"POINT"` | `"POINT"`, `"EDGE"`, `"FACE"`, `"CORNER"`, `"CURVE"`, or `"INSTANCE"`. |
| `type` | compile-time `String` or `None` | `None` | Optional Blender data type override, for example `"FLOAT"`, `"INT"`, `"BOOLEAN"`, `"VECTOR"`, or `"COLOR"`. |

Returns: `Geometry`.

```python
pts = points(64)
i = index()
height = i * 0.02
pos = vector(i * 0.05, 0, height)
pts = set_position(pts, pos)
pts = store_named_attribute(pts, 'height', height, domain='POINT', type='FLOAT')
output('Geometry', pts)
```

The attribute `name` can also be a runtime `String`, for example a value returned by `input_string(...)`.

### `capture_attribute(geometry, value, selection=True, domain="POINT", type=None)`

Captures a field on geometry as an anonymous attribute. The function returns both the updated Geometry and the captured field value.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Geometry on which to capture the field. |
| `value` | `Float`, `Int`, `Bool`, or `Vector` | required | Field to capture. |
| `selection` | `Bool` | `True` | Optional field mask. |
| `domain` | compile-time `String` | `"POINT"` | `"POINT"`, `"EDGE"`, `"FACE"`, `"CORNER"`, `"CURVE"`, or `"INSTANCE"`. |
| `type` | compile-time `String` or `None` | `None` | Optional result type override: `"FLOAT"`, `"INT"`, `"BOOLEAN"`, or `"VECTOR"`. The value type is used when omitted. |

Returns: `(Geometry, captured value)`.

```python
geo = grid(16, 16)
height = position().z
geo, captured_height = capture_attribute(
    geo,
    height,
    domain='POINT',
)
output('Geometry', geo)
output('Height', captured_height)
```

The two results can also be stored as one result and selected with a compile-time index.

```python
captured = capture_attribute(cube(), normal())
geo = captured[0]
captured_normal = captured[1]
```

### `sample_index(geometry, value, index, domain="POINT", clamp=False)`

Samples a field from a specific element index of another geometry. The result has the same runtime type as `value`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Geometry whose selected domain is sampled. |
| `value` | `Float`, `Int`, `Bool`, or `Vector` | required | Field to evaluate on the sampled geometry. |
| `index` | compile-time `Int` or runtime `Int` | required | Element index. Compile-time indices use the same signed 32-bit `Int` domain as runtime integer values. |
| `domain` | compile-time `String` | `"POINT"` | `"POINT"`, `"EDGE"`, `"FACE"`, `"CORNER"`, `"CURVE"`, or `"INSTANCE"`. Matching is case-insensitive and normalized to the canonical token. |
| `clamp` | compile-time `Bool` | `False` | Sets the Sample Index node's Clamp option. |

Returns: the same type as `value`.

```python
geo = grid(8, 8)
source_height = position().z
sampled_height = sample_index(
    geo,
    source_height,
    input_int('Index', default=0),
    domain='POINT',
    clamp=True,
)
output('Height', sampled_height)
```

`domain` and `clamp` configure the generated node and must therefore be known at compile time. `index` may remain a runtime `Int`.

### `set_material(geometry, material)`

Assigns a material to geometry. Pass a runtime `Material` value or a compile-time material name. A named material is created when it does not exist.

| Parameter | Type | Description |
| --- | --- | --- |
| `geometry` | `Geometry` | Input geometry. |
| `material` | `Material` or compile-time `String` | Material socket value or Blender material name. |

Returns: `Geometry`.

```python
geo = cube(size=2.0)
geo = set_material(geo, 'NodeForge Material')
output('Geometry', geo)
```

```python
material = input_material('Material')
geo = set_material(cube(size=2.0), material)
output('Geometry', geo)
```


### `empty_geometry()`

Creates a valid Geometry value with zero elements. Use it as the neutral starting value for generated geometry collections, optional branches, and repeat-loop accumulators.

Returns: `Geometry`.

```python
geo = empty_geometry()
count = input_int('Count', default=3)
for i in repeat_range(count):
    part = transform(cube(0.25), translation=vector(i, 0, 0))
    geo = join(geo, part)
output('Geometry', geo)
```


### `geometry_builder()`

Creates a script-local Geometry accumulator for procedural construction. The builder is a compiler object, not a Geometry value. Add Geometry values with `add(...)` or `extend(...)`, then read `builder.geometry` when a Geometry snapshot is needed.

Supported operations:

| Operation | Description |
| --- | --- |
| `builder.add(geometry)` | Append one Geometry value. |
| `builder.extend([geometry_a, geometry_b])` | Append an array of Geometry values in order. |
| `builder.geometry` | Return the accumulated Geometry at that source position. |

A new builder with no additions returns the same empty Geometry value as `empty_geometry()`. Builder objects cannot be output, aliased, captured by local functions, or passed to built-ins; use `builder.geometry` whenever a normal Geometry value is required.

```python
builder = geometry_builder()
for size in [0.5, 1.0, 1.5]:
    builder.add(cube(size))
output('Geometry', builder.geometry)
```

Inside `repeat_range(...)`, builder mutations become Repeat Zone Geometry state updates. The same builder can combine compile-time additions, runtime-loop additions, and post-loop additions.

```python
builder = geometry_builder()
pos = vector(0, 0, 0)
steps = input_int('Steps', default=4)
for i in repeat_range(steps):
    next_pos = pos + vector(1, 0, 0)
    builder.add(line(pos, next_pos))
    pos = next_pos
output('Geometry', builder.geometry)
```

### `point(position)`

Creates one point and sets its position. `position` can be a literal vector expression or a runtime `Vector` value.

| Parameter | Type | Description |
| --- | --- | --- |
| `position` | `Vector` | Point position. |

Returns: `Geometry`.

```python
pos = input_vector('Position', default=(0, 0, 1))
geo = point(pos)
output('Geometry', geo)
```

### `line(start, end)`

Creates a curve line segment between two endpoints. `start` and `end` can be literal vector expressions or runtime `Vector` values.

| Parameter | Type | Description |
| --- | --- | --- |
| `start` | `Vector` | Start endpoint. |
| `end` | `Vector` | End endpoint. |

Returns: `Geometry`.

```python
start = input_vector('Start', default=(0, 0, 0))
end = input_vector('End', default=(1, 0, 0))
geo = line(start, end)
output('Geometry', geo)
```

### `cube(size=1.0)`

Creates a cube mesh.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `size` | `Float`, `Int`, or `Vector` | `1.0` | Cube side length or per-axis size. Positional and `size=` forms are supported. |

Returns: `Geometry`.

```python
size = input_float('Size', default=1.0)
geo_1 = cube(size=size)
output('Geometry', geo_1)
```

### `join(geometry, ...)`
### `join([geometry_a, geometry_b, ...])`

Joins multiple geometry values. Arguments can be separate geometry values, arrays of geometry values, or one literal list/tuple of geometry values. A supplied empty array such as `join([])` returns `empty_geometry()`. A call with no arguments is still invalid.

| Parameter | Type | Description |
| --- | --- | --- |
| `geometry` | `Geometry` values or an array of `Geometry` values | Geometry values to join. A supplied array may be empty. |

Returns: `Geometry`.

```python
geo_1 = cube(size=0.4)
vec_2 = vector(-1, 0, 0)
geo_3 = transform(geo_1, translation=vec_2)
geo_4 = cube(size=0.4)
vec_5 = vector(1, 0, 0)
geo_6 = transform(geo_4, translation=vec_5)
geo_7 = points(8)
parts = [geo_3, geo_6, geo_7]
geo_8 = join(parts)
output('Geometry', geo_8)

empty_parts = []
empty = join(empty_parts)
```

### `transform(geometry, translation=None, scale=None, rotation=None)`

Transforms geometry. `translation`, `scale`, and `rotation` can be supplied by keyword. The first two transform options can also be supplied as positional arguments after `geometry`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Input geometry. |
| `translation` | `Vector` | no transform | Translation vector. |
| `scale` | `Float`, `Int`, or `Vector` | no transform | Scale value. Use `vector(x, y, z)` for non-uniform scale. |
| `rotation` | `Vector` | no transform | Euler rotation in radians. |

Returns: `Geometry`.

```python
geo = cube(size=1.0)
vec_1 = vector(0, 0, 1)
vec_2 = vector(2, 1, 0.5)
geo = transform(geo, translation=vec_1, scale=vec_2)
output('Geometry', geo)
```

### `polyline(points)`

Creates curve geometry from a compile-time list of vector points.

| Parameter | Type | Description |
| --- | --- | --- |
| `points` | compile-time list of vectors | Ordered points for the polyline. |

Returns: `Geometry`.

```python
vec_1 = vector(-1, 0, 0)
vec_2 = vector(0, 1, 0)
vec_3 = vector(1, 0, 0)
shape = polyline([vec_1, vec_2, vec_3])
output('Geometry', shape)
```

## Instancing

### `instance_on_points(instance, points, selection=True, scale=None, rotation=None, realize=True)`

Instances one geometry value on point geometry.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `instance` | `Geometry` | required | Geometry to instance. |
| `points` | `Geometry` | required | Point geometry receiving the instances. |
| `selection` | `Bool` | `True` | Points on which to create instances. |
| `scale` | value accepted by the underlying instance helper | Blender default | Optional instance scale. |
| `rotation` | value accepted by the underlying instance helper | Blender default | Optional instance rotation. |
| `realize` | compile-time `Bool` | `True` | When true, realizes instances before returning. |

Returns: `Geometry`.

```python
pts = points(12)
value_1 = index()
vec_2 = vector(value_1 * 0.3, 0, 0)
pts = set_position(pts, vec_2)
geo_3 = cube(size=0.15)
vec_4 = vector(1, 1, 1)
selection = index() % 2 == 0
geo = instance_on_points(geo_3, pts, selection=selection, scale=vec_4, realize=True)
output('Geometry', geo)
```

### `realize_instances(geometry)`

Realizes instances on geometry.

| Parameter | Type | Description |
| --- | --- | --- |
| `geometry` | `Geometry` | Geometry containing instances. |

Returns: `Geometry`.

```python
pts = points(6)
geo_1 = cube(size=0.2)
instanced = instance_on_points(geo_1, pts, realize=False)
geo_2 = realize_instances(instanced)
output('Geometry', geo_2)
```

## Raw Blender node construction

### `node(bl_idname, props={...}, inputs={...}, output=..., typ=...)`
### `node(bl_idname, props={...}, inputs={...}, outputs={...})`

Creates a Blender node directly. Use this for Blender node types that are not exposed through a dedicated NodeForge built-in. The dictionary expressions shown here are special compile-time syntax accepted only in arguments to `node(...)`; general DSL dictionaries remain unsupported.

`bl_idname`, node properties, socket selectors, output aliases, and type tokens are compile-time declarations. Runtime values are allowed inside `inputs={...}` and are linked to the selected Blender input sockets. `props=` is applied before socket selectors are resolved, so selectors see the node configuration produced by those properties.

Supported type tokens are `Float`, `Int`, `Bool`, `Vector`, `Geometry`, `Material`, `Object`, `String`, and `Bundle`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `bl_idname` | compile-time `String` | required | Blender node type identifier, for example `"ShaderNodeMath"`. |
| `props` | literal dict | `{}` | Blender node properties assigned before socket resolution. Custom properties are not supported. |
| `inputs` | literal dict | `{}` | Maps input socket selectors to literal defaults, runtime values, or a literal list for multi-input fanout. |
| `output` | socket selector | required in single-output mode | Output socket returned by the call. |
| `typ` | type token | required in single-output mode | NodeForge type of `output`. |
| `outputs` | literal dict | required in multi-output mode | Maps NodeForge result aliases to type tokens or explicit `(selector, TypeToken)` pairs. |

Returns: one runtime value in single-output mode; a named multi-output result in `outputs=` mode.

### Socket selectors

A raw-node socket selector has one of three forms:

| Selector | Meaning |
| --- | --- |
| `"Name"` | Exact Blender `socket.name`. The name must identify exactly one addressable socket. |
| `0`, `1`, ... | Zero-based ordinal among addressable sockets after `props=` is applied. Use this when Blender exposes duplicate display names. |
| `ID("identifier")` | Exact Blender `socket.identifier`. Use this when stable identifier-level addressing is required. |

Addressable sockets are sockets in the requested direction excluding virtual and unavailable sockets. Positional selectors count only this filtered list; they are not raw indices into `node.inputs` or `node.outputs`.

`ID(...)` has special meaning only in a socket-selector position inside `node(...)`. It does not reserve the name `ID` elsewhere in the DSL. A plain string never means `socket.identifier`.

```python
value = node(
    'ShaderNodeMath',
    props={'operation': 'ADD'},
    inputs={0: 1.0, 1: 2.0},
    output='Value',
    typ=Float,
)
output('Value', value)
```

For nodes with known Blender identifiers, the same sockets can be selected explicitly:

```python
value = node(
    'ShaderNodeMath',
    props={'operation': 'ADD'},
    inputs={ID('Value'): 1.0, ID('Value_001'): 2.0},
    output=ID('Value'),
    typ=Float,
)
output('Value', value)
```

### Multiple outputs

The shorthand `outputs={'X': Float}` uses the alias `X` as both the NodeForge result name and the Blender socket-name selector. Access the result as `.X` or `['X']`.

```python
separate = node(
    'ShaderNodeSeparateXYZ',
    inputs={'Vector': position()},
    outputs={'X': Float, 'Y': Float, 'Z': Float},
)
output('Height', separate.Z)
```

When the public result alias should differ from the Blender socket selector, use `(selector, TypeToken)`:

```python
parts = node(
    'ShaderNodeSeparateXYZ',
    inputs={'Vector': position()},
    outputs={
        'left': (0, Float),
        'middle': (ID('Y'), Float),
    },
)
output('Left', parts.left)
output('Middle', parts.middle)
```

Raw-node declarations are validated before defaults, links, or result bindings are applied. Newly generated raw-node metadata stores Blender socket identifiers so later updates can fail closed if the physical node schema no longer matches the declaration.

## Runtime loops

Runtime loops compile to Blender Repeat Zones. Use `repeat_range(...)` when the iteration count or loop state must be represented in the generated Geometry Nodes graph. Use compile-time `range(...)` for fixed authoring loops that should be unrolled during compilation.

### `for i in repeat_range(steps): ...`

Updates existing Geometry, Vector, Float, Int, Bool, and Bundle variables inside one Repeat Zone. A carried value must keep exactly the same type on every assignment throughout the loop. The body supports assignments, `geometry_builder` method statements, nested `if` blocks, and nested `repeat_range(...)` loops. It must update at least one existing variable or builder state. Assigned names that did not exist before the loop are iteration-local temporaries rather than Repeat Zone state items.

State item order follows first assignment in the loop body, including nested branches. Conditional branches preserve omitted state values and merge changed state through Switch nodes. The loop index name cannot also be state, and state names cannot collide with Repeat Zone system socket names such as `Iterations` or `Iteration`.

| Part | Type | Description |
| --- | --- | --- |
| `steps` | `Int` | Repeat count. May be an integer literal, compile-time integer, or runtime `Int` value. |
| state variables | `Geometry`, `Vector`, `Float`, `Int`, `Bool`, or `Bundle` | Existing variables assigned inside the loop. |

```python
n = input_int('N', default=8)
geo = cube(0.1)
pos = vector(0, 0, 0)
angle = input_float('Angle', default=45)
for i in repeat_range(n):
    next_pos = pos + vector(1, 0, 0)
    part = transform(cube(0.05), translation=next_pos)
    geo = join(geo, part)
    pos = next_pos
    angle = -angle
output('Geometry', geo)
output('Position', pos)
output('Angle', angle)
```

`range(...)` does not create Repeat Zones. A non-constant `range(...)` count raises `CompileError`; use `repeat_range(...)` for runtime repetition.

## Lists, tuples, and unrolled loops

Script lists and tuples are compile-time collections of compiled values. They are useful for building a fixed list of geometry parts and then passing the list to `join(...)`.

```python
parts = []
vec_1 = vector(-1, 0, 0)
vec_2 = vector(0, 0, 0)
vec_3 = vector(1, 0, 0)
for offset in [vec_1, vec_2, vec_3]:
    geo_4 = cube(size=0.25)
    geo_5 = transform(geo_4, translation=offset)
    parts.append(geo_5)
geo_6 = join(parts)
output('Geometry', geo_6)
```

Lists and tuples cannot be output directly and cannot be used as runtime socket values.
