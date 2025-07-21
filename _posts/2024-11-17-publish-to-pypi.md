---
layout: post
giscus_comments: true
related_posts: false
title: 'Publish Python packages to PyPI (pip)'
date: 2024-11-17
tags:
  - python
  - publication
category:
  - tutorial
---

A step-by-step pipeline for uploading and publishing a Python package to [PyPI](https://pypi.org/) that can be installed with `pip`. This pipeline includes the repo structure, py project configuration, building and publishing package.

Note that more complex issues may arise if the package is a mixture of multiple languages such as Python and C++. 

# Pre-requisites

We need Python3, `pip`, and an account on [PyPI](https://pypi.org/), with 2FA enabled. 
We also need the following packages, which can be installed with pip

- build
- hatch
- twine

```sh
pip install build
pip install hatch
pip install twine
```



# Build python package

## Repository dir structure

The structure of my repo dir looks like this. All source code of my package `your_pack_name` is in the directory `src/your_pack_name`. Other first-level directories (e.g. `example` which contains small example test data) won't be packaged or uploaded .

```
.
├── LICENSE
├── README.md
├── example
│   └── example.data
├── pyproject.toml
├── requirements.txt
└── src
    └── your_pack_name
        ├── __init__.py
        ├── main.py
        ├── source_file1.py
        ├── source_file2.py
        └── util.py
```

Note that all source files should be organized in the `src/your_pack_name` directory, so that after installation, the relevant functions can be imported by `import your_pack_name`.

Assuming that `main.py` is the entry point of your package when run from the command line (e.g. `python main.py --args something`).

## `__init__.py` 

An `__init__.py` file is necessary for Python to recognize the directory should be treated as a package, not just a regular folder with Python files.

It should contain basic information such as `VERSION`, `AUTHOR`, and functions that are importable by other scripts.

```python
# __init__.py
VERSION = "0.1.0"

# defines: "from your_pack_name import *"
__all__ = ['func1', 'class2']

# those two are importable
from .source_file1 import func1
from .source_file2 import class2

# import main
from .main import main
```

We also want to make an executable command for this package, which is essentially the `main()` function in `main.py` file. This can be configured in the `toml` file below.



## Configure pyproject.toml

We need a configuration file `pyproject.toml` for `hatch` to build the package for us. An example looks like this:

```toml
# pyproject.toml

[build-system]
requires      = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/your_pack_name"]	

[project]
name        = "your_pack_name"	# name of your package, should not conflict with existing PyPI packages
version     = "0.1.0"
license     = "BSD-3-Clause"  # license of your choice
description = "your_pack_name is a py tool that needs short descriptions to modify here" 
authors     = [
  { name="Author 1", email="author1@school.edu" }, 
  { name="Author 2", email="author2@institute.org" },
]
dependencies    = [
  "numpy>=1.0",  # dependencies, it is recommended to include versions of dependencies
  "another dependency available in pypi",	# all dependencies should already be available in PyPI
  "another dependency available in pypi",
]
readme          = "README.md"	# remember to edit the README
requires-python = ">=3.7"
classifiers     = [
  "Programming Language :: Python :: 3",
  "License :: OSI Approved :: BSD License",
]
include = [
    "src/your_pack_name/*.py",	# only those files will be packed
]

[project.scripts]
# If you want an executable command, e.g. "your_exctable_name --args something"
# It has the effect as "python main.py --args something"
"your_exctable_name" = "your_pack_name.main:main"  	

[project.urls]
"Homepage" = "https://your-github"
"Bug Tracker" = "https://your-github/issues"
```



To test whether the `toml` file is valid. We can try to install our package to our own computer. Run this command from the same directory as `pyproject.toml` .

```sh
pip install .
```

If the package is installed successfully and works as expected, we can continue to build and publish.



## Build package

Lastly, build the package using `hatch` or `build`.

```sh
# build with hatch
hatch build	
# OR build with build
python -m build 
```

Then the built packages should be available in a new directory `dist`, as two files `your_pack_name.tar.gz` and `your_pack_name.whl` files. Those two files will be eventually uploaded to PyPI.

Optionally, you can unzip  `your_pack_name.tar.gz` file to ensure it contains all necessary files but no unwanted files.

# Publish to PyPI 

## Optional: test publication on TestPyPI

We can try if everything is right by first publish our package on [TestPyPI](https://test.pypi.org/), so that we won't accidentally screw-up a production environment. Note that TestPyPI is a **different** space from PyPI, so a separate TestPyPI account is needed.

Use `twine` to publish to testpypi. You will be prompted to enter your credentials.

```sh
twine upload --repository testpypi dist/*
```

Let's check whether your package can be installed from [TestPyPI](https://test.pypi.org/). Also you should search your package name on TestPyPI to ensure everything is correct.

```sh
pip install --index-url https://test.pypi.org/simple/ your_pack_name
```

## Publish to PyPI

We will use `twine` to publish the compiled `dist` files to PyPI. You will be prompted to enter your credentials.

```sh
twine upload --repository pypi dist/*
```

Let's check whether your package can be installed from PyPI. Also, you should search your package name on PyPI to ensure everything is correct.

```sh
pip install  your_pack_name
```

Great! We just **published** a package to PyPI! If one is curious, statistics such as the number of downloads can be found on [pepy](https://pepy.tech/) or [pypistats](https://pypistats.org/).

## Optional: PyPI API token

Although it's OK to use passwords, using an API token is more convenient for frequent package updates/uploads.

An API token can be generated in the [Account settings](https://pypi.org/manage/account/) at the bottom of the webpage. Note that all tokens start with `pypi`. Since we use twine, it reads the token from `~/.pypirc`. We should edit the file like the following:

```toml
# in your $HOME/.pypirc file
[pypi]
  username = __token__
  password = pypi-your_new_token
```

From now on, `twine` won't ask for the credentials anymore, but using the token from the `~/.pypirc` file.
