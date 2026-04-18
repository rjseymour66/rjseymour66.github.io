---
title: "Python"
linkTitle: "Python"
weight: 10
description: >
  Notes about basic Python.
---

This section covers Python tooling and language fundamentals. Start here to set up your environment, then work through the language topics in order.

## Toolset

### venv

A *virtual environment* (venv) is an isolated Python installation for a single project. It keeps each project's dependencies separate, so upgrading a package for one project does not break another.

#### Create a virtual environment

Run the following command to create a new virtual environment in the current directory:

```bash
python3 -m venv ./<venv-name>
# For example:
python3 -m venv ./venv
```

#### Activate a virtual environment

Activate the environment before installing packages or running the project:

```bash
source ./<venv-name>/bin/activate
```

#### Deactivate a virtual environment

Run the following command to exit the virtual environment:

```bash
deactivate
```

### pip

*pip* is Python's package installer. Before uninstalling a package, run `pip show` on it to confirm no other package depends on it.

The following commands cover the most common pip tasks:

```bash
pip list                         # list installed packages
pip show <package-name>          # display package details and dependencies
pip uninstall <package-name> -y  # uninstall without a confirmation prompt
```

### requirements.txt

A `requirements.txt` file lists all of a project's dependencies. Pass this file to pip to replicate the environment on another machine.

#### Create and install from a requirements file

The following steps walk through creating a requirements file and installing from it:

1. Run pip freeze to capture the current environment:

```bash
pip freeze > requirements.txt
```

2. Verify the output:

```bash
cat requirements.txt
```

3. In a new virtual environment, install every listed package:

```bash
pip install -r requirements.txt
```

#### Version specifiers

Edit `requirements.txt` manually to adjust version constraints. The following specifiers control which package versions pip accepts:

| Specifier | Meaning | Example |
|---|---|---|
| `==` | Exact version | `Django==4.2.0` |
| `>=` | Minimum version | `requests>=2.28.0` |
| `!=` | Exclude a version | `urllib3!=1.26.0` |
| `<=` | Maximum version | `numpy<=1.24.0` |

Combine specifiers with a comma. For example, `pyvim>=3.0.2,<=4.0.0,!=3.0.6` requires version 3.0.2 through 4.0.0 but excludes 3.0.6.

To upgrade all packages to the latest versions allowed by your constraints:

```bash
pip install --upgrade -r requirements.txt
```

#### Development requirements files

Large projects typically maintain multiple requirements files for different environments. The following table describes each file's purpose:

| File | Purpose |
|---|---|
| `requirements.txt` | Base dependencies required in all environments |
| `requirements_dev.txt` | Additional tools for local development (linters, test runners) |
| `requirements_locked.txt` | Pinned versions of all packages for a known-good production snapshot |

A `requirements_dev.txt` file typically includes the base file and adds development-only packages. For example, a Django project's dev requirements file might look like this:

```
-r requirements.txt
pytest>=7.4.0
black>=23.0.0
mypy>=1.5.0
ruff>=0.1.0
```

The `-r requirements.txt` line tells pip to install the base requirements first, then the additional packages listed below it.
