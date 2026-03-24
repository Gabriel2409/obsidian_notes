https://packaging.python.org/en
https://alpopkes.com/posts/python/packaging_tools/

Note: python makes a difference between distribution package (discussed here) and import packages (collection of modules)

A **wheel** (`.whl`) is a precompiled package that can be installed directly without the need for building or compiling code. It differs from a **source distribution** package (`sdist`, usually distributed in `.tar.gz`), which contains the raw source code and needs to be built before it can be used.

To build a package, you need both a frontend and a backend:
- The easiest frontend is using python -m build, but you can also use alternatives like `hatch`, `poetry`, `flit`, or `uv`.
- For the build backend, you can use tools such as `setuptools`, `hatchling`, or others, which you specify in the `pyproject.toml` file.

Examples: 

```python
# with uv

[build-system]
requires = ["uv_build>=0.9.11,<0.10.0"]
build-backend = "uv_build"

[tool.uv.build-backend]
module-name = "orthofinance" # useful if it does not match package name
# by default uv searches src/<package_name> so if name matches, omit this section


# with hatchling
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/orthofinance"]
```

In the Python packaging ecosystem, the frontend (like `pip`) is responsible for coordinating the installation and management of packages, which includes building them when necessary by calling the appropriate backend.


Note: a wheel is basically a zip archive with a given structure. Example below with uvicorn wheel. Here i extracted the uvicorn wheel in the extracted folder: `unzip uvicorn-0.30.6-py3-none-any.whl -d extracted`

Now in `extracted`, there are 2 folders:
- `uvicorn`, which contains all the source code
- `uvicorn-0.30.6.dist-info/`
```txt
uvicorn-0.30.6.dist-info/
├── METADATA
├── RECORD
├── WHEEL
├── entry_points.txt
└── licenses
    └── LICENSE.md

1 directory, 5 files
```
- licenses contains the list of licenses
- METADATA contains information about the package. This is basically what is displayed on PyPI if you publish it here, which is why it contains html code
- RECORD contains a list of all the files included in the wheel package. Each entry typically consists of 
		- The path to the file (relative to the installation directory).
	- The size of the file in bytes.
	- The SHA256 hash of the file.
- WHEEL contains metadata about the wheel itself:
	- the wheel version
	- the generator (setuptools, hatch, etc)
	- Root-Is-Purelib: boolean indicating whether it is a pure python wheel. If true, all files can be installed in the purelib directory. If false, it may contain platform-specific compiled code. When installing a pure Python wheel, all files can be placed directly in the `site-packages` directory or the equivalent `purelib` directory of the Python installation, as they are universally compatible.
	- Tag: compatibility tags for the wheel
- entry_points.txt: the entrypoint

```txt
[console_scripts]
uvicorn = uvicorn.main:main
```
this entrypoint means that if you run `uvicorn` from the cmd line, it will actually launch uvicorn.main:main

Note: in the uvicorn library, in pyproject.toml, you can find
```txt
[project.scripts]
uvicorn = "uvicorn.main:main"
```
When the wheel is built, it creates the entry_point.txt using this information


NOTE: when you do a `pip install -e mylib` to install your own package in editable mode, in the venv, in site-packages, you will only find the dist-info folder of mylib. This folder contains a `direct_url.json` file, which contains a link to the root folder