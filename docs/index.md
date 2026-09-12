# NodeForge Documentation

NodeForge lets you write Python-like scripts and compile them into Blender Geometry Nodes node groups. A script describes the group interface and node logic, and NodeForge builds the native node group inside Blender.

## What NodeForge is

NodeForge uses a DSL, or **domain-specific language**, designed for Geometry Nodes. Assignments, function calls, comments, arithmetic, `if`, `for`, lists, imports, and local helper functions use familiar Python syntax.

The language is a focused, compiler-supported subset of Python. NodeForge accepts the syntax and built-ins documented in this manual and translates them into Geometry Nodes operations. Inputs and outputs become group sockets, expressions become nodes and links, and supported control flow becomes generated graph structure.

The compiled result is an ordinary Blender Geometry Nodes group. It can be inspected and connected like a manually authored node group, and it remains usable in the `.blend` file when the NodeForge add-on is disabled.

## Getting Started

- [Get Started](GET_STARTED.md)
- [DSL Syntax and Semantics](SYNTAX.md)
- [Geometry Nodes Coverage](GEOMETRY_NODES_COVERAGE.md)
- [Core DSL Built-ins](BUILTINS.md)
- [Writing Functions](WRITING_FUNCTIONS.md)
- [Packages](PACKAGES.md)

## Libraries

- [Math Methods](MATH_METHODS.md) and [Math Functions](FUNCTIONS.md)
- [LSystem Reference](LSYSTEMS.md)
- [Writing L-System Scripts](LSYSTEMS_SCRIPTING.md)
