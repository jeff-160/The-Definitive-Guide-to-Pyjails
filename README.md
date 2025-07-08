# The Definitive Guide to Pyjails

## Introduction

A PyJail, or Python Jail, refers to a restricted Python environment designed to limit the capabilities of code executed within it. It is commonly encountered in Capture The Flag (CTF) challenges and security exercises, where the goal is to "escape" the jail and achieve a higher level of access or control, often by accessing restricted functions.

---
## Traditional Pyjails

### Unicode bypass 😈😈😈
Probably the most straightforward way of escaping a pyjail if a SKID wrote it.  

Python normalises italics for some reason, but the characters don't match your normal ASCII table, so standard blacklists won't work.  

https://lingojam.com/ItalicTextGenerator

### Helpful builtins

`eval`, `exec`, `compile`: execute arbitrary code

### Spawning shells / running system commands

Shell commands: `sh`, `python3`

Helpful functions:
- `os.system()`: probably the most well-known method     
- `subprocess.call()`, `subprocess.run()`: allows you to spawn shells
- `subprocess.Popen()`, `subprocess.check_output()`: runs commands and returns the output
- `pty.spawn`: lesser known but still helpful!

Debug mode: `breakpoint()`, `__import__('sys').breakpointhook()`, `pdb.set_trace()`
- starts a python debugger

Python Interactive mode
- `code.interact()`: launches a Python shell
- `os.environ["PYTHONINSPECT"] = "1"`: launches a Python shell when your script finishes execution

### Reading files

### No builtins???

### NO PARENTHESES??? 💔💔💔

### More stuff

Dynamically accessing attributes
- `getattr()`: for objects
- `__getitem__()`: for dictionaries

Octal encoding
- bypass blacklists within your strings!
- eg: `\146\154\141\147.txt` -> `flag.txt`

Assigning variables
- the walrus operator treats an assignment as an expression
- eg: `a:=1`

---
## SSTI

---
## Pickle