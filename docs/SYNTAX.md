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
| `Int` | Integer scalar or field. |
| `Bool` | Boolean scalar or field. |
| `Vector` | Three-component vector or field. |

### Compile-Time Types

| Type | Description |
| --- | --- |
| Integer | Whole number. |
| Float | Floating-point number. |
| Boolean | `True` or `False`. |
| String | Text value. |
| `None` | Absence of a value. |
| Vector | Three-component vector constant. |
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

Strings are compile-time values used for names and identifiers required while the script is compiled.

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

## Comments

A comment starts with `#` and continues to the end of the line.

```python
# Store the initial size.
size = 2.0
```

## Group Inputs and Outputs

Group sockets can be declared explicitly or inferred from name usage.

### Explicit Inputs

An `input_*` call declares an input socket.

```python
width = input_float('Width', default=2.0)
```

### Implicit Inputs

A name read before assignment becomes an implicit group input when it does not resolve to another declared name.

```python
geometry = cube(size=width)
```

Here, `width` becomes a group input.

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

A final assignment uses its variable name. A final expression uses `out`.

## Reusable Functions

A reusable function is a saved DSL script whose group inputs become call parameters and whose output becomes the returned value.

```python
size = input_float('Size', default=1.0)
geometry = cube(size=size)
output('Geometry', geometry)
```

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

Names beginning with `_` are private and excluded from imports. Plain imports, relative imports, block-local imports, and imports from arbitrary Python modules are unsupported.

## Local Functions

A script-local function is declared with `def` and returns one value with `return`.

```python
def double(value):
    result = value * 2.0
    return result

x = input_float('X', default=1.0)
y = double(x)
output('Y', y)
```

Parameters are names without default values. A local function must contain a return statement. Nested function declarations are unsupported.

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

A runtime top-level `if` requires an `else` branch, and values assigned in both branches must have compatible types.

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

Values assigned before the loop and changed inside it become repeat state items. Use `range(...)` for compile-time iteration and `repeat_range(...)` for runtime iteration.

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

NodeForge does not support regular `import` statements, `lambda`, `while`, `try`, `class`, decorators, comprehensions, dictionaries, sets, `with`, `yield`, generators, multiple assignment targets, or top-level `return`, `del`, `global`, and `nonlocal` statements.
