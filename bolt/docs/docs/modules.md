## Modules (v2.0.0+)

Getting started: to access the newest feature set, make sure your Bolt runtime is **2.0.0+**. When this section is opened, the page refreshes once and then loads module docs from the current documentation set.

- BV metadata
- CORE flag
- VER tag
- LANG: Bolt / Python

### Creating a module file

Use the `.boltmodule` file extension. Inside the file, include a metadata header and the module wrapper:

```text
[BV=200, CORE=0, VER: 1.0, LANG=Bolt]
```

The fields mean:

- `BV=X` matches the Bolt version target your module file is built for.
- `CORE=X` is the core mode flag, use `1` or `0` depending on the build target.
- `VER=X.X` is the version of your `.boltmodule` file.
- `LANG=Bolt` or `LANG=Python` determines which language the module body uses.

```text
[BV=200, CORE=0, VER: 1.0, LANG=python]

public.module[name$1$lang] {{
# code here
}}
```

Use `LANG=Python` when the code inside the module is valid Python. Use Bolt syntax when the body is Bolt.

There is no extra indentation requirement after `{{`. Keep the body clean and aligned exactly as your module needs it.

### Importing modules in Bolt

Import from the `bmodules/` folder with `$get.name`. The runtime checks `bmodules/` automatically.

```bolt
$get.name

# example
$get.mymodule
```

Keep the module identifier consistent with the file name and exported wrapper name. Use the current docs for the surrounding setup, install, and runtime flow.

### Python module support

When `LANG=python` is used, the code inside the module should be Python valid. The wrapper name still follows the module system rules, and the import path remains the same.

```text
[BV=200, CORE=0, VER: 1.0, LANG=python]

public.module[pytools$1$lang] {{
def hello():
    print("hello from python module")
}}
```

Opening the Modules section refreshes the page once and then loads the module docs view. That keeps the newest module instructions in sync with the current page state.

---

### Pre-baked Libraries

Bolt ships with a small set of convenience libraries for common tasks. They are optional runtime modules.

### Boltclock

Utilities for deterministic pauses and simple scheduling.

```bolt
# pause for 2 seconds
Boltclock.pause(2)

fn afterwait();
  sys.out 'resuming after wait'
end

Boltclock.pause(1)
let.call afterwait()
```

### Boltloop

Simple repeated execution helper.

```bolt
BOLTLOOP.LOOP(5)
  sys.out 'tick'
end.loop
```

Use `BOLTLOOP.LOOP(times)` for fixed-count repetition. The block ends with `end.loop`.

### Random

Pick random items from a provided list or data arguments.

```bolt
Random.main('data1', 'data2', 'data3')

choice = Random.main('red','blue','green')
sys.out /choice/
```

### void.0 - runtime stop

To immediately halt Bolt execution from your script use the sentinel `void.0`.

```bolt
sys.out 'starting'
void.0
# code below will not execute
```