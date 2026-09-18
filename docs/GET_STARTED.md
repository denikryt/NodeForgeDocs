# Get Started with NodeForge

NodeForge compiles a Python-like script into a native Blender Geometry Nodes group. This page takes you from installing the add-on to compiling and updating your first generated node group.

## Install NodeForge

1. Open the [NodeForge releases page](https://github.com/denikryt/NodeForge/releases).
2. In the latest release, download the `NodeForge-v<version>-blender.zip` file from **Assets**.
3. In Blender, open **Edit → Preferences → Add-ons**.
4. Open the Add-ons menu and choose **Install from Disk...**.
5. Select the downloaded ZIP file.
6. Enable **NodeForge** in the add-on list.

The installed add-on adds a **NodeForge** tab to the sidebar of the Geometry Node Editor.

## Create your first NodeForge group

1. Select a mesh object.
2. Open the **Geometry Node Editor** and click **New** to create a Geometry Nodes modifier and node tree.
3. Press `N` to open the sidebar, then select the **NodeForge** tab.
4. Open a **Text Editor** in another Blender area and click **New** to create a Text datablock.
5. Enter this script:

```python
geo = cube(size=2.0)
output("Geometry", geo)
```

6. In the NodeForge sidebar, select the new Text datablock in **Text Script**.
7. Click **Compile Script**. NodeForge compiles the text and adds the generated group node to the current Geometry Nodes tree.
8. Connect the generated node **Geometry** output to the **Group Output** node **Geometry** input.

The object now displays the cube produced by the script.

## Update the generated group

1. Edit the source in the Text Editor. For example, change `size=2.0` to `size=3.0`.
2. Select the generated NodeForge group node in the Geometry Node Editor.
3. Click **Update Selected NodeGroup** in the NodeForge sidebar.

NodeForge recompiles the selected text into the same node group. The group name, compatible input values, and compatible links are preserved.

## Main actions

| Button | What it does |
| --- | --- |
| **Compile Script** | Compiles the selected Text datablock into a new Geometry Nodes group and inserts it into the active Geometry Nodes editor. |
| **Update Selected NodeGroup** | Recompiles the selected Text datablock into the selected generated group. Compatible input values and links are preserved. |
| **Load Script From Selected NodeGroup** | Copies the source embedded in the selected generated group into the selected Text datablock. |
| **Reload from Source** | Rebuilds a selected Local or package-backed group from its current catalog source. |

Compiled Geometry Nodes groups remain in the Blender file and continue evaluating when the NodeForge add-on is disabled or uninstalled.

## First script: one input, one output

Start with the smallest useful node group: one float input and one float output.

```python
value = input_float('Value', default=1.0)
output('Value', value)
```

What this means:

| Line | Meaning |
| --- | --- |
| `input_float('Value', default=1.0)` | Creates a float input socket named `Value` with default value `1.0`. |
| `value = ...` | Stores that socket value in the script variable `value`. |
| `output('Value', value)` | Creates an output socket named `Value` and connects the value to it. |

After compiling, the generated node group has one input socket and one output socket. This is not visually exciting, but it shows the core model: **inputs create node-group inputs, outputs create node-group outputs**.

## First geometry script: adjustable cube

Now create geometry. This script makes a cube whose size is controlled by a group input.

```python
size = input_float('Size', default=2.0)
geo = cube(size=size)
output('Geometry', geo)
```

The generated group has:

| Socket | Direction | Type |
| --- | --- | --- |
| `Size` | Input | Float |
| `Geometry` | Output | Geometry |

The important difference from normal Python is that `geo` is not a mesh object in Python memory. It is a Geometry Nodes value being built by the compiler.

## Use the object's input geometry

A compiled NodeForge group can process geometry supplied by the surrounding Geometry Nodes tree. Declare a Geometry input in the script, then connect the surrounding tree's geometry to that socket on the generated group node. The generated NodeForge group is a nested node group; it is not itself the modifier's root node tree.

```python
geo = input_geometry('Geometry')
scale = vector(1, 1, 2)
result = transform(geo, scale=scale)
output('Geometry', result)
```

After compiling, connect the object's geometry to the generated group's `Geometry` input if it is not connected automatically. In a Geometry Nodes modifier, this is the socket that carries the mesh or curve from the object in the scene.

Use this when you want to modify an existing object instead of generating all geometry from scratch with functions like `cube(...)`.

## Add variables and transforms

Use variables to keep scripts readable. The following script creates a cube and scales it in Z.

```python
size = input_float('Size', default=1.0)
height = input_float('Height', default=3.0)

base = cube(size=size)
scale = vector(1, 1, height)
geo = transform(base, scale=scale)

output('Geometry', geo)
```

This pattern is usually clearer than nesting calls directly inside other calls. Write each meaningful value once, name it, and pass the name to the next operation.

## Build a small shape with a loop

Lists and compile-time `for` loops are useful for generating repeated geometry. This example creates a row of five cubes.

```python
spacing = input_float('Spacing', default=1.25)
count = 5

parts = []
for i in range(count):
    base = cube(size=1.0)
    offset = vector(i * spacing, 0, 0)
    moved = transform(base, translation=offset)
    parts.append(moved)

geo = join(parts)
output('Geometry', geo)
```

What is happening:

| Part | Meaning |
| --- | --- |
| `count = 5` | A compile-time value. NodeForge knows it while compiling. |
| `range(count)` | Unrolls the loop during compilation. |
| `parts = []` / `parts.append(...)` | Collects Geometry values into a compile-time list. |
| `join(parts)` | Combines the generated cubes into one Geometry output. |

Use `range(...)` when the number of loop iterations is known during compilation. Use [`repeat_range(...)`](BUILTINS.md#runtime-loops) when the iteration count must be a runtime input or when you want a Blender Repeat Zone.

## Use the Library panel

The **Library** panel contains reusable scripts and package management tools. Package-provided entries appear only after that package has been installed explicitly:

| Catalog | Purpose |
| --- | --- |
| **Local** | NodeForge-managed `.nf` files and read-only external source folders. |
| **Functions** | Reusable DSL functions provided by installed packages. |
| **Examples** | Example scripts provided by installed packages. |
| **Packages** | Install and remove NodeForge packages. |

Each script catalog has **Refresh** and **Add Node Group** actions.

1. Open a catalog section, for example **Functions**.
2. Click **Refresh** to scan that catalog.
3. Select an entry from the list.
4. Click **Add Node Group** to insert that entry into the active Geometry Nodes editor.
5. Select the inserted group node and click **Load Script From Selected NodeGroup** if you want to inspect or edit the stored source.

## Save your own script to Local

Use **Local** when you want to keep a script as a reusable file instead of only keeping it as a node group inside the Blender file.

1. Write or load a script in a Text datablock.
2. Open **Library → Local**.
3. Use the folder rows and arrow buttons to open the managed destination where the script should be stored.
4. Optionally click **New Folder** to create a subfolder in the current managed directory.
5. Click **Save**, enter the script name, and confirm. Enable **Overwrite** when replacing an existing file.
6. Select the saved script and click **Add Node Group** to insert it into the current Geometry Nodes editor.

The **Save** and **New Folder** actions are available only in NodeForge-managed Local directories.

## Add an external Local folder

Use **Add Folder...** to make an existing directory of `.nf` files available in Local without copying it.

1. Open **Library → Local** and click **Add Folder...**.
2. Select an existing folder.
3. Select the linked folder row and use its arrow button to browse its scripts and subfolders.
4. Select a script and click **Add Node Group**, or import it with `from local import ...`.

External folders are read-only in NodeForge. Edit their `.nf` files with your normal editor, then select an inserted group and click **Reload from Source** to rebuild it from the current file. **Remove from Local** removes the folder registration and leaves the external files on disk.

For managed Local content, **Delete File** removes a selected `.nf` file. **Delete Folder** removes a selected empty folder.

Folders organize the Local browser, while import names remain flat. A file such as `shapes/cube_row.nf` is imported with `from local import cube_row`. If separate Local roots contain the same public filename, rename or remove one source before importing that name.

## Reuse a saved script from another script

A saved Local script can be imported and called from another NodeForge script. This is the main reuse mechanism: write a script once, save it as a `.nf` file, then call it from other scripts like a function.

The saved script's `input_*` declarations become function parameters. One output becomes the returned value; multiple outputs can be unpacked or selected with a compile-time index.

For example, save this as a Local script named `cube_row`:

```python
spacing = input_float('Spacing', default=1.25)
count = 5

parts = []
for i in range(count):
    base = cube(size=1.0)
    offset = vector(i * spacing, 0, 0)
    moved = transform(base, translation=offset)
    parts.append(moved)

geo = join(parts)
output('Geometry', geo)
```

Then use it from another script:

```python
from local import cube_row

spacing = input_float('Spacing', default=2.0)
geo = cube_row(spacing)
output('Geometry', geo)
```

Here `Spacing` from the saved script becomes the `spacing` argument of `cube_row(spacing)`. The saved script's `Geometry` output becomes the returned `geo` value.

Existing compiled groups keep the Local dependency source snapshot used when they were built. Editing a Local file affects a group when you compile a new caller or use **Reload from Source** on the existing library-backed group.

For the import and call rules, see [Imports](SYNTAX.md#imports) and [Reusable Functions](SYNTAX.md#reusable-functions). For reusable functions from the Math library, see [Math Functions](FUNCTIONS.md).

## What to read next

Read these in order:

1. [DSL Syntax and Semantics](SYNTAX.md) — learn the language rules and what is not Python.
2. [Core DSL Built-ins Reference](BUILTINS.md) — learn the primitive functions available everywhere.
3. [Writing Functions](WRITING_FUNCTIONS.md) — create reusable functions with the Core DSL.
4. [Math Methods](MATH_METHODS.md) — use the methods added by the Math library.
5. [Math Functions](FUNCTIONS.md) — use reusable DSL functions from the Math library.
6. [Math Examples](MATH_EXAMPLES.md) — explore complete scripts included with the Math package.
7. [LSystem](LSYSTEMS.md) — create geometry with L-system rules.

To study an installed example, add it from **Library → Examples**, load its source with **Load Script From Selected NodeGroup**, and edit the source in a Text datablock.
