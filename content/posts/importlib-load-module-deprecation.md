---
title: "Fixing the importlib load_module() DeprecationWarning"
date: 2023-03-31T08:53:23-04:00
tags: [python, load_module, importlib, exec_module, DeprecationWarning]
draft: false
---

You may have come across this nasty warning:

```shell
DeprecationWarning: the load_module() method is deprecated and slated for removal in Python 3.12; use exec_module() instead
```

Say you have this piece of code, which is trying to load a `settings.py` file at runtime:

```python
from importlib.machinery import SourceFileLoader
from pathlib import Path

settings_file = Path.cwd() / "settings.py"

user_settings_module = SourceFileLoader(
    settings_file.name,
    str(settings_file.resolve()),
).load_module()
```

The first argument to `SourceFileLoader` is the name of the file and the second is the absolute path.

To fix the warning, the code above must be rewritten as:

```python
from importlib.util import module_from_spec, spec_from_file_location
from pathlib import Path

settings_file = Path.cwd() / "settings.py"

spec = spec_from_file_location(
    settings_file.name, settings_file.resolve()
)
settings_module = module_from_spec(spec)
spec.loader.exec_module(settings_module)
```

`spec_from_file_location` also supports `Path`-like objects.
First, we create a `ModuleSpec` instance to specify its import-system-related
 state. Then, the actual module is created based on
that spec using `module_from_spec`. Finally, we execute the module in its own
namespace using `exec_module`.

### Extra

One of my co-workers found this hack to pass in a global variable using the
return value of `module_from_spec`.

You can use it for something like overriding a setting when the module is loaded.

```python
from importlib.util import module_from_spec, spec_from_file_location
import os
from pathlib import Path

settings_file = Path.cwd() / "settings.py"

spec = spec_from_file_location(
    settings_file.name, settings_file.resolve()
)
settings_module = module_from_spec(spec)
# allow overriding default settings
settings_module.user_date_format = os.getenv("DATE_FORMAT")
spec.loader.exec_module(settings_module)
```

Your `settings.py` can use it this way:

```python
date_format = user_date_format if user_date_format else "DD-MM-YY"
```

The code in the module will be executed using the global variable `user_date_format` when
`exec_module` is called. If the loaded module has breakpoints to debug, those
will also work as they would when run directly.
