Installing Python
=================

On **Mac** the easiest way to install Python is with [Homebrew](https://brew.sh/):

    $ brew install python
    $ python3 --version
    Python 3.7.7

And on **Ubuntu** just use `apt`:

    $ sudo apt install python3
    $ python3 --version
    Python 3.7.3

That's it.

But there's an alternative - [pyenv](https://github.com/pyenv/pyenv) (see next section). It allows easy version management and an approach that doesn't require root access.

Once you've installed Python, you should start using Python virtual environments for your projects. See my notes [here](python-venv.md) about them.

Pyenv
-----

As just noted above, `pyenv` allows for easy version management and an approach that doesn't require root access. E.g. the standard install tools for your system maybe a little behind the current release cycle of Python, `pyenv` makes it easy to pick up the latest version of Python if you want (or to precisely pick an old version if you need to for some reason) and it makes it easy to switch between such Python versions. And at least as importantly, it does this on a per-user basis and without affecting any system installed version of Python, i.e. `pyenv` does all its work without requiring root or administrator privileges and doesn't risk breaking system behavior.

To install on Mac or Linux:

    $ curl https://pyenv.run | bash

Then add the following three lines to your `~/.bashrc`:

    export PATH=$HOME/.pyenv/bin:$PATH
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"

You can omit the last of those three lines if you're not interested in using pyenv's ability to create and manage venvs (see [here](https://github.com/pyenv/pyenv-virtualenv#usage) for more details). Personally, I just use the normal Python 3 [venv approach](python-venv.md).

Now open a new terminal session or just force a reload of `~/.bashrc` like so:

    $ exec $SHELL

List all the Python versions that `pyenv` knows:

    $ pyenv install --list

There are various Python variants, e.g. `anaconda`, `miniconda`, `pypy` and `stackless`. There's even `micropython` (see [#1588](https://github.com/pyenv/pyenv/issues/1588) for Mac). But just look at the plain version numbers, e.g. `3.8.2`, choose the latest one and install it:

    $ pyenv install 3.8.2

It takes a while but once it's installed, set it as the version to use and check:

    $ pyenv global 3.8.2
    $ python --version
    Python 3.8.2

Note: if you've got an active venv, the version _won't_ change, see [`venv-and-python-version.md`](venv-and-python-version.md) for more about why this is.

When you use `pyenv install --list`, `pyenv` doesn't retrieve a list of known Python versions, it just has a static set that it knows about. You can see that set like so:

    $ ls ~/.pyenv/plugins/python-build/share
    2.1.3    3.1.1    3.5.9    anaconda-2.0.1    anaconda3-4.3.1    ...

So if newer Python versions have been released since you installed `pyenv`, you have to manually update it like so:

    $ pyenv update
    Updating ~/.pyenv...
    remote: Enumerating objects: 113, done.
    ...

Note that `update` is essentially just doing:

    $ cd ~/.pyenv
    $ git pull

Actually, it does a bit more - it does some sanity checking and also updates the various plugins found under `~/.pyenv/plugins` (which are pulled from separate git repos).

Tkiniter
--------

The following section covers `tkinter` on MacOS - on Linux things are rather simpler, see this SO [answer](https://stackoverflow.com/a/31299142).

One issue with `pyenv` on MacOS is that installed Python versions do not automatically include [`tkinter`](https://docs.python.org/3/library/tkinter.html) support.

This is fairly easy to resolve. Assuming you're using `brew`, first determine what version of `tcl-tk` you have installed:

```
$ brew list --version tcl-tk
tcl-tk 8.6.11
```

If it doesn't show a version, then install `tcl-tk`:

```
$ brew install tcl-tk
```

Then determine what version of Python you're using:

```
$ pyenv global
3.8.7
```

Uninstall it (enter `y` when asked if you want to remove it):

```
$ pyenv uninstall 3.8.7
```

Normally, if you wanted to reinstall this version, you'd just enter:

```
$ pyenv install 3.8.7
```

To get it to pick up `tcl-tk`, we need to precede this with a `PYTHON_CONFIGURE_OPTS` value that tells it where to find things:

```
$ PYTHON_CONFIGURE_OPTS="--with-tcltk-includes=-I/usr/local/opt/tcl-tk/include --with-tcltk-libs='-L/usr/local/opt/tcl-tk/lib -ltcl8.6 -ltk8.6'" pyenv install 3.8.7
```

Adjust the references to `8.6` if your version of `tcl-tk` is different (note that the version above was `8.6.11`, you drop the `.11` bit).

To check that things worked, first make sure you're using the Python version you just installed and then run the `tkinter` module:

```
$ pyenv global 3.8.7
$ python --version
Python 3.8.7
$ python -m tkinter
```

A little `tkinter` window should whizz around your screen and then stop somewhere looking like this:

![tkinter demo window](images/tkinter.png)

Note: how to set `PYTHON_CONFIGURE_OPTS` correctly was found in `pyenv` issue [#1375](https://github.com/pyenv/pyenv/issues/1375).
