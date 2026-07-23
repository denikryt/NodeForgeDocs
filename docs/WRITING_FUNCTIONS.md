# Writing Functions

A reusable function is a DSL script whose inputs become function parameters and whose output becomes the returned value. Save the script to **Local** to use it from other scripts.

## Write the Function

Create the function in a Blender Text datablock. Use `input_*` methods for parameters and `output(...)` for the result.

```python
geometry = input_geometry('Geometry')
translation = input_vector('Translation', default=vector(0, 0, 1))

result = transform(geometry, translation=translation)
output('Geometry', result)
```

The input socket names define the parameters exposed by the function. A function used as an expression has one output.

## Save to Local

Open **Library → Local**, then click **Save to Local**.

Choose **Text Block** as the source, enter a name, and select a folder when needed. The saved name becomes the function name used by imports. Use letters, digits, and underscores, and begin the name with a letter.

Click **Refresh** in the **Local** section after saving.

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

Load the saved function into a Text datablock, edit the source, and save it to **Local** again using the same name. Refresh the **Local** section before adding or compiling the updated function.
