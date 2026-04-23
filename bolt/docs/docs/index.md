# Bolt Runtime Documentation

Official technical reference for Bolt v2.0.0+ and the newest module workflow. High-performance distribution meets structured engineering ⚙️

## Table of Contents

- [Installation & Verification](#installation--verification)
- [Running a Script](#run-flow--bytecode)
- [Command Line Interface](#command-line-interface)
- [Syntax Reference](#syntax-reference)
- [Modules](#modules-v200)
- [Pre-baked Libraries](#pre-baked-libraries)
- [Syntax - Deep Reference](#syntax---deep-reference)
- [Error Handling](#error-handling)
- [Calculator Example](#calculator-example)
- [Examples](#examples)
- [Articles](#articles)
- [Troubleshooting](#troubleshooting)
- [Legal & Credits](#legal--credits)

---

## Installation & Verification

Download Bolt from the official distribution and verify signed artifacts before running binaries. After installation confirm the runtime is accessible from your shell.

### Verify installation

```bash
bolt --version
```

Canonical output sample to expect (example):

```
Bolt Runtime Engine v2.0.0+ (V: official=X/X)
[build metadata and release date fields here]
This version was released on 202X-0X-0X proudly built by the Swifteex Studio build system.
Please verify signed artifacts before running.
```

Release metadata and authorship are published with the builds. For reference, this documentation was produced by the Swifteex Studio stack and distributed by HyperClouD.

If your output reports at least **v1.61+**, typed helpers are available. For v2.0.0+ features such as module file metadata, refreshed module loading, and Python module wrappers, the runtime must report **v2.0.0+**.

Support and downloads: https://swifteexstudio.github.io/Swifteex/

---

## Run Flow & Bytecode

Bolt keeps the edit-compile-run workflow that produces portable bytecode files. This section describes the recommended developer flow.

### Step-by-step: write → compile → run

1. **Create your source file**
   Save your script with the `.bolt` extension.

2. **Open a command prompt / terminal in the folder where your file is located**
   Use PowerShell or Command Prompt on Windows.

3. **Compile**
   Run the compiler switch specifying your `.bolt` file:

   ```bash
   Bolt -c filename.bolt
   ```

   If compilation succeeds a `filename.bvmb` file will be produced in the same folder.

4. **Run the bytecode**
   Execute the produced bytecode with the runtime:

   ```bash
   bolt filename.bvmb
   ```

   Use `--debug` or `--json-errors` for diagnostics in automation or CI.

Notes:

- Compilation is optional for quick interactive experiments, but the `-c` → `.bvmb` flow keeps artifacts consistent for shipping and automation.
- Bytecode files are portable across compatible minor releases.

---

## Command Line Interface

Flags and runtime modes in Bolt. Combine them to fit your workflow.

- `--run [files...]` — execute `.bolt` or `.bvmb` files
- `--exec-string "<source>"` — run inline source
- `--cmd` — single-line interactive mode
- `--safe` — sandbox mode
- `--debug` — verbose logging and tracebacks
- `--inspect-ast` — print AST before execution for diagnostics
- `--json-errors` — machine-readable error output for CI

```bash
bolt --safe --debug --run myscript.bolt
# or run compiled bytecode:
bolt myscript.bvmb
```

---

## Syntax Reference

This section focuses on syntax and features available from Bolt v1.61+ and newer. Bolt v2.0.0+ keeps compatibility while adding the refreshed module workflow.

### Printing - `sys.out`

**What it is and what it does:** `sys.out` writes text or evaluated tokens to standard output. Use forward slashes to expand variables inside output: `/var/`.

```bolt
sys.out 'hello, world!'

name = 'Ariyan'
sys.out /name/

num1 = 10
num2 = 15
sys.out /num1/ + /num2/

sys.out 'the calculation is /num1 + num2/'
```

If you see literal tokens such as `/name/` in output, confirm the variable is defined and that you used forward slashes without spaces.

### Variables

Assignment uses equals. Starting with v1.61+ integers and numeric types are available and usable in arithmetic.

```bolt
myvariable = 'Alice'
age = 11

sys.out /myvariable/
sys.out /age/
```

### Lists

Create lists with `let.list['a' ; 'b']`. Index with 0-based notation.

```bolt
listname = let.list['1', '2']

sys.out /listname[0]/
```

### `in` Keyword

```bolt
listname = let.list['1', '2']
if 3 in listname
    sys.out 'ok'
else
    sys.out 'nope'
end.if
```

### Functions and Invocation

Functions are declared with `fn name(); ... end`. The helper `let.call name()` is available and standard on v1.61+.

```bolt
fn greet(name);
  sys.out 'hello /name/!'
end

let.call greet(ariyan)
```

### Typed Input Helpers (v1.61+)

In v1.61+ Bolt provides typed input helpers to capture typed values directly: `let.userinput.int 'prompt'` for integers and `let.userinput.str 'prompt'` for strings.

```bolt
num = let.userinput.int 'enter a number: '
name = let.userinput.str 'enter your name: '
```

Bolt supports specific typed inputs like `int` and `str`. You can also check the input type using `:int` or `:str`.

```bolt
intvalue = let.userinput.int 'value: '
if intvalue == :int
    sys.out 'it is integer'
eif intvalue == :str
    sys.out 'it is string'
else
    sys.out 'INVALID'
end.if
```

### Control Flow - `if`, `eif`, `else`

v1.61+ includes the conditional ladder: `if`, `eif` for else-if, and `else` for fallback. Use `end.if` to close the block.

```bolt
if condition
  sys.out 'yes'
eif other_condition
  sys.out 'other'
else
  sys.out 'no'
end.if
```

### While Loops

Bolt supports `while` for repeated execution.

```bolt
while condition
  work
end.while
```

Example:

```bolt
x = 1
while x < 5
  sys.out 'looping'
  x = x + 1
end.while
```

You can also use `boltloop` for fixed-count repetition. See [Boltloop](#boltloop---fixed-repetition).

---

## Syntax - Deep Reference

Strict and detailed reference for Bolt v1.61+ and v2.0.0+ features. Examples follow the documented syntax and avoid unsupported constructs.

### Printing - `sys.out` (deep)

#### Purpose

Provide deterministic output to stdout. It is the primary documented mechanism for emitting textual results and computed numeric values.

#### Exact syntax

`sys.out 'literal string'` prints the literal string verbatim.
`sys.out /identifier/` prints the value of a variable named `identifier`.

#### Variable interpolation rules

1. Interpolation tokens are delimited with forward slashes: `/.../`.
2. Inside an interpolation token place a single variable or a numeric expression built from numeric variables and literal numbers.
3. Do not include whitespace between the slashes and the token name if you expect expansion.

#### Examples

```bolt
name = 'Ariyan'
sys.out /name/

a = 5
b = 4
sys.out /a/ + /b/

sys.out 'the sum is /a + b/'
```

#### Common errors & fixes

- **Tokens printed verbatim:** confirm the variable is defined and the token contains valid content.

#### Requirements & runtime notes

The interpolation behavior assumes Bolt v1.61+. Older runtimes may not evaluate expressions inside tokens.

---

### Variables - Declaration & Types

#### Purpose

Hold immutable or mutable values such as strings, integers, numeric types, and lists.

#### Exact syntax and semantics

```bolt
identifier = expression
```

#### Examples

```bolt
username = 'guest'
count = 3
total = count + 2

sys.out /username/
sys.out /total/
```

---

### Lists - Creation, Indexing, Length

#### Purpose

Lists are ordered collections providing indexed access using 0-based positions.

#### Exact syntax

`let.list['item1' ; 'item2' ; 'itemN']` constructs a list. Index with square brackets: `listVar[index]`.

#### Examples

```bolt
listname = let.list['1' ; '2']

sys.out /listname[0]/
```

#### Indexing rules & safety

- Indexes are integers starting at 0.
- Accessing an index outside the valid range raises `bolt.error.MemoryError`. Ensure the index value you use is valid before accessing a list element.
- List contents may be mixed types, so be careful when using items in numeric expressions.

---

### Functions - Declaration, Invocation, Conventions

#### Purpose

Functions encapsulate logic and allow reuse. Bolt uses a compact declaration and a predictable call model.

#### Exact syntax

Declare:

```bolt
fn name();
  ...
end
```

Invoke:

```bolt
let.call name()
```

or, on many runtimes, direct invocation `name()`.

#### Examples

```bolt
fn greet();
  sys.out 'hello'
end

let.call greet()
```

#### Arguments and returns

Bolt v1.61+ keeps the function model compact. To pass data, assign variables in the calling scope before the call or use module-level variables as a deliberate convention. To return values, write to a known variable.

---

### Typed Input Helpers

#### Purpose

Capture user-provided values directly as specific types to avoid manual parsing and conversion bugs.

#### Exact syntax

```bolt
var = let.userinput.int 'prompt'
```

returns an integer.

```bolt
var = let.userinput.str 'prompt'
```

returns a string.

#### Examples

```bolt
num = let.userinput.int 'enter a number: '
name = let.userinput.str 'enter your name: '
```

---

### Control Flow - `if` / `eif` / `else`

#### Purpose

Model conditional decision points. The control flow ladder is intentionally compact and readable.

#### Exact syntax

```bolt
if condition
  ...
eif other_condition
  ...
else
  ...
end.if
```

#### Examples & patterns

```bolt
if /score/ >= 90
  sys.out 'grade: A'
eif /score/ >= 80
  sys.out 'grade: B'
else
  sys.out 'grade: C or below'
end.if
```

---

### While Loops

#### Purpose

Use `while` when repetition depends on a condition. There is no `for` loop in this documented syntax set.

#### Exact syntax

```bolt
while condition
  work
end.while
```

#### Rules

- The condition is checked before each iteration.
- Update the condition inside the block when needed to avoid an endless loop.
- Close the loop with `end.while`.

---

### Boltloop - Fixed Repetition

#### Purpose

Use `BOLTLOOP.LOOP` for a fixed number of repetitions. This is the clean count-based pattern.

#### Exact syntax

```bolt
$get.boltloop

boltloop.loop(times)
  work
end.loop
```

#### Example

```bolt
$get.boltloop

boltloop.loop(5)
  sys.out 'hi'
end.loop
```

#### Rules

- `times` controls the number of executions.
- The block ends with `end.loop`.
- Use this instead of `while` when the count is known in advance.

---

### Error Handling & Diagnostics (CLI)

#### Purpose

Provide clear, reproducible failure messages and machine-readable diagnostics for automation and support workflows.

#### Best practices

- Run with `--debug` for verbose tracebacks during development.
- Use `--json-errors` when attaching logs to support tickets or CI systems.

For structured in-script error handling, see the [Error Handling](#error-handling) section.

---

## Error Handling

Bolt provides a first-class `try`/`catch` construct for intercepting and recovering from both compile-time and runtime errors. This section covers the complete syntax, every catchable error identifier, and idiomatic usage patterns.

---

### The `try` Block

#### Purpose

Wrap any code that may fail — file operations, user input parsing, arithmetic, module calls — inside a `try` block. If an error matching the specified identifier is raised during execution of that block, control transfers immediately to the `catch` branch. The remaining lines inside the `try` block are skipped.

#### Exact syntax

```bolt
try;
    <guarded code>
catch(<error.identifier>) and <handler expression>
end.try
```

Every `try` block must be closed with `end.try`. Every `try` must have at least one `catch`.

The `and` keyword separates the caught error identifier from the handler expression. The handler expression may be a single statement such as `sys.out 'message'`, or a call to a function that contains more complex recovery logic.

#### Minimal example

```bolt
try;
    sys.out 'risky code'
catch(bolt.error.RuntimeError) and sys.out 'something went wrong'
end.try
```

---

### Error Identifier Reference

Bolt organises its error identifiers into four groups. Use the exact identifier string in your `catch` clause.

---

#### Syntax & Structural Errors

These errors are raised by the parser and lexer before the script begins execution. They indicate malformed source that the engine cannot process.

| Identifier | When raised |
|---|---|
| `bolt.error.SyntaxError` | Generic catch-all for any parsing failure not covered by a more specific identifier. |
| `bolt.error.IndentationError` | Code layout violates the strict 2-space indentation requirement. |
| `bolt.error.UnterminatedBlockError` | A block (`fn`, `if`, `try`, `loop`, etc.) was opened but never closed with a matching `end` or `}}`. |
| `bolt.error.LexicalError` | The lexer encountered an unrecognised or illegal character in the source. |

Example — catching a known structural issue during dynamic evaluation:

```bolt
try;
    let.call undefined_fn()
catch(bolt.error.SyntaxError) and sys.out 'parse failed: check your source'
end.try
```

---

#### Runtime Errors

These errors occur while the engine is actively processing logic. They represent failures in execution rather than in parsing.

| Identifier | When raised |
|---|---|
| `bolt.error.ReferenceError` | Attempting to use a variable or function that has not been defined in the current scope. |
| `bolt.error.TypeError` | Incompatible data type operations (e.g. adding a string to a null) or attempting to call a non-function as a function. |
| `bolt.error.ZeroDivisionError` | Division or modulo operation where the divisor is zero. |
| `bolt.error.MemoryError` | Memory safety violations — accessing freed heap memory, or out-of-bounds index access on lists or strings. |
| `bolt.error.ImportError` | The Bolt Module Manager (`bmm`) failed to find, read, or validate a required module. |
| `bolt.error.SystemError` | OS-level issues interfere with execution — commonly during file system interactions or terminal history processing. |
| `bolt.error.RuntimeError` | Base class for general execution failures. Use this as a broad catch when no more specific identifier applies. |

Examples:

```bolt
# Guard against division by zero
divisor = let.userinput.int 'enter divisor: '

try;
    result = 100 / /divisor/
    sys.out /result/
catch(bolt.error.ZeroDivisionError) and sys.out 'ERROR: cannot divide by zero'
end.try
```

```bolt
# Guard against a missing variable reference
try;
    sys.out /undefined_variable/
catch(bolt.error.ReferenceError) and sys.out 'ERROR: variable not defined'
end.try
```

```bolt
# Guard against a type mismatch
try;
    bad = 'hello' + 99
    sys.out /bad/
catch(bolt.error.RuntimeError) and sys.out 'ERROR: incompatible types in operation'
end.try
```

```bolt
# Guard against an out-of-bounds list access
items = let.list['a' ; 'b' ; 'c']

try;
    sys.out /items[99]/
catch(bolt.error.MemoryError) and sys.out 'ERROR: index out of bounds'
end.try
```

---

#### Null-Safety & State Errors (v2.0.0+)

These errors were introduced in v2.0.0 alongside the variable locking system. They are specific to the `$null` literal and the null-lock mechanism.

| Identifier | When raised |
|---|---|
| `bolt.error.NullAssignmentError` | Attempting to assign a new value to a variable that has been locked with the `$null` literal. |
| `bolt.error.NullLockViolation` | Attempting to read or access the value of a variable that is currently nulled. |

Example:

```bolt
# Attempt to write to a null-locked variable
try;
    locked_var = $null
    locked_var = 'new value'
catch(bolt.error.NullAssignmentError) and sys.out 'ERROR: cannot reassign a null-locked variable'
end.try
```

```bolt
# Attempt to read a null-locked variable
try;
    dead_var = $null
    sys.out /dead_var/
catch(bolt.error.NullLockViolation) and sys.out 'ERROR: variable is null-locked and cannot be read'
end.try
```

---

#### External Module Errors — `bfile`

When the `bfile` module is loaded, it registers three additional catchable error identifiers into the current execution environment. These follow the same `catch` syntax as native Bolt errors.

| Identifier | When raised |
|---|---|
| `bfile.error.FileNotFound` | The target file does not exist at the specified path. |
| `bfile.error.AccessDenied` | The OS denied access to the file (insufficient permissions). |
| `bfile.error.IOError` | All other file system failures — corrupt file, full disk, unavailable mount, etc. |

Example — safe file read:

```bolt
$get.bfile

try;
    content = bfile.read(local + '/config.txt')
    sys.out /content/
catch(bfile.error.FileNotFound) and sys.out 'ERROR: config.txt not found'
end.try
```

Example — distinguishing access errors from missing files:

```bolt
$get.bfile

try;
    data = bfile.read('/etc/secret.conf')
    sys.out /data/
catch(bfile.error.FileNotFound) and sys.out 'ERROR: file does not exist'
catch(bfile.error.AccessDenied) and sys.out 'ERROR: permission denied'
catch(bfile.error.IOError) and sys.out 'ERROR: unexpected I/O failure'
end.try
```

---

### Multiple `catch` Clauses

A single `try` block may have multiple `catch` clauses. The engine evaluates them top-to-bottom and executes the first one whose identifier matches the raised error. Subsequent clauses are skipped.

```bolt
try;
    val = let.userinput.int 'enter a number: '
    result = 100 / /val/
    sys.out /result/
catch(bolt.error.ZeroDivisionError) and sys.out 'ERROR: divisor was zero'
catch(bolt.error.TypeError) and sys.out 'ERROR: input was not a number'
catch(bolt.error.RuntimeError) and sys.out 'ERROR: unspecified runtime failure'
end.try
```

**Ordering advice:** Place the most specific error identifiers first and `bolt.error.RuntimeError` last. Since `RuntimeError` is the base class for general execution failures, placing it first would swallow more specific errors before they can be matched.

---

### Using `bolt.error.RuntimeError` as a Broad Catch

`bolt.error.RuntimeError` functions as the widest net available. Use it as a last-resort clause when you need to ensure an error is always handled regardless of type, but you have no specific recovery logic to apply.

```bolt
try;
    let.call risky_operation()
catch(bolt.error.ZeroDivisionError) and sys.out 'specific: division by zero'
catch(bolt.error.RuntimeError) and sys.out 'unhandled runtime error — operation aborted'
end.try
```

---

### Using a Handler Function

For non-trivial recovery logic, extract the handler into a dedicated function and invoke it from the `catch` clause.

```bolt
fn on_file_error();
    sys.out 'File operation failed.'
    sys.out 'Check that the path exists and permissions are correct.'
    sys.out 'Run with --debug for a full trace.'
end

$get.bfile

try;
    content = bfile.read(local + '/data.txt')
    sys.out /content/
catch(bfile.error.FileNotFound) and let.call on_file_error()
catch(bfile.error.AccessDenied) and let.call on_file_error()
end.try
```

---

### CLI Diagnostics Alongside `try`/`catch`

The `try`/`catch` construct handles errors at the script level. For development-time diagnostics that show full stack traces and engine internals, combine it with CLI flags:

```bash
bolt --debug --run script.bolt
```

```bash
bolt --json-errors --run script.bolt
```

`--debug` will report the full context of any error that propagates past all `catch` clauses. `--json-errors` formats uncaught errors as structured JSON for CI pipelines and automated log parsers.

---

### Complete Reference Table — All Catchable Identifiers

| Identifier | Category | Available from |
|---|---|---|
| `bolt.error.SyntaxError` | Syntax & Structural | v1.61+ |
| `bolt.error.IndentationError` | Syntax & Structural | v1.61+ |
| `bolt.error.UnterminatedBlockError` | Syntax & Structural | v1.61+ |
| `bolt.error.LexicalError` | Syntax & Structural | v1.61+ |
| `bolt.error.ReferenceError` | Runtime | v1.61+ |
| `bolt.error.TypeError` | Runtime | v1.61+ |
| `bolt.error.ZeroDivisionError` | Runtime | v1.61+ |
| `bolt.error.MemoryError` | Runtime | v1.61+ |
| `bolt.error.ImportError` | Runtime | v1.61+ |
| `bolt.error.SystemError` | Runtime | v1.61+ |
| `bolt.error.RuntimeError` | Runtime (base) | v1.61+ |
| `bolt.error.NullAssignmentError` | Null-Safety | v2.0.0+ |
| `bolt.error.NullLockViolation` | Null-Safety | v2.0.0+ |
| `bfile.error.FileNotFound` | External (bfile) | v1.70+ |
| `bfile.error.AccessDenied` | External (bfile) | v1.70+ |
| `bfile.error.IOError` | External (bfile) | v1.70+ |

---

## Calculator Example

The calculator example below assumes Bolt v1.61+ so that typed input helpers and modern control flow are available.

### Canonical source

```bolt
fn calc();
  sys.out 'welcome to calc.'

  num1 = let.userinput.int 'enter number first: '
  num2 = let.userinput.int 'enter number second: '

  sys.out '1) addition'
  sys.out '2) subtraction'
  sys.out '3) multiplication'
  sys.out '4) division'
  sys.out 'choose 1 to 4'

  choose = let.userinput.str 'choose: '

  if choose == '1'
    sys.out /num1/ + /num2/
  eif choose == '2'
    sys.out /num1/ - /num2/
  eif choose == '3'
    sys.out /num1/ * /num2/
  eif choose == '4'
    if /num2/ == 0
      sys.out 'ERROR: divide by zero'
    else
      sys.out /num1/ / /num2/
    end.if
  else
    sys.out 'INVALID'
  end.if
end

let.call calc()
```

### Hardened version with error handling

```bolt
fn calc();
    sys.out 'welcome to calc.'

    try;
        num1 = let.userinput.int 'enter number first: '
        num2 = let.userinput.int 'enter number second: '
    catch(bolt.error.TypeError) and sys.out 'ERROR: numeric input required'
    end.try

    sys.out '1) addition'
    sys.out '2) subtraction'
    sys.out '3) multiplication'
    sys.out '4) division'
    sys.out 'choose 1 to 4'

    choose = let.userinput.str 'choose: '

    if choose == '1'
        sys.out /num1/ + /num2/
    eif choose == '2'
        sys.out /num1/ - /num2/
    eif choose == '3'
        sys.out /num1/ * /num2/
    eif choose == '4'
        try;
            sys.out /num1/ / /num2/
        catch(bolt.error.ZeroDivisionError) and sys.out 'ERROR: divide by zero'
        end.try
    else
        sys.out 'INVALID'
    end.if
end

let.call calc()
```

---

## Examples

Small copy-ready samples for quick testing.

### Save as hello.bolt

```bolt
# Save as hello.bolt
sys.out 'hello, world!'
```

### AST inspection

```bash
bolt --inspect-ast --exec-string "sys.out '1+2'"
```

### Module example (mymod.boltmodule)

```
[BV=200, CORE=0, VER: 1.0, LANG=Bolt]

public.module[mymod$1$bolt] {{
fn module_init();
  sys.out 'mymod initialized'
end

fn greet();
  sys.out 'hello from mymod'
end
}}
```

### Using the module in a script (app.bolt)

```bolt
$get.mymodule

let.call mymod.module_init()
let.call mymod.greet()
```

---

## Articles

Short articles to help you design scripts for both interactive usage and automation.

### Safe scripting practices

Detect runtime version with `bolt --version` and choose typed helpers when v1.61+ is detected. Wrap all file operations, user input handling, and arithmetic in `try`/`catch` blocks. For automation, avoid blocking user prompts and provide explicit error codes.

### Distributing scripts

Include a small header that prints expected runtime version and usage. Provide typed-helper paths for v1.61+ and fallback parsing code for earlier runtimes. In distributed scripts, handle errors gracefully with `try`/`catch` so end users receive clean messages rather than raw tracebacks.

---

## Troubleshooting

Concrete steps when Bolt is not recognised or scripts fail.

### Bolt not recognised

1. Confirm install folder exists.
2. Run the executable directly from the folder.
3. Add the install folder to PATH.
4. If the binary is missing or corrupt, re-download and reinstall.

### Script errors or missing helpers

Confirm your runtime version is v1.61+ for typed helpers and modern control flow. For v2.0.0+ module features, confirm the runtime reports v2.0.0+. For null-safety errors (`NullAssignmentError`, `NullLockViolation`), confirm v2.0.0+.

### Collect logs

Run scripts with `bolt --debug --run script.bolt` or `bolt --json-errors --run script.bolt` to get structured logs. Errors that propagate past all `catch` clauses will appear in full detail with either flag.

---

## Legal & Credits

All rights reserved. Verify downloadable artifacts with the signed checksums on the official download page before installation.

© 2026 Swifteex Studio. Documentation and builds assembled by Swifteex Studio. Recommended editor: Visual Studio Code.

Support & downloads: https://swifteexstudio.github.io/Swifteex/

---

© 2026 Bolt Documentation 2.0 Engineering • Lead Architect: Ariyan Ahmed

Support | Home | Terms of Service