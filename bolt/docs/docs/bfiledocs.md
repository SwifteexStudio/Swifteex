# Bfile Module — Official Reference

**Module:** `bfile`
**Type:** Core Built-in
**Available in:** Bolt v1.70+
**Recommended:** Bolt v2.0.0+
**Synchronicity:** Fully synchronous (blocking I/O)

---

Bfile is the official filesystem module bundled directly into the Bolt runtime. It provides a clean, safe, and portable API for reading, writing, appending, copying, and managing files and directories — without requiring any external libraries or manual stream handling. All operations are synchronous and return values immediately.

---

## Table of Contents

- [Importing Bfile](#importing-bfile)
- [The `local` Keyword](#the-local-keyword)
- [Safe Mode](#safe-mode)
- [Error Identifiers](#error-identifiers)
- [Functions](#functions)
  - [bfile.read](#bfileread)
  - [bfile.write](#bfilewrite)
  - [bfile.writefile](#bfilewritefile)
  - [bfile.append](#bfileappend)
  - [bfile.exists](#bfileexists)
  - [bfile.isfile](#bfileisfile)
  - [bfile.isdir](#bfileisdir)
  - [bfile.listdir](#bfilelistdir)
  - [bfile.remove](#bfileremove)
  - [bfile.rmdir](#bfilermdir)
  - [bfile.rmtree](#bfilermtree)
  - [bfile.copy](#bfilecopy)
  - [bfile.mkdir](#bfilemkdir)
  - [bfile.joinpath](#bfilejoinpath)
- [Error Handling Patterns](#error-handling-patterns)
- [Cross-Platform Notes](#cross-platform-notes)

---

## Importing Bfile

Bfile must be explicitly loaded before use. Use the `$get` directive at the top of your script.

```
$get.bfile
```

This directive links all `bfile.*` functions into the current execution context. Calling any `bfile.*` function without this directive will raise a runtime module error.

**Version check:**
If your runtime is below v1.70, the directive will fail silently or throw an import error. Verify your version:

```bash
bolt --version
```

---

## The `local` Keyword

Bfile introduces a special path keyword: `local`.

When `local` is passed as a path argument to any bfile function, it resolves to **the directory containing the currently running script** — not the working directory from which the `bolt` command was invoked.

```
$get.bfile

# These two calls are equivalent when running from C:\projects\app\
content = bfile.read("C:\projects\app\config.txt")
content = bfile.read(local + "\config.txt")
```

**Fallback behaviour:** If the script has no known source path (e.g., it was passed as an inline string via the REPL or `bolt -e`), `local` resolves to the current working directory (`os.getcwd()`).

**Usage with joinpath (recommended):**

```
$get.bfile

config_path = bfile.joinpath(local, "config", "settings.json")
data = bfile.read(config_path)
sys.out data
```

`local` works as a first-class value anywhere a path string is accepted.

---

## Safe Mode

Bfile respects the Bolt interpreter's **safe mode** flag. When the interpreter is started in safe mode, all file operations are restricted to the directory of the running script.

**Restrictions in safe mode:**

- Read, write, append, copy, list, and remove operations must target paths **within** the script's own directory tree.
- Attempting to access paths outside that tree raises a `RuntimeError` immediately.
- Overwriting the running script itself is also blocked.

**Error raised:**

```
File access to '/etc/passwd' is disallowed in safe mode.
```

Safe mode is intended for sandboxed or untrusted script execution environments. There is no bfile-level API to toggle safe mode — it is set at the interpreter level.

---

## Error Identifiers

Bfile uses a three-value error namespace accessible at `bfile.error`:

| Identifier | Value String | When Raised |
|---|---|---|
| `bfile.error.FileNotFound` | `"bfile.error.FileNotFound"` | Target file does not exist |
| `bfile.error.AccessDenied` | `"bfile.error.AccessDenied"` | OS-level permission denied |
| `bfile.error.IOError` | `"bfile.error.IOError"` | All other I/O failures |

These string values are embedded in all exception messages raised by bfile functions, making them parseable in error-handling logic or log filtering.

```
$get.bfile

result = bfile.read("missing.txt")
# If the file is missing, the runtime throws:
# RuntimeError: bfile.error.FileNotFound: File not found: 'missing.txt'
```

Use `--debug` for full stack traces:

```bash
bolt --debug --run script.bolt
```

---

## Functions

---

### bfile.read

Reads the full contents of a file and returns them as a string.

```
bfile.read(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to the file. Accepts absolute paths, relative paths, or `local`. |

**Returns:** `string` — the raw file contents including all whitespace and newlines.

**Errors raised:**

- `bfile.error.FileNotFound` if the file does not exist.
- `bfile.error.AccessDenied` if the OS denies the read.
- `bfile.error.IOError` for all other failures.

**Behaviour note:** If the return value is not assigned to a variable, the content is printed directly to `sys.out`.

**Examples:**

```
$get.bfile

# Read and print a file
bfile.read("readme.txt")

# Read into a variable
content = bfile.read("readme.txt")
sys.out content
```

```
$get.bfile

# Read using the local keyword
notes = bfile.read(local + "/notes.txt")
sys.out notes
```

```
$get.bfile

# Read and check contents
data = bfile.read("users.txt")
if "admin" in data
    sys.out "Admin record found"
else
    sys.out "No admin record"
end.if
```

---

### bfile.write

Writes data to a new file constructed from a directory, filename, and extension. **Overwrites** the file if it already exists.

```
bfile.write(data, directory, filename, extension)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `data` | any | Yes | The content to write. Converted to string automatically. |
| `directory` | string | Yes | Target directory. Accepts absolute path, relative path, or `local`. |
| `filename` | string | Yes | The base filename without extension. |
| `extension` | string or type literal | Yes | File extension. A leading `.` is added automatically if omitted. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` on write failure.
- Safe mode error if the resolved path is outside the script directory.

**Extension handling:** The extension parameter accepts both plain strings (`"txt"`, `".txt"`) and Bolt type literals (`.txt`). In all cases, a single leading `.` is guaranteed.

**Examples:**

```
$get.bfile

# Write a plain text file to the script directory
bfile.write("Hello, world!", local, "output", "txt")
# Creates: <script_dir>/output.txt
```

```
$get.bfile

username = "Ariyan"
report = "User: " + username + " logged in."
bfile.write(report, local, "log", ".log")
# Creates: <script_dir>/log.log
```

```
$get.bfile

# Write with a constructed directory
bfile.write("data here", local + "/exports", "result", "csv")
# Creates: <script_dir>/exports/result.csv
```

---

### bfile.writefile

Writes data to a specific, fully-specified file path. **Overwrites** the file if it already exists. This is the simpler alternative to `bfile.write` when you already have a complete path.

```
bfile.writefile(path, data)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Full path to the target file. Accepts `local`, absolute, or relative paths. |
| `data` | any | Yes | The content to write. Converted to string automatically. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` on write failure.
- Safe mode error if path is outside the script directory.

**Examples:**

```
$get.bfile

bfile.writefile(local + "/config.txt", "debug=true")
```

```
$get.bfile

path = bfile.joinpath(local, "output", "result.txt")
bfile.writefile(path, "Final result: 42")
```

---

### bfile.append

Appends data to an existing file. If the file does not exist, it is **created**.

```
bfile.append(path, data)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to the target file. Accepts `local`, absolute, or relative. |
| `data` | any | Yes | The content to append. Converted to string automatically. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` on append failure.
- Safe mode error if path is outside the script directory.

**Note:** No newline is appended automatically. If you want each append on a new line, include `"\n"` in your data.

**Examples:**

```
$get.bfile

bfile.append(local + "/log.txt", "Session started\n")
bfile.append(local + "/log.txt", "User connected\n")
```

```
$get.bfile

# Build a log entry
entry = "[INFO] Operation completed\n"
bfile.append(local + "/events.log", entry)
```

---

### bfile.exists

Checks whether a file or directory exists at the given path.

```
bfile.exists(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to check. Accepts `local`, absolute, or relative. |

**Returns:** `bool` — `true` if the path exists (file or directory), `false` otherwise.

**Examples:**

```
$get.bfile

if bfile.exists(local + "/config.txt")
    sys.out "Config found"
else
    sys.out "Config missing — using defaults"
end.if
```

```
$get.bfile

target = local + "/data/output.csv"
if bfile.exists(target)
    bfile.remove(target)
end.if
bfile.writefile(target, "col1,col2\n")
```

---

### bfile.isfile

Checks whether a path exists **and** refers specifically to a file (not a directory or symlink to a directory).

```
bfile.isfile(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to check. |

**Returns:** `bool`

**Examples:**

```
$get.bfile

path = local + "/data.json"
if bfile.isfile(path)
    content = bfile.read(path)
    sys.out content
else
    sys.out "Not a file"
end.if
```

---

### bfile.isdir

Checks whether a path exists **and** refers specifically to a directory.

```
bfile.isdir(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to check. |

**Returns:** `bool`

**Examples:**

```
$get.bfile

target_dir = local + "/exports"
if bfile.isdir(target_dir)
    sys.out "Export directory ready"
else
    bfile.mkdir(target_dir)
    sys.out "Export directory created"
end.if
```

---

### bfile.listdir

Returns a list of names (files and subdirectories) inside a directory.

```
bfile.listdir()
bfile.listdir(path)
```

**Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `path` | string | No | `local` | Directory to list. Defaults to the script's own directory. |

**Returns:** `list` of strings — the names of entries in the directory (not full paths).

**Errors raised:**

- `bfile.error.IOError` if the directory cannot be listed (e.g., does not exist or no permission).
- Safe mode error if path is outside the script directory.

**Note:** Entries are names only, not full paths. Use `bfile.joinpath` to construct full paths from results.

**Examples:**

```
$get.bfile

# List the script's own directory
entries = bfile.listdir()
sys.out entries
```

```
$get.bfile

# List a subdirectory
files = bfile.listdir(local + "/assets")
sys.out files
```

```
$get.bfile

# Check if a specific file is present
entries = bfile.listdir()
if "config.bolt" in entries
    sys.out "Config script detected"
end.if
```

---

### bfile.remove

Removes a single file at the given path. **Silently succeeds** if the file does not exist.

```
bfile.remove(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to the file to delete. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` on OS-level failure (e.g., path is a directory, permission denied).
- Safe mode error if path is outside the script directory.

**Note:** This function only removes files. To remove directories, use `bfile.rmdir` or `bfile.rmtree`.

**Examples:**

```
$get.bfile

bfile.remove(local + "/temp.txt")
sys.out "Cleanup complete"
```

```
$get.bfile

old_log = local + "/logs/session_old.log"
if bfile.isfile(old_log)
    bfile.remove(old_log)
end.if
```

---

### bfile.rmdir

Removes a **single, empty** directory. Fails if the directory contains any files or subdirectories.

```
bfile.rmdir(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to the empty directory to remove. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` if the directory is not empty or cannot be removed.
- Safe mode error if path is outside the script directory.

**Silently succeeds** if the directory does not exist.

**Examples:**

```
$get.bfile

bfile.rmdir(local + "/tmp_empty")
```

To remove a directory with contents, use `bfile.rmtree`.

---

### bfile.rmtree

Recursively removes a directory **and all of its contents** — files, subdirectories, and nested structures. This is a destructive operation with no undo.

```
bfile.rmtree(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path to the root directory to remove recursively. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` on failure.
- Safe mode error if path is outside the script directory.

**Silently succeeds** if the directory does not exist.

**Warning:** This permanently deletes everything inside the target directory. There is no confirmation prompt or recycle bin. Use with care.

**Examples:**

```
$get.bfile

# Wipe and recreate a build directory
bfile.rmtree(local + "/build")
bfile.mkdir(local + "/build")
sys.out "Build directory reset"
```

---

### bfile.copy

Copies a file from a source path to a destination path. Preserves file metadata (timestamps, permissions) using the underlying `copy2` mechanism.

```
bfile.copy(src_path, dest_path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `src_path` | string | Yes | Path to the source file. |
| `dest_path` | string | Yes | Path to the destination (file path or directory). |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` if the copy fails for any reason.
- Safe mode errors apply independently to both `src_path` and `dest_path`.

**Behaviour:** If `dest_path` is an existing directory, the file is copied into it using the source filename. If `dest_path` is a full file path, it is used as the output filename.

**Examples:**

```
$get.bfile

# Copy a config file to a backup
bfile.copy(local + "/config.txt", local + "/config.bak")
```

```
$get.bfile

# Copy a file into a subdirectory
bfile.mkdir(local + "/backup")
bfile.copy(local + "/data.json", local + "/backup")
# Result: <script_dir>/backup/data.json
```

---

### bfile.mkdir

Creates a directory at the given path. **Automatically creates all missing parent directories.** Silently succeeds if the directory already exists.

```
bfile.mkdir(path)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | Yes | Path of the directory to create. Supports nested paths. |

**Returns:** `null`

**Errors raised:**

- `bfile.error.IOError` if creation fails (e.g., a file already exists at that path).
- Safe mode error if path is outside the script directory.

**Examples:**

```
$get.bfile

# Create a single directory
bfile.mkdir(local + "/output")
```

```
$get.bfile

# Create a deeply nested directory structure in one call
bfile.mkdir(local + "/output/reports/2025/q1")
```

```
$get.bfile

# Conditional creation
if !bfile.isdir(local + "/cache")
    bfile.mkdir(local + "/cache")
end.if
```

---

### bfile.joinpath

Joins multiple path components into a single well-formed path string, handling separators automatically across platforms.

```
bfile.joinpath(part1, part2, ...)
```

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `part1, part2, ...` | string (variadic) | At least one recommended | Path components to join. The first component may be `local`. |

**Returns:** `string` — a joined path string using the OS-appropriate separator.

**Behaviour:** If called with no arguments, returns an empty string. If the first argument is `local`, it is resolved to the script's directory before joining.

**Note:** This is the recommended way to build paths in Bolt scripts. It prevents double-slash errors, handles Windows/Unix differences, and integrates cleanly with `local`.

**Examples:**

```
$get.bfile

# Join three components
path = bfile.joinpath(local, "data", "results.txt")
sys.out path
# Output (Linux): /home/user/project/data/results.txt
# Output (Windows): C:\Users\user\project\data\results.txt
```

```
$get.bfile

# Use in a read call
folder = "configs"
file = "app.json"
full = bfile.joinpath(local, folder, file)
content = bfile.read(full)
sys.out content
```

```
$get.bfile

# Build a log path and append
log_path = bfile.joinpath(local, "logs", "run.log")
bfile.append(log_path, "Startup complete\n")
```

---

## Error Handling Patterns

Bfile does not provide a try/catch-style mechanism in Bolt v1.x. Errors raised by bfile functions propagate as runtime exceptions and will halt execution unless the interpreter is invoked with error tolerance flags.

**Recommended defensive pattern — check before acting:**

```
$get.bfile

config = local + "/config.txt"

if !bfile.exists(config)
    sys.out "Error: config.txt not found"
    void.0
end.if

data = bfile.read(config)
sys.out data
```

**Recommended pattern — validate before write:**

```
$get.bfile

out_dir = bfile.joinpath(local, "output")

if !bfile.isdir(out_dir)
    bfile.mkdir(out_dir)
end.if

bfile.writefile(bfile.joinpath(out_dir, "result.txt"), "done")
```

**Debugging failed operations:**

Run with the `--debug` flag to receive full stack traces and error context:

```bash
bolt --debug --run script.bolt
```

The error message format for all bfile errors is:

```
RuntimeError: bfile.error.<ErrorType>: <human-readable description>
```

Parse the `bfile.error.*` prefix to distinguish filesystem errors from other runtime errors in logs or tooling.

---

## Cross-Platform Notes

Bfile is designed to work identically on Windows, Linux, and macOS.

- **Path separators:** Use `bfile.joinpath` to construct paths — never hard-code `\` or `/` separators manually in path strings.
- **`local` keyword:** Resolves correctly regardless of the working directory from which `bolt` was invoked.
- **Line endings:** `bfile.read` and `bfile.write` use UTF-8 encoding with Python's default newline handling. On Windows, `\r\n` in files is read as-is. If you need consistent `\n` across platforms, normalise the string after reading.
- **Encoding:** All read and write operations use UTF-8 exclusively. Files with other encodings (e.g., Latin-1) may produce garbled output or errors.
- **Metadata preservation:** `bfile.copy` uses `copy2` internally, preserving timestamps and permission bits where the OS allows.

---

*End of bfile reference. For interpreter internals, see the bolt200 engine documentation.*
