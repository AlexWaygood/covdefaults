[![Build Status](https://dev.azure.com/asottile/asottile/_apis/build/status/asottile.covdefaults?branchName=master)](https://dev.azure.com/asottile/asottile/_build/latest?definitionId=62&branchName=master)
[![Azure DevOps coverage](https://img.shields.io/azure-devops/coverage/asottile/asottile/62/master.svg)](https://dev.azure.com/asottile/asottile/_build/latest?definitionId=62&branchName=master)

covdefaults
===========

A coverage plugin to provide sensible default settings

## installation

```bash
pip install covdefaults
```

## usage

to enable the plugin, add `covdefaults` to your coverage plugins

in `.coveragerc`:

```ini
[run]
plugins = covdefaults
```

in `setup.cfg` / `tox.ini`:

```ini
[coverage:run]
plugins = covdefaults
```

in `pyproject.toml`:

```ini
[tool.coverage.run]
plugins = ["covdefaults"]
```

## default settings

### `[coverage:run]`

```ini
branch = True
source = .
omit =
    */.tox/*
    */__main__.py
    */setup.py
    */venv*/*
```

Note: if you enable installed packages the python environment folders (`.tox`,
`venv*`) might not be added like here, but instead exploded for folders going
into the python virtual environment to avoid excluding the installed packages
path.

### `[coverage:report]`

```ini
show_missing = True
skip_covered = True
fail_under = 100
exclude_lines =
    # a more strict default pragma
    \# pragma: no cover\b

    # allow defensive code
    ^\s*raise AssertionError\b
    ^\s*raise NotImplementedError\b
    ^\s*return NotImplemented\b
    ^\s*raise$

    # typing-related code
    ^if (False|TYPE_CHECKING):
    : \.\.\.$
    ^ +\.\.\.$
    -> ['"]?NoReturn['"]?:

    # non-runnable code
    if __name__ == ['"]__main__['"]:$

    # additional platform related pragmas (see below)
```

### platform specific `# pragma: no cover`

several `# pragma: no cover` tags will be added automatically based on the
platform and implementation.

these will be in the form of:

```python
# pragma: TAG no cover
```

or

```python
# pragma: TAG cover
```

these tags will be generated by the following values:

- `os.name`
    - `nt` (windows)
    - `posix` (linux, macOs, cygwin, etc.)
- `sys.platform`
    - `cygwin`
    - `darwin` (macOs)
    - `linux`
    - `msys`
    - `win32`
- `sys.implementation.name`
    - `cpython`
    - `pypy`

for every tag which does not match, you can use negation.  here's an example:

```python
if sys.platform == 'win32':  # pragma: win32 cover
    bin_dir = 'Scripts'
else:  # pragma: win32 no cover
    bin_dir = 'bin'
```

note here that `# pragma: win32 cover` will become a "no cover" for everything
which is not `win32` -- whereas the `# pragma: win32 no cover` will be a
"no cover" only on `win32`.

### overriding options

several of the options can be overridden / extended in your coverage
configuration.  the examples below assume `.coveragerc` however any of the
files `coverage` supports work as well.

#### `run:omit`

```ini
[run]
omit =
    pre_commit/resources/*
```

this will result in the `pre_commit/resources/*` being `omit`ted in addition
to the defaults provided by `covdefaults`.

```ini
[covdefaults]
subtract_omit = */.tox/*
```

this will result in `*/.tox/*` not being `omit`ted (`*/.tox/*` is among the
defaults provided by `covdefaults`).

#### coverage for installed libraries

By default ``covdefaults`` assumes that you want to track coverage
in your local source tree, code that is version controlled. To detect
coverage on your installed version, and map that back to the source files.
You can enable tracking installed libraries via:

```ini
[coverage:covdefaults]
installed_libraries = tox:src virtualenv
```

In this example we say we'll track the installed package/module tox that maps
back to the source folder present at ``src``, and ``virtualenv`` that maps back
to the source folder present at ``.``. Note the installed package/module must be
present in either the ``platlib`` or ``purelib`` path of the virtual environment
as specified by ``sysconfig``.

```ini
[covdefaults]
subtract_omit = */.tox/*
```

this will result in `*/.tox/*` not being `omit`ted (`*/.tox/*` is among the
defaults provided by `covdefaults`).

#### `report:exclude_lines`

```ini
[report]
exclude_lines =
    ^if MYPY:$
```

this will result in lines matching `^if MYPY:$` to additionally be excluded
from coverage in addition to the defaults provided by `covdefaults`.

#### `report:fail_under`

```ini
[report]
fail_under = 90
```

`covdefaults` will not change the value if you provide one for `fail_under`
