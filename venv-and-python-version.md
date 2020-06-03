Venv and the Python version
===========================

The whole point of Python venvs is that no matter what changes occur at a system level, nothing should change within your current environment.

So, if you've activated a venv, almost nothing you should do will change the Python version. By default a venv uses the Python version used to create it:

    $ python3 --version
    Python 3.7.3
    $ python3 -m venv env
    $ source env/bin/activate
    (env) $ python --version
    Python 3.7.3

If virtual environments are new to you, see my notes [here](python-venv.md). As noted there, once the venv is activated, I can simply use `python` and be confident it always means the version used to create the environment.

As a result of all this any system-level changes you make won't be apparent if you're in an active venv:

    (env) $ python --version
    Python 3.7.3
    (env) $ pyenv global 3.8.2
    (env) $ python --version
    Python 3.7.3

Here, despite setting the global version to 3.8.2, the version remains unchanged in my active venv. If I want to see the global version, I have to deactivate my current venv:

    (env) $ deactivate
    $ python3 --version
    Python 3.8.2

If I reactivate my venv the version will return to what it was when the venv was created:

    $ source env/bin/activate
    (env) $ python --version
    Python 3.7.3

You _should_ be able to upgrade a venv like so:

    (env) $ deactivate
    $ python3 -m venv --upgrade env

However, this doesn't work for me - the [docs](https://docs.python.org/3/library/venv.html#creating-virtual-environments) say this option works "assuming Python has been upgraded in-place." It's unclear, to me, what this means but I suspect this may be the issue.

Hardly ideal, but the only way I've found to upgrade the Python version for a venv is to throw away the existing one and recreate it:

    (env) $ deactivate
    $ python3 --version
    Python 3.8.2
    $ rm -r env
    $ python3 -m venv env
    $ source env/bin/activate
    (env) $ python --version
    Python 3.8.2

You of course now have to e.g. upgrade `pip` again and reinstall the various packages you'd installed in the old venv.
