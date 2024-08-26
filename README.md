# ApiVer - a sustainable approach for library development and versioning

[![ApiVer presentation](https://github.com/reef-technologies/apiver/assets/122983254/8e2b9b63-475c-4d3f-aed3-ae8c4a784b61)](https://youtu.be/FgcoAKchPjk?t=82)


## Why ApiVer?

ApiVer saves library users from having to cross their fingers every time they upgrade a package,
thus encouraging them to keep it up to date while reducing the maintenance burden on the ecosystem as a whole. 

### Problem

SemVer promises a stable API for a given major version.
The problem is that rarely do opensource projects have the resources
to maintain multiple major versions of their package.
This shifts and multiplies the burden of maintenance from a package maintainer to all the users of the package.
End users have to postpone upgrading to the latest version of a package 
because they are afraid of the cost associated with breaking API changes.
This leads to a situation where end users are stuck with old versions of a package,
which may contain bugs and security vulnerabilities.
In a world where millions of applications are built on top of thousands of packages,
this is not sustainable and leads to wasted resources and frustration.

### Solution

ApiVer is a way to introduce a new API while minimizing the cost of maintenance across the package ecosystem.
Instead of bumping the major version of your package, you can create a new API interface in a separate namespace - also called - "apiver".
To prevent having to maintain multiple major versions of your package, you provide a thin compatibility layer that translates old API to new one.
That way old users can rest assured that their code will continue to work, while new users can take advantage of the new API.

## ApiVer & SemVer are complementary

Even if you use ApiVer, you should still use SemVer for your releases.
There will be times when you will need to make a breaking change - due to the limits of your programming language, or it's package distribution ecosystem.
Thanks to ApiVer tho, you will need to bump the major version of your package much less often.

## ApiVer Good practices

How ApiVer can be implemented highly depends on the programming language, and it's package distribution ecosystem.

### ApiVer for CLI

There are multiple ways to implement ApiVer for CLI tools.

The recommended way is to use environment variable to specify the expected apiver.
That way, you can ensure that the users of your CLI tool in interactive mode are using the latest and greatest API, while still allowing them to use the old API in their scripts.

e.g. 

```bash
export MY_SCRIPT_APIVER=v1
my-script --help
``` 

Alternatively, you can symlinked (or entirely separate) binary for each apiver.

```bash
my-script-v1 --help
```

These approaches can be combined, as it is easy to recognize the "called name" of the script.
However, while env vars are accessible on virtually all OSes, symlinks may not be.

### ApiVer for Python packages

Python with it's dynamic nature, and package distribution ecosystem, is a great fit for ApiVer.

#### Suggested python package structure

```
<package-name>
├── __init__.py
├── v1.py
├── v2.py
├── _v3.py
└── _internal
    ├── __init__.py
    ├── module1.py
    └── module2.py
```

`_v3`
```python
from your_package._internal.module1 import func1
from your_package._internal.module2 import func2
```

```python
import your_package._v3 as _v3
from your_package._v3 import *


def func2():
    print("func2 got called")
    return _v3.func2()
```

`v2`
```python
import your_package._v2 as _v2
from your_package._v2 import *


def func():
    print("func got called")
    return _v2.func()
```


By default, you should consider everything private, and therefore hide it in private `_internal` module.
Thanks to python flexibility, users that want to make a conscious decision to use private, therefore prone to change, API, can do so by importing from `._internal` module.
However, the majority, who just want to use the public apiver modules (i.e., `.v1`, `.v2`, etc.), can do so without worrying about changes in the private API.

Before you are ready to release new apiver module, you can keep it in `_` prefix, to indicate that it is not yet ready for public consumption.
In the meatime, you can publish all compatible changes to the previous apiver modules.

Flat modules for each API versions' are not requirement, but they are recommended as it makes easier to import things.
To keep import time low, you may want to keep using submodules for modules that import large 3rd party libraries.


### Testing Python ApiVer modules

Tests are critical in any project that wants to prevent regressions.
If you use `pytest` already or are open to using it, you will be well served by [pytest-apiver](https://github.com/reef-technologies/pytest-apiver) plugin.

## Projects using ApiVer

### Libraries and SDKs
* [b2sdk](https://github.com/Backblaze/b2-sdk-python)
* [logfury](https://github.com/reef-technologies/logfury)
* [django-business-metrics](https://github.com/reef-technologies/django-business-metrics)
* your next project

### CLI  
* [bt-auto-dumper](https://github.com/bactensor/bt-auto-dumper/blob/9ec42ed946086f88cdd963b1842f718b28e06071/src/bt_auto_dumper/__main__.py#L20) - CLI using Environment variable to select CLI ApiVer
* [B2 Command Line Tool](https://github.com/Backblaze/B2_Command_Line_Tool) - CLI using multiple binaries providing different apivers for scripting purposes, while main binary provides latest stable apiver
