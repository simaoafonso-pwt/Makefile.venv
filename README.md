# Seamlessly manage Python virtual environment with a Makefile

*Makefile.venv* takes care of creating, updating and invoking Python virtual
environment that you can use in your Makefiles. It will allow you to reduce
venv related routines to almost zero!

[![test status][badge]][tests]

[badge]: https://github.com/sio/Makefile.venv/workflows/Run%20automated%20tests/badge.svg
[tests]: tests


## Installation

### Recommended method

Copy [*Makefile.venv*](Makefile.venv) to your project directory and add
include statement to the bottom of your `Makefile`:

```make
include Makefile.venv
```

### Alternative method

Alternatively, you can add installation actions as the Makefile rule:

> **Note the checksum step!** Do not skip it, it would be as bad as [piping curl
> to shell](https://0x46.net/thoughts/2019/04/27/piping-curl-to-shell/)!

```make
include Makefile.venv
Makefile.venv:
	curl \
		-o Makefile.fetched \
		-L "https://github.com/sio/Makefile.venv/raw/v2020.08.14/Makefile.venv"
	echo "5afbcf51a82f629cd65ff23185acde90ebe4dec889ef80bbdc12562fbd0b2611 *Makefile.fetched" \
		| sha256sum --check - \
		&& mv Makefile.fetched Makefile.venv
```


## Usage

When writing your Makefile use `$(VENV)/python` to refer to the Python
interpreter within virtual environment and `$(VENV)/executablename` for any
other executable in venv.


## Demo screencast

<a href="https://asciinema.org/a/279646" target="_blank">
<img src="https://asciinema.org/a/279646.svg" title="Demo screencast"/>
</a>


## Targets

*Makefile.venv* provides the following targets. Some are meant to be executed
directly via `make $target`, some are meant to be dependencies for other
targets written by you.

##### venv

Use this as a dependency for any target that requires virtual environment to
be created and configured.

*venv* is a .PHONY target and rules that depend on it will be executed every
time make is run. This behavior is sensible as default because most Python
projects use Makefiles for running development chores, not for artifact
building. In cases where that is not desirable use [order-only prerequisite]
syntax:

```make
artifacts.tar.gz: | venv
	...
```

[order-only prerequisite]: https://www.gnu.org/software/make/manual/html_node/Prerequisite-Types.html

##### python, ipython

Execute these targets to launch interactive Python shell within virtual
environment. Ipython is not installed by default when creating the virtual
environment but will be installed automatically when called for the first
time.

##### shell, bash, zsh

Execute these targets to launch interactive command line shell. `shell` target
launches the default shell Makefile executes its rules in (usually /bin/sh).
`bash` and `zsh` can be used to refer to the specific desired shell (if it's
installed).

##### show-venv

Execute this target to show versions of Python and pip, and the path to the
virtual environment. Use this for debugging purposes.

##### clean-venv

Execute this target to remove virtual environment. You can add this as a
dependency to the `clean` target in your main Makefile.

##### $(VENV)/executablename

Use this target as a dependency for tasks that need `executablename` to be
installed if `executablename` is not listed as project's dependency neither in
`requirements.txt` nor in `setup.py`. Only packages with names matching the
name of the corresponding executable are supported.

This can be a lightweight mechanism for development dependencies tracking.
E.g. for one-off tools that are not required in every developer's
environment, therefore are not included in formal dependency lists.

**Note:** Rules using such dependency MUST be defined below
`include` directive to make use of correct $(VENV) value.

Example (see `ipython` target for another example):

```Makefile
codestyle: $(VENV)/pyflakes  # `venv` dependency is assumed and may be omitted
	$(VENV)/pyflakes .
```

## Variables

*Makefile.venv* can be configured via following variables:

##### PY

Command name for system Python interpreter. It is used only initially to
create the virtual environment. *Default: python3*

##### REQUIREMENTS_TXT

Space separated list of paths to requirements.txt files.
Paths are resolved relative to current working directory.
*Default: requirements.txt*

##### WORKDIR

Parent directory for the virtual environment. *Default: current working
directory*

##### VENVDIR

Python virtual environment directory. *Default: $(WORKDIR)/.venv*

##### VENV_ARGS

Arguments passed to the virtual environment creation. Defaults to nothing.

##### PIP_*

Variables named starting with `PIP_` are not processed by *Makefile.venv* in
any way and are passed to underlying pip calls as is. See [pip
documentation](https://pip.pypa.io/en/stable/user_guide/#environment-variables)
for more information.

Use these variables to customize pip invocation, for example to provide custom
package index url:

```
PIP_EXTRA_INDEX_URL="https://your.index/url"
```


## Samples

Makefile:

```make
.PHONY: test
test: venv
	$(VENV)/python -m unittest

include Makefile.venv
```

Larger sample from a real project can be seen
[here](https://github.com/sio/issyours/blob/master/Makefile).
Also see [an introductory blog
post](https://potyarkin.ml/posts/2019/manage-python-virtual-environment-from-your-makefile/)
from project author.

Command line:

```
$ make test

...Skipped: creating and updating virtual environment...

...
----------------------------------------------------------------------
Ran 3 tests in 0.000s

OK
```
```
$ make show-venv
Python 3.5.4 (v3.5.4:3f56838, Aug  8 2017, 02:07:06) [MSC v.1900 32 bit (Intel)]
pip 19.2.3 from c:\users\99e7~1\appdata\local\temp\.venv\lib\site-packages\pip (python 3.5)
venv: C:\Users\99E7~1\AppData\Local\Temp\.venv
```
```
$ make python
C:/Users/99E7~1/AppData/Local/Temp/.venv/Scripts/python
Python 3.5.4 (v3.5.4:3f56838, Aug  8 2017, 02:07:06) [MSC v.1900 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> _
```


## Compatibility

*Makefile.venv* was written for GNU Make and may not work with other make
implementations. Please be aware that GNU Make [can not correctly handle][spaces]
whitespace characters in file paths. Such filepaths therefore are
considered unsupported by *Makefile.venv*

[spaces]: https://stackoverflow.com/questions/9838384/can-gnu-make-handle-filenames-with-spaces

*Makefile.venv* is being continuously tested on Linux, Windows and macOS. Any
inconsistency encountered when running on Windows should be considered a bug
and should be reported via [issues].


## Support and contributing

If you need help with using this Makefile or including it into your project,
please create **[an issue][issues]**.
Issues are also the primary venue for reporting bugs and posting feature
requests. General discussion related to this project is also acceptable and
very welcome!

In case you wish to contribute code or documentation, feel free to open
**[a pull request](https://github.com/sio/Makefile.venv/pulls)**. That would
certainly make my day!

I'm open to dialog and I promise to behave responsibly and treat all
contributors with respect. Please try to do the same, and treat others the way
you want to be treated.

If for some reason you'd rather not use the issue tracker, contacting me via
email is OK too. Please use a descriptive subject line to enhance visibility
of your message. Also please keep in mind that public discussion channels are
preferable because that way many other people may benefit from reading past
conversations. My email is visible under the GitHub profile and in the commit
log.

[issues]: https://github.com/sio/Makefile.venv/issues


## License and copyright

Copyright 2019-2020 Vitaly Potyarkin

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
