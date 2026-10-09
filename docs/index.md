# NodeForge Documentation

This manual documents the current NodeForge language and user-facing APIs.

NodeForge is a Python-like domain-specific language (DSL) that compiles source scripts into native Blender Geometry Nodes groups. A script declares the group interface and the graph computation that connects its inputs to its outputs.

## What NodeForge is

NodeForge uses familiar assignments, function calls, arithmetic, `if`, `for`, lists, imports, and local helper functions as a focused language for graph authoring.

A source variable can hold compile-time data or represent a typed Geometry Nodes value. Operations on runtime values become nodes and links: group inputs supply values, expressions define data flow, and outputs connect that flow to the group interface.

The language separates two evaluation layers:

- **Compile time** resolves graph structure, declarations, and authoring decisions. A loop over `range(...)` is unrolled while the graph is built.
- **Runtime** is represented inside the generated graph. Group-input values remain adjustable, runtime branches use Switch nodes, and `repeat_range(...)` creates a native Repeat Zone.

The compiled result is an ordinary Blender Geometry Nodes group. It can be inspected and connected like a manually authored group, and it remains usable in the `.blend` file when the NodeForge add-on is disabled.

Every generated group embeds its source. The source can be loaded into a Text datablock, edited, and compiled back into the same group while compatible external links and user-overridden input values are preserved.

Read [Core Concepts](CORE_CONCEPTS.md) for the complete compilation and update model.

## Getting Started

- [Core Concepts](CORE_CONCEPTS.md)
- [Get Started](GET_STARTED.md)
- [DSL Syntax and Semantics](SYNTAX.md)
- [Geometry Nodes Coverage](GEOMETRY_NODES_COVERAGE.md)
- [Core DSL Built-ins](BUILTINS.md)
- [Writing Functions](WRITING_FUNCTIONS.md)
- [Package Guide](PACKAGES.md)
- [Extension API Reference](EXTENSION_API.md)

## Libraries

- [Math Methods](MATH_METHODS.md), [Math Functions](FUNCTIONS.md), and [Math Examples](MATH_EXAMPLES.md)
- [L-System Guide](LSYSTEMS_SCRIPTING.md)
- [L-System Reference](LSYSTEMS.md)
