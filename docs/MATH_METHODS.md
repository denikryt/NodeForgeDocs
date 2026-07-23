# Math Methods

These callables are provided by the `nodeforge.math` package, not by the Core DSL. Install that package explicitly before using them.

The Math library adds scalar and procedural methods to the DSL. These methods are called directly in expressions.

```python
angle = input_float('Angle', default=0.0)
value = sin(angle)
output('Value', value)
```

## Scalar Math Callables

These names are available directly when the Math library is installed. They accept runtime values and create the corresponding Geometry Nodes operations. When all arguments are compile-time numbers, the compiler can evaluate supported calls during compilation.

### Unary Callables

| Function | Description |
| --- | --- |
| `sin(value)` | Sine. |
| `cos(value)` | Cosine. |
| `tan(value)` | Tangent. |
| `asin(value)` | Arcsine. |
| `acos(value)` | Arccosine. |
| `atan(value)` | Arctangent. |
| `sqrt(value)` | Square root. |
| `abs(value)` | Absolute value. |
| `floor(value)` | Floor. |
| `ceil(value)` | Ceiling. |
| `round(value)` | Rounded value. |
| `fract(value)` | Fractional component. |
| `radians(value)` | Converts degrees to radians. |
| `degrees(value)` | Converts radians to degrees. |
| `exp(value)` | Exponential. |
| `ln(value)` | Natural logarithm. |

### Binary Callables

| Function | Description |
| --- | --- |
| `min(a, b)` | Minimum value. |
| `max(a, b)` | Maximum value. |
| `pow(a, b)` | Raises `a` to power `b`. |
| `log(a, b)` | Logarithm of `a` with base `b`. |
| `atan2(a, b)` | Two-argument arctangent. |
| `mod(a, b)` | Modulo. |

### Range and Selection Callables

| Function | Description |
| --- | --- |
| `clamp(value, min, max)` | Clamps a value to a range. |
| `mix(a, b, factor)` | Interpolates between two values. |
| `select(cond, true, false)` | Selects one value from a runtime boolean condition. |
| `map_range(value, from_min, from_max, to_min, to_max)` | Maps a value between ranges. |

```python
value = input_float('Value', default=0.25)
angle = radians(value * 360.0)
wave = sin(angle)
result = clamp(wave, -0.5, 0.5)
output('Value', result)
```

## Procedural Value Callables

### `noise(vector=position(), scale=..., detail=..., roughness=..., lacunarity=..., distortion=..., normalize=...)`

Creates a 3D Noise Texture field and returns its Factor output.

| Parameter | Type | Description |
| --- | --- | --- |
| `vector` | `Vector` | Coordinate field. Defaults to `position()`. |
| `scale` | numeric | Noise scale. |
| `detail` | numeric | Noise detail. |
| `roughness` | numeric | Noise roughness. |
| `lacunarity` | numeric | Noise lacunarity. |
| `distortion` | numeric | Noise distortion. |
| `normalize` | compile-time `Bool` | Sets the node normalization option. |

Returns: `Float`.

### `random_value()`
### `random_value(min, max, seed=..., id=...)`

Creates a Random Value field. With no bounds it returns a `Float` in the default range. Numeric bounds return `Float`; vector bounds return `Vector`.

| Parameter | Type | Description |
| --- | --- | --- |
| `min` | `Float` or `Vector` | Lower bound. |
| `max` | same as `min` | Upper bound. |
| `seed` | `Int` | Random seed. |
| `id` | `Int` | Per-element identifier. |

```python
pts = points(100)
pos = random_value(vector(-2, -2, 0), vector(2, 2, 1), seed=12, id=index())
pts = set_position(pts, pos)
output('Geometry', pts)
```
