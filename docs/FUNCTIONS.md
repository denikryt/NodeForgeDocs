# Math Functions

These reusable DSL functions are provided by the installed `nodeforge.math` package and are imported from `functions`.

The `functions` catalog of the Math library contains reusable functions written in the DSL. Import a function before calling it. See [Imports](SYNTAX.md#imports) for import syntax and name-resolution rules.

## Math Helpers

### `inverse_lerp(a=0.0, b=1.0, x=0.0)`

Computes the unbounded interpolation factor of `x` between `a` and `b`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `a` | `Float` | `0.0` | Range start. |
| `b` | `Float` | `1.0` | Range end. |
| `x` | `Float` | `0.0` | Value to measure inside the range. |

Returns: `Float`.

```python
from functions import inverse_lerp
value_1 = input_float('Value', default=4.0)
t = inverse_lerp(2.0, 6.0, value_1)
output('Factor', t)
```

### `remap(x=0.0, in_min=0.0, in_max=1.0, out_min=0.0, out_max=1.0)`

Maps `x` from one numeric range to another. The result is not clamped.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `x` | `Float` | `0.0` | Input value. |
| `in_min` | `Float` | `0.0` | Source range lower bound. |
| `in_max` | `Float` | `1.0` | Source range upper bound. |
| `out_min` | `Float` | `0.0` | Target range lower bound. |
| `out_max` | `Float` | `1.0` | Target range upper bound. |

Returns: `Float`.

```python
from functions import remap
count = input_int('Count', default=32)
pts = points(count)
value_1 = index()
z = remap(value_1, in_min=0.0, in_max=count - 1.0, out_min=-1.0, out_max=1.0)
value_2 = index()
vec_3 = vector(value_2 * 0.1, 0, z)
pts = set_position(pts, vec_3)
output('Geometry', pts)
```

### `saturate(x=0.0)`

Clamps `x` to the `0..1` range.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `x` | `Float` | `0.0` | Input value. |

Returns: `Float`.

```python
from functions import saturate
value = input_float('Value', default=1.25)
value_1 = saturate(value)
output('Clamped', value_1)
```

### `step(edge=0.0, x=0.0)`

Returns `1.0` when `x >= edge`; otherwise returns `0.0`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `edge` | `Float` | `0.0` | Threshold. |
| `x` | `Float` | `0.0` | Value to test. |

Returns: `Float`.

```python
from functions import step
value_1 = position()
value_2 = noise(value_1, scale=3.0)
mask_value = step(0.5, value_2)
output('Mask', mask_value)
```

### `smoothstep(edge0=0.0, edge1=1.0, x=0.0)`

Returns a clamped Hermite interpolation from `0` to `1` between `edge0` and `edge1`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `edge0` | `Float` | `0.0` | Lower transition edge. |
| `edge1` | `Float` | `1.0` | Upper transition edge. |
| `x` | `Float` | `0.0` | Input value. |

Returns: `Float`.

```python
from functions import smoothstep
pts = points(64)
value_1 = index()
t = smoothstep(0.0, 63.0, value_1)
value_2 = index()
vec_3 = vector(value_2 * 0.05, 0, t)
pts = set_position(pts, vec_3)
output('Geometry', pts)
```

### `smootherstep(edge0=0.0, edge1=1.0, x=0.0)`

Returns a clamped smoother interpolation from `0` to `1` between `edge0` and `edge1`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `edge0` | `Float` | `0.0` | Lower transition edge. |
| `edge1` | `Float` | `1.0` | Upper transition edge. |
| `x` | `Float` | `0.0` | Input value. |

Returns: `Float`.

```python
from functions import smootherstep
x = input_float('X', default=0.35)
value_1 = smootherstep(0.0, 1.0, x)
output('Value', value_1)
```

### `pingpong(x=0.0, length=1.0)`

Wraps `x` into a repeating triangular wave between `0` and `length`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `x` | `Float` | `0.0` | Input value. |
| `length` | `Float` | `1.0` | Peak value of the wave. |

Returns: `Float`.

```python
from functions import pingpong
pts = points(80)
value_1 = index()
z = pingpong(value_1 * 0.1, 1.0)
value_2 = index()
vec_3 = vector(value_2 * 0.05, 0, z)
pts = set_position(pts, vec_3)
output('Geometry', pts)
```

### `wrap(x=0.0, min=0.0, max=1.0)`

Wraps `x` into the half-open range `[min, max)` using modulo arithmetic.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `x` | `Float` | `0.0` | Input value. |
| `min` | `Float` | `0.0` | Lower bound. |
| `max` | `Float` | `1.0` | Upper bound. |

Returns: `Float`.

```python
from functions import wrap
value_1 = input_float('Angle', default=7.0)
angle = wrap(value_1, min=0.0, max=tau)
output('Angle', angle)
```

### `sign(x=0.0)`

Returns `-1.0` for negative values, `0.0` for zero, and `1.0` for positive values.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `x` | `Float` | `0.0` | Input value. |

Returns: `Float`.

```python
from functions import sign
x = input_float('X', default=-2.0)
value_1 = sign(x)
output('Direction', value_1)
```

## Vector Helpers

### `rotate2d(v=vector(1, 0, 0), angle=0.0)`

Rotates the XY components of `v` around the Z axis. The original Z component is preserved.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `v` | `Vector` | `vector(1, 0, 0)` | Input vector. |
| `angle` | `Float` | `0.0` | Rotation angle in radians. |

Returns: `Vector`.

```python
from functions import rotate2d
vec_1 = vector(1, 0, 0)
value_2 = radians(45)
v = rotate2d(vec_1, value_2)
output('Vector', v)
```

### `polar(radius=1.0, angle=0.0)`

Creates an XY vector from polar coordinates. Z is `0.0`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `radius` | `Float` | `1.0` | Distance from origin. |
| `angle` | `Float` | `0.0` | Angle in radians. |

Returns: `Vector`.

```python
from functions import polar
value_1 = radians(30)
pos = polar(radius=2.0, angle=value_1)
output('Vector', pos)
```

### `angle_between(a=vector(1, 0, 0), b=vector(0, 1, 0))`

Computes the angle between two vectors. Inputs are normalized internally and the dot product is clamped to `-1..1` before `acos(...)`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `a` | `Vector` | `vector(1, 0, 0)` | First vector. |
| `b` | `Vector` | `vector(0, 1, 0)` | Second vector. |

Returns: `Float`.

```python
from functions import angle_between
vec_1 = vector(1, 0, 0)
value_2 = position()
vec_3 = normalize(value_2)
angle = angle_between(vec_1, vec_3)
output('Angle', angle)
```

### `rotate_around_axis(v=vector(1, 0, 0), axis=vector(0, 0, 1), angle=0.0)`

Rotates `v` around `axis` using Rodrigues' rotation formula. `axis` is normalized internally.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `v` | `Vector` | `vector(1, 0, 0)` | Input vector. |
| `axis` | `Vector` | `vector(0, 0, 1)` | Rotation axis. |
| `angle` | `Float` | `0.0` | Rotation angle in radians. |

Returns: `Vector`.

```python
from functions import rotate_around_axis
vec_1 = vector(1, 0, 0)
vec_2 = vector(0, 1, 0)
value_3 = radians(90)
v = rotate_around_axis(vec_1, axis=vec_2, angle=value_3)
output('Vector', v)
```

## Point Creation and Layouts

Layout functions operate on point-domain geometry. They use the current point `index()` field and the explicit `count` parameter when present; they do not inspect the input geometry's point count.

### `layout_grid(geometry, count=vector(1, 1, 1), spacing=vector(1, 1, 1), centered=False)`

Places existing points in a 3D lattice.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Point geometry to position. |
| `count` | `Vector` | `vector(1, 1, 1)` | Lattice dimensions interpreted as `(count_x, count_y, count_z)`. Components are clamped to at least `1.0` for index math. |
| `spacing` | `Vector` | `vector(1, 1, 1)` | Per-axis spacing. Use `vector(s, s, s)` for uniform spacing. |
| `centered` | `Bool` | `False` | When true, positions are shifted by half of the layout extent so the lattice is centered around the origin. When false, positions use raw lattice coordinates from the origin. |

Returns: `Geometry`.

```python
from functions import layout_grid
pts = points(12)
vec_1 = vector(4, 3, 1)
vec_2 = vector(0.5, 0.5, 0.0)
pts = layout_grid(pts, count=vec_1, spacing=vec_2, centered=False)
output('Geometry', pts)
```

### `grid_points(count=vector(1, 1, 1), spacing=vector(1, 1, 1), centered=False)`

Creates points and places them with `layout_grid(...)`. The point count is `max(count.x, 0) * max(count.y, 0) * max(count.z, 0)`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `count` | `Vector` | `vector(1, 1, 1)` | Number of generated points per axis. Zero or negative components produce zero generated points for that component. |
| `spacing` | `Vector` | `vector(1, 1, 1)` | Per-axis spacing. |
| `centered` | `Bool` | `False` | Passed to `layout_grid(...)`. |

Returns: `Geometry`.

```python
from functions import grid_points
vec_1 = vector(5, 4, 1)
vec_2 = vector(0.5, 0.5, 0.0)
grid = grid_points(count=vec_1, spacing=vec_2, centered=False)
output('Geometry', grid)
```

### `layout_circle(geometry, count=16, radius=1.0, start_angle=0.0, end_angle=tau, include_endpoint=False)`

Places existing points on an XY circle or arc.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Point geometry to position. |
| `count` | `Int` | `16` | Explicit count used to compute the normalized index. |
| `radius` | `Float` | `1.0` | Circle or arc radius. |
| `start_angle` | `Float` | `0.0` | Start angle in radians. |
| `end_angle` | `Float` | `tau` | End angle in radians. |
| `include_endpoint` | `Bool` | `False` | When true, the last point reaches `end_angle`. For full circles the default avoids duplicating the first point. |

Returns: `Geometry`.

```python
from functions import layout_circle
pts = points(24)
pts = layout_circle(pts, count=24, radius=2.0, start_angle=0.0, end_angle=tau)
output('Geometry', pts)
```

### `circle_points(count=16, radius=1.0, start_angle=0.0, end_angle=tau, include_endpoint=False)`

Creates point geometry and places the points with the same circle/arc formula used by `layout_circle(...)`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `count` | `Int` | `16` | Number of generated points. Negative values are clamped to zero for point generation. |
| `radius` | `Float` | `1.0` | Circle or arc radius. |
| `start_angle` | `Float` | `0.0` | Start angle in radians. |
| `end_angle` | `Float` | `tau` | End angle in radians. |
| `include_endpoint` | `Bool` | `False` | When true, the last point reaches `end_angle`. |

Returns: `Geometry`.

```python
from functions import circle_points
arc = circle_points(count=16, radius=2.0, start_angle=0.0, end_angle=pi, include_endpoint=True)
output('Geometry', arc)
```

### `layout_spiral(geometry, count=16, radius=1.0, turns=1.0, height=0.0, start_radius=0.0, start_angle=0.0)`

Places existing points along a radial spiral in the XY plane, with optional Z height.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Point geometry to position. |
| `count` | `Int` | `16` | Explicit count used to compute the normalized index. |
| `radius` | `Float` | `1.0` | Final radius. |
| `turns` | `Float` | `1.0` | Number of full revolutions. |
| `height` | `Float` | `0.0` | Final Z height. |
| `start_radius` | `Float` | `0.0` | Initial radius. |
| `start_angle` | `Float` | `0.0` | Initial angle in radians. |

Returns: `Geometry`.

```python
from functions import layout_spiral
pts = points(96)
pts = layout_spiral(pts, count=96, radius=3.0, turns=4.0, height=1.5)
output('Geometry', pts)
```

### `spiral_points(count=16, radius=1.0, turns=1.0, height=0.0, start_radius=0.0, start_angle=0.0)`

Creates point geometry and places the points with `layout_spiral(...)`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `count` | `Int` | `16` | Number of generated points. Negative values are clamped to zero for point generation. |
| `radius` | `Float` | `1.0` | Final radius. |
| `turns` | `Float` | `1.0` | Number of full revolutions. |
| `height` | `Float` | `0.0` | Final Z height. |
| `start_radius` | `Float` | `0.0` | Initial radius. |
| `start_angle` | `Float` | `0.0` | Initial angle in radians. |

Returns: `Geometry`.

```python
from functions import spiral_points
spiral = spiral_points(count=128, radius=3.0, turns=5.0, height=2.0, start_radius=0.25)
output('Geometry', spiral)
```

### `layout_random(geometry, min=vector(-1, -1, -1), max=vector(1, 1, 1), seed=0)`

Places existing points at random positions between vector bounds. The point index is used as the random ID.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Point geometry to position. |
| `min` | `Vector` | `vector(-1, -1, -1)` | Lower random bound. |
| `max` | `Vector` | `vector(1, 1, 1)` | Upper random bound. |
| `seed` | `Int` | `0` | Random seed. |

Returns: `Geometry`.

```python
from functions import layout_random
pts = points(50)
vec_1 = vector(-2, -2, 0)
vec_2 = vector(2, 2, 1)
pts = layout_random(pts, min=vec_1, max=vec_2, seed=7)
output('Geometry', pts)
```

### `random_points(count=16, min=vector(-1, -1, -1), max=vector(1, 1, 1), seed=0)`

Creates point geometry and places the points with `layout_random(...)`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `count` | `Int` | `16` | Number of generated points. Negative values are clamped to zero for point generation. |
| `min` | `Vector` | `vector(-1, -1, -1)` | Lower random bound. |
| `max` | `Vector` | `vector(1, 1, 1)` | Upper random bound. |
| `seed` | `Int` | `0` | Random seed. |

Returns: `Geometry`.

```python
from functions import random_points
vec_1 = vector(-3, -3, 0)
vec_2 = vector(3, 3, 2)
pts = random_points(count=100, min=vec_1, max=vec_2, seed=42)
output('Geometry', pts)
```

## Geometry Helper

### `copy_by_offsets(geometry, scale=vector(1/3, 1/3, 1))`

Duplicates input geometry into the eight cells around the center of a 3x3 grid. This is the offset pattern used for Sierpinski-carpet style subdivision.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `geometry` | `Geometry` | required | Source geometry. |
| `scale` | `Vector` | `vector(1/3, 1/3, 1)` | Per-copy scale. |

Returns: `Geometry`.

```python
from functions import copy_by_offsets
geo = cube(size=1.0)
vec_1 = vector(1 / 3, 1 / 3, 1 / 3)
geo = copy_by_offsets(geo, scale=vec_1)
output('Geometry', geo)
```

## Fractal Helper

### `sierpinski_carpet(geometry, steps=2, scale=vector(1/3, 1/3, 1))`

Repeatedly applies `copy_by_offsets` to create a Sierpinski-carpet-style arrangement. `steps` is a runtime `Int`; non-positive values perform no repeat iterations.

```python
from functions import sierpinski_carpet

source = cube(size=1.0)
geometry = sierpinski_carpet(source, steps=2)
output('Geometry', geometry)
```

## Sequence Helper

### `fibonacci(n=8)`

Computes the Fibonacci sequence with a runtime Repeat Zone. `F(0)=0`, `F(1)=1`, and `F(n)=F(n-1)+F(n-2)`.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `n` | `Int` | `8` | Iteration count. |

Returns: `Int` or numeric value compatible with the Repeat Zone state.

```python
from functions import fibonacci
n = input_int('N', default=10)
value = fibonacci(n)
output('Value', value)
```

For complete scripts included with the package, see [Math Examples](MATH_EXAMPLES.md).
