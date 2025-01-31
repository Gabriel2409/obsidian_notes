#python
## Install

`git clone git@github.com:python/cpython.git --branch 3.9`

Then in repo: 
```bash
./configure
make -j2
```

- Note: if changes to grammar: run `make regen-pegen`, `make regen-token`, `make regen-all`
- After each change, recompile with `make`
- Use compiled version with `./python`


Note: for lsp, add `compile_flags.txt` with 
```txt
-I./Include
-I./Include/internal
```
## Structure

- Doc: documentation as restructured text: Check `content.rst` to see all entries in the same order as what there is on https://docs.python.org/3/
- Grammar: Contains grammar and tokens (computer readable language definition)
- Include: C header files
- Lib: std library modules written in python
- Mac: macos support files
- Misc: miscellanous files
- Modules: std lib modules written in C
- Objects: core types and object model
- Parser: Python parser source code
- PC: old windows support files
- PCBuild: windows build support files
- Programs: source code for the python executable and other binaries
- CPython: the CPython interpreter source code
- Tools: standalone tools useful for building/extending cpython
- m4: custom scripts to automate configuration of the makefile

## Language and grammar

When you run a python application, the CPython runtime compiles your code to bytecode when it runs for the first time. Compilation is implicit

It is then stored in pyc (python compiled) files and cached for execution. 
This bytecode is what the Python interpreter actually runs. The compiled bytecode instructions are executed by the **Python Virtual Machine (PVM)**.


