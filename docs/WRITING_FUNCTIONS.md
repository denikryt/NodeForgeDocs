# Writing Functions

A reusable function is a DSL script whose inputs become function parameters and whose outputs become its returned values. Save the script to **Local** to use it from other scripts.

## Write the Function

Create the function in a Blender Text datablock. Use `input_*` methods for parameters and `output(...)` for the result.

```python
geometry = input_geometry('Geometry')
translation = input_vector('Translation', default=vector(0, 0, 1))

result = transform(geometry, translation=translation)
output('Geometry', result)
```

The input socket labels define the public parameter names shown in Blender. Call arguments can always be positional. A unique label can also be used as a keyword: NodeForge normalizes it with Unicode case folding and ignores punctuation, spaces, and underscores for keyword matching. For example, `Scale XY` can be called as `scale_xy=...`, and a Unicode label such as `Привет Мир` remains keyword-addressable.

Each `input_*()` call creates a separate parameter. The label is display text, not declaration identity. If two inputs normalize to the same keyword (including duplicate labels), that keyword is ambiguous and the function must be called positionally for those inputs. A label containing no letters or digits has no keyword alias and is positional-only.

## Return Multiple Values

Declare more than one output when the caller needs separate values.

```python
geometry = input_geometry('Geometry')
value = input_float('Value', default=1.0)

result = transform(geometry, scale=value)
output('Geometry', result)
output('Scaled Value', value * 2.0)
```

Unpack the results in the same order as the output declarations.

```python
from local import scale_and_measure

geometry = input_geometry('Geometry')
result, scaled_value = scale_and_measure(geometry, 2.0)
output('Geometry', result)
output('Scaled Value', scaled_value)
```

You can also store the fixed result and select an output with a compile-time integer index, including a negative index.

## Save to Local

Open **Library → Local**, browse to a NodeForge-managed folder, then click **Save**.

Enter a name for the selected Text datablock. The saved name becomes the function name used by imports. Use letters, digits, and underscores, and begin the name with a letter. Enable **Overwrite** when replacing an existing function.

You can also use **Add Folder...** to register an existing directory of `.nf` files. External Local folders are read-only in NodeForge; edit those files with your normal editor.

## Use the Function

Import the saved script from `local` and call it with the inputs declared by the function.

```python
from local import move_geometry

geometry = input_geometry('Geometry')
result = move_geometry(geometry, translation=vector(0, 0, 2))
output('Geometry', result)
```

See [Imports](SYNTAX.md#imports) for aliases, star imports, and name-resolution rules.

## Update the Function

For a managed Local function, load or open its source in a Text datablock, edit it, then save it to the same Local path with **Overwrite** enabled. For an external Local function, edit its `.nf` file on disk.

Existing generated groups keep the dependency source used when they were compiled. Compile a new caller to use the current function source, or select an existing library-backed group node and click **Reload from Source** to rebuild that group in place. Compatible links and input values are preserved during the reload.
