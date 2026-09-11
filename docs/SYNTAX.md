# DSL Syntax and Semantics

NodeForge uses a Python-like domain-specific language for building Blender Geometry Nodes groups.

## Names and Assignments

An assignment binds a name to a value.

```python
size = 2.0
```

The assigned name can be used in later statements.

```python
width = 2.0
height = width
```

Assigning another value to the same name replaces the previous value.

```python
size = 1.0
size = 2.0
```

An assignment target must be a single name.

## Literals

NodeForge supports integer, floating-point, Boolean, string, and `None` literals.

```python
count = 4
scale = 0.5
enabled = True
name = 'Height'
missing = None
```

Strings can use single or double quotes.

## Value Types

NodeForge supports runtime socket values and compile-time values.

### Runtime Types

| Type | Description |
| --- | --- |
| `Geometry` | Geometry socket value. |
| `Float` | Floating-point scalar or field. |
| `Int` | Integer scalar or field. Some operations accept it as a numeric field or return a `Float`; the called operation defines the result type. |
| `Bool` | Boolean scalar or field. |
| `Vector` | Three-component vector or field. |
| `Material` | Blender material socket value. |
| `Object` | Blender object socket value with Object Info properties. |
| `String` | Blender string socket value. |
| `Bundle` | Blender bundle socket containing named runtime values. |

### Compile-Time Types

| Type | Description |
| --- | --- |
| Integer | Whole number. |
| Float | Floating-point number. |
| Boolean | `True` or `False`. |
| String | Text value. |
| `None` | Absence of a value. |
| Vector | Three-component value. `vector(...)` is compile-time only when all arguments are compile-time; runtime arguments produce a runtime `Vector`. |
| List | Ordered mutable collection. |
| Tuple | Ordered immutable collection. |

## Function Calls

A function call uses a function name followed by parentheses.

```python
direction = vector(1, 0, 0)
```

Arguments can be positional or named.

```python
size = input_float('Size', default=2.0)
```

The returned value can be assigned to a name.

```python
geometry = cube(size=2.0)
```

Calls can be nested.

```python
geometry = transform(cube(size=2.0), translation=vector(0, 0, 1))
```

Assigning intermediate values is usually easier to read.

```python
base = cube(size=2.0)
offset = vector(0, 0, 1)
geometry = transform(base, translation=offset)
```

Only simple function names can be called. Supported method calls are documented in the sections that introduce them.

## Operators and Expressions

Expressions combine values and produce a result.

| Syntax | Meaning |
| --- | --- |
| `a + b` | Addition. |
| `a - b` | Subtraction. |
| `a * b` | Multiplication. |
| `a / b` | Division. |
| `a ** b` | Power. |
| `a % b` | Modulo. |
| `-a`, `+a` | Unary sign. |
| `not a` | Boolean NOT. |
| `a and b`, `a or b` | Boolean AND and OR. |
| `a < b`, `a <= b`, `a > b`, `a >= b`, `a == b`, `a != b` | Comparisons. |
| `x if condition else y` | Conditional expression. |

```python
width = 2.0
height = 3.0
area = width * height
is_large = area > 5.0
result = area if is_large else 0.0
```

Operations are type-checked. The operands must support the selected operation.

## Compile-Time and Runtime Values

A compile-time value is resolved while the script is compiled.

```python
socket_name = 'Height'
default_height = 1.0
height = input_float(socket_name, default=default_height)
```

A runtime value is represented in the generated node group and can change when Blender evaluates that group.

```python
height = input_float('Height', default=1.0)
scaled_height = height * 2.0
```

The operation receiving a value determines whether it must be available at compile time or can remain a runtime value.

Compile-time helper calls are documented in [Core DSL Built-ins](BUILTINS.md#compile-time-helper-calls).

## Strings and F-Strings

String literals are compile-time values when NodeForge needs a name, identifier, or option while compiling the script.

```python
name = 'Height'
height = input_float(name, default=1.0)
```

Compile-time strings can be concatenated.

```python
prefix = 'Generated'
input_name = prefix + ' Value'
value = input_float(input_name, default=1.0)
```

F-strings are supported when every interpolated value is a compile-time string.

```python
prefix = 'Generated'
input_name = f'{prefix} Value'
value = input_float(input_name, default=1.0)
```

Runtime interpolation, conversion flags, and format specifications are unsupported.

`input_string(...)` creates a runtime `String` socket. Runtime strings can be passed to operations that accept a String field, such as the attribute name in `store_named_attribute(...)` and Bundle paths. A runtime string cannot be used where a compile-time name or option is required.

## Lists and Tuples

Lists and tuples group values at script level.

```python
sizes = [0.5, 1.0, 2.0]
offset = (1.0, 0.0, 0.0)
```

They can also contain runtime values.

```python
first = cube(size=1.0)
second = cube(size=2.0)
parts = [first, second]
geometry = join(parts)
```

A list can be extended with `append(...)`.

```python
parts = []
parts.append(cube(size=1.0))
parts.append(cube(size=2.0))
geometry = join(parts)
```

Lists and tuples cannot be exposed directly as group sockets.

Functions with multiple outputs return a fixed tuple-like result. Unpack it into a flat list or tuple of names, or select an element with a compile-time integer index.

```python
geo, captured = capture_attribute(cube(), position().z)
```

## Attribute Access and Indexing

Vector components are available through `.x`, `.y`, and `.z`.

```python
direction = vector(1, 2, 3)
x = direction.x
```

Vectors also support compile-time integer indices `0`, `1`, and `2`.

```python
direction = vector(1, 2, 3)
y = direction[1]
```

Lists and tuples support compile-time integer indexing.

```python
values = [0.5, 1.0, 2.0]
size = values[1]
```

A runtime value cannot be used as an index.

`Object` values provide `.geometry`, `.location`, `.rotation`, and `.scale`. Configure their Object Info node with `.info(...)` before the first property access. See [Object Values](BUILTINS.md#object-values).

## Comments

A comment starts with `#` and continues to the end of the line.

```python
# Store the initial size.
size = 2.0
```

## Group Inputs and Outputs

Group sockets can be declared explicitly or inferred from name usage.

### Explicit Inputs

An `input_*` call declares an input socket. It must be the complete right-hand side of a simple assignment.

```python
width = input_float('Width', default=2.0)
```

Every declaration creates a new input socket. Its string argument is the displayed socket label, so separate declarations may use the same label and retain different defaults.

```python
first = input_float('Value', default=1.0)
second = input_float('Value', default=2.0)
output('Sum', first + second)
```

Call reusable groups with duplicate input labels positionally. A keyword cannot identify which duplicate label to use.

### Interface Panels

Use `panel(...)` to organize previously declared inputs in Blender's node-group interface.

```python
radius = input_float('Radius', default=1.0)
segments = input_int('Segments', default=32)
panel([radius, segments], name='Shape', collapsed=True)
```

Panels are standalone top-level declarations. Their members must be group input variable names, and each input can belong to one panel. See [`panel(...)`](BUILTINS.md#interface-panels) for the complete reference.

### Implicit Inputs

A name read before assignment becomes an implicit group input when it does not resolve to another declared name.

```python
geometry = cube(size=width)
```

Here, `width` becomes an implicit group input. Its type must be inferable from the parameter position where it is used. If the use site does not provide one unambiguous expected socket type, compilation fails; use an explicit `input_*` declaration instead.

### Explicit Outputs

The `output(...)` call declares an output socket.

```python
geometry = cube(size=2.0)
output('Geometry', geometry)
```

### Implicit Final Output

The final top-level assignment or expression can define one implicit output.

```python
geometry = cube(size=2.0)
geometry
```

A final assignment uses its variable name. A final expression uses `out`. The value must be a supported runtime output: `Geometry`, `Vector`, `Float`, `Int`, `Bool`, `Material`, `Object`, `String`, or `Bundle`. A statement without a resulting value cannot define an implicit output.

## Reusable Functions

A reusable function is a saved DSL script whose group inputs become call parameters. One output becomes the call result. Multiple outputs become a fixed result that can be unpacked or indexed.

```python
size = input_float('Size', default=1.0)
geometry = cube(size=size)
output('Geometry', geometry)
```

```python
from local import analyze_geometry

geometry, value = analyze_geometry(cube(size=1.0))
output('Geometry', geometry)
output('Value', value)
```

Reusable function calls share one backing function group by default. Add `__unique__=True` when one supported call needs its own shallow function-group copy, for example when you intend to edit a Curve Map node separately for that occurrence.

```python
def remap_curve(value: Float):
    return node(
        'ShaderNodeFloatCurve',
        inputs={'Factor': 1.0, 'Value': value},
        output='Value',
        typ=Float,
    )

value_a = input_float('A')
value_b = input_float('B')
first = remap_curve(value_a, __unique__=True)
second = remap_curve(value_b, __unique__=True)
```

`__unique__` must be a compile-time `Bool`. It is supported by script-local `def` calls and editable `.nf` functions imported from `functions` or `examples`. Omitting it or passing `__unique__=False` uses the shared group. Local catalog calls, core built-ins, systems, backend helpers, and `node(...)` use their normal call behavior.

The copy is shallow, so nested reusable dependencies remain shared unless their calls also use `__unique__=True`. A unique group's manual Blender-node state is preserved when an update leaves that function unchanged. Changing the function source or a reusable dependency used by it rebuilds the group from source.

See [Writing Functions](WRITING_FUNCTIONS.md) for creating and saving a local reusable function.

## Imports

Imports are allowed only at top level. Reusable scripts can be imported from `functions`, `examples`, or `local`.

```python
from functions import circle_points
from local import move_geometry
```

An alias changes the name used in the current script.

```python
from functions import circle_points as make_circle
```

A star import adds every public name from the selected catalog.

```python
from functions import *
```

Names beginning with `_` are private and excluded from imports. Public callable names must be unique across active package catalogs; conflicting names make the package inventory invalid. Plain imports, relative imports, block-local imports, and imports from arbitrary Python modules are unsupported.

## Local Functions

A script-local function is declared with `def` and returns one value or a flat tuple of values with `return`.

```python
def double(value):
    result = value * 2.0
    return result

x = input_float('X', default=1.0)
y = double(x)
output('Y', y)
```

Parameters are names without default values. Add a NodeForge type token annotation when a parameter's type cannot be inferred unambiguously from the call.

```python
def inspect(value: Float, pos: Vector):
    return value * 2.0, pos.z, value > 0.0

value = input_float('Value', default=1.0)
pos = input_vector('Position')
doubled, height, positive = inspect(value, pos)
output('Doubled', doubled)
output('Height', height)
output('Positive', positive)
```

Annotations support `Float`, `Int`, `Bool`, `Vector`, `Geometry`, `Material`, `Object`, `String`, and `Bundle`. An annotation constrains the generated function input socket; unannotated parameters use call-site type inference. A tuple return must be flat and non-empty. A local function must contain a return statement. Nested function declarations are unsupported.

The generated function call node uses a readable title derived from the function name. For example, `mix_biomes()` appears as **Mix Biomes**.

## If Statements

An `if` statement selects between two blocks.

```python
use_large_size = True
if use_large_size:
    size = 2.0
else:
    size = 1.0
```

A compile-time condition selects one branch while the script is compiled. A runtime `Bool` compiles both branches and merges their resulting values.

```python
use_large_size = input_bool('Large', default=False)
if use_large_size:
    size = 2.0
else:
    size = 1.0
geometry = cube(size=size)
output('Geometry', geometry)
```

A runtime top-level `if` requires an `else` branch. Values merged from both branches must have compatible types supported by Blender's Switch node: `Float`, `Int`, `Vector`, `Bool`, `Geometry`, `String`, or `Bundle`. Compile-time collections are not runtime branch-merge values.

## For Statements

A `for` statement over a compile-time list, tuple, or `range(...)` is unrolled while the script is compiled.

```python
parts = []
for size in [0.5, 1.0, 1.5]:
    parts.append(cube(size=size))
geometry = join(parts)
output('Geometry', geometry)
```

A loop target can be a single name or a list or tuple pattern that matches each compile-time item.

```python
positions = [(0, 0), (1, 0), (0, 1)]
parts = []
for x, y in positions:
    point = points(1)
    point = set_position(point, vector(x, y, 0))
    parts.append(point)
geometry = join(parts)
output('Geometry', geometry)
```

`repeat_range(...)` creates a runtime repeat loop.

```python
value = 0.0
steps = input_int('Steps', default=5)
for index in repeat_range(steps):
    value = value + 1.0
output('Value', value)
```

Values assigned before the loop and changed inside it become repeat state items. Repeat state is limited to existing `Geometry`, `Vector`, `Float`, `Int`, `Bool`, and `Bundle` values. The loop must update at least one existing state value; names first created inside the loop are local temporaries. Use `range(...)` for compile-time iteration and `repeat_range(...)` for runtime iteration.

`repeat_range(...)` loops can be nested. Each loop creates its own Repeat Zone and its own index value.

```python
value = 0.0
rows = input_int('Rows', default=3)
columns = input_int('Columns', default=4)
for row in repeat_range(rows):
    for column in repeat_range(columns):
        value = value + row + column
output('Value', value)
```

## Geometry Builder

`geometry_builder()` accumulates geometry. The current result is available through `.geometry`.

```python
builder = geometry_builder()
builder.add(cube(size=1.0))
builder.extend([cube(size=0.5), cube(size=0.25)])
output('Geometry', builder.geometry)
```

Builder methods can be used inside compile-time and runtime loops. A builder cannot be assigned to another name, inserted into a collection, passed to a function, or exposed as an output.

## Unsupported Python Syntax

NodeForge does not support regular `import` statements, `lambda`, `while`, `try`, `class`, decorators, comprehensions, general dictionary or set expressions, `with`, `yield`, generators, assignment expressions, multiple assignment targets, or top-level `return`, `del`, `global`, and `nonlocal` statements. `break`, `continue`, `raise`, `assert`, and async statements are also unsupported. Dictionary syntax is accepted only as the compile-time `props`, `inputs`, or `outputs` declaration passed directly to `node(...)`.
