Python virtual environments
===========================

Python virtual environments are an attempt to define a known good environment with a clearly established set of packages that is insulated from changes outside the environment.

**TL;DR** - for every Python project you start you should create a correspond virtual environment like so:

    $ cd my-project
    $ python3 -m venv env
    $ source env/bin/activate
    (env) $ pip install --upgrade pip

That's it. The `source` step is the only one you ever need to repeat, you'll need to use it whenever you open a new terminal session in order to activate the environment for that session.

Once the environment is active, the version of `python` and `pip` are unaffected by changes made at a wider system level:

    (env) $ python --version
    Python 3.5.2
    (env) $ pip --version
    pip 20.1.1 from .../env/lib/python3.5/site-packages/pip (python 3.5)

Note that when I create the venv here, I use `python3` to make sure I don't accidentally pick up a pre-Python 3 version that may exist on the system. But once the venv is activated I can simply use `python` and be confident it always means the version used to create the environment. Similarly `pip` will be the version appropriate to the environment.

> According to the ["For Python runtime distributors"](https://www.python.org/dev/peps/pep-0394/#for-python-runtime-distributors) section of PEP 394:
>
>> When a virtual environment (created by the [PEP 405](https://www.python.org/dev/peps/pep-0405/) `venv` package or a similar tool such as `virtualenv` or `conda`) is active, the `python` command should refer to the virtual environment's interpreter and should always be available. The `python3` or `python2` command (according to the environment's interpreter version) should also be available.

The `venv env` step above simply creates a subdirectory in the current directory with the name `env` (or whatever you choose, `env` is just a name) that contains everything that makes up your environment:

    (env) $ ls env
    bin/  include/  lib/  lib64@  pyvenv.cfg  share/

The `source` step activates the environment by reading and executing the commands in `env/bin/activate`. Among other things this includes `env/bin` in you path and updates your prompt to indicate the activated environment. You can deactive it like so:

    (env) $ deactive
    $

The package installer tool `pip` is generally updated far more often than Python. When you create a virtual environment you generally get the version of `pip` that was packaged with the version of Python that you used to create the environment.

This may be quite out-of-date so typically the first thing to do, on having created a new virtual environment, is to update `pip`.

Note that there's no similarly simple step you can take to upgrade Python itself. The best tool for managing Python versions and keeping up-to-date with the latest releases of Python is `pyenv`. Even then you need to have determined the version of Python you want to use _before_ you create a venv, once created the venv is essentially tied to that Python version.

You might ask "why not just create one venv for all my projects?" This is essentially what you get if you use `pip` which the `--user` flag. This is indeed safer than using `sudo` and installing packages in some system location - you won't break any system behavior. But you will eventually break one of your own projects. If you use a single venv you'll eventually install a newer version of some package that one of your older projects already depended on but which has changed in a non-backward compatible way that will break that older project. It's much better to maintain environments on a per-project basis.

The dependency problem
----------------------

So what's the underlying problem? If you're used to dependency management systems from other languages you know it's completely possible to maintain multiple versions of the same dependency under a common root.

E.g. if you're using a language like Java with a package manager like [Maven](https://en.wikipedia.org/wiki/Apache_Maven) and are developing with a popular framework like [Spring Boot](https://spring.io/projects/spring-boot) then you'll be used to multiple versions of Spring Boot (and other dependencies) building up over time under `~/.m2/repository` without any problem:

    $ ls ~/.m2/repository/org/springframework/boot/spring-boot
    2.0.3.RELEASE/ 2.0.5.RELEASE/ 2.1.4.RELEASE/ 2.1.5.RELEASE/ 2.2.2.RELEASE/ 2.2.5.RELEASE/ 2.3.0.RELEASE/

The issue with Python is that while `pip` downloads (and properly caches) specific versions of packages it unpacks them into unversioned directory names, i.e. it's missing the `2.0.3.RELEASE` etc. bit that we see above for the Spring Boot example.

E.g. if you install `numpy` version 1.16.6 then `pip` will download the bundle that corresponds to that version, cache it and then unpack it to a directory named `numpy`.

If you ever ask to install that version again `pip` can correrctly find the locally cached version and quickly install it. The issue is that it if you use a common root installation directory then a given dependency is always unpacked to the same subdirectory, e.g. `numpy`, irrespective of its version.

So if you later install `numpy` version 1.18.4, it will essentially end up in the same location as your earlier 1.16.6 version - essentially upgrading the package.

Upgraded packages may sound nice, but if the package has changed in a non-backward compatible manner then this may well break any code that was depending on 1.16.6 behavior.

This isn't a theoretical concern - if you do any serious Python development you will quickly hit this issue in practice.

The problem often isn't the packages you install directly, it's the things those packages pull in, sometimes called transitive or indirect dependencies, i.e. the dependencies of dependencies and so on (you can end up with a whole tree of transitive dependencies for a single package that you install directly).

So take two popular packages [cryptography](https://pypi.org/project/cryptography/) and [html5lib](https://pypi.org/project/html5lib/), both depend on [six](https://pypi.org/project/six/) - if you used cryptography in one project and html5lib in another but installed all your Python packages to a common root location then you could find cryptography depended on a version of six that's incompatible with the version needed by html5lib and depending on the order which you installed things you could end up unknowingly breaking cryptography or html5lib though this indirect dependency.

Note: even if you use venvs, you can still end up with this type of problem, e.g. if you used both cryptography and html5lib there's no getting around the fact that you may have difficulties satisfying the transitive dependency versions. Luckily this generally isn't an issue for popular packages that are kept up-to-date - you more often run into this issue if installing one package now and another several months later, once the world and the underlying dependencies have moved on.

Luckily things aren't quite as extreme as the Javascript world where it's very common for packages to pull in huge numbers of additional dependencies. In the Python world most packages pull in relatively few additional dependencies.

For reasons that aren't clear to me this whole problem seems to have become less of an issue over the years - perhaps this reflects a more mature Python ecosystem.

Back-in-the-day getting a compatible set of transitive dependencies for a mid-sized project with a reasonable number of direct dependencies could be a nightmare and you'd end up using a tool like Conda.

Conda
-----

Sometimes venvs aren't enough and getting a work set of dependencies within a single project becomes too hard. This is where [Conda](https://docs.conda.io/projects/conda/en/latest/) comes in. It's an alternative to `pip`, the crucial difference it that the company behind conda maintain a huge set of packages that are known to be compatible.

You can choose to download the entire repository of packages and know that you have every package you're ever probably likely to need and that they, and their transitive dependencies, should all work well together.

But more likely you want to install just the package manager and download packages as you need.

The Conda people came up with a very confusing set of similar terms that many people have difficulty keeping separate in their heads - Conda, Miniconda and Anaconda. For a good explanation of what these terms mean, see this [SO answer](https://stackoverflow.com/a/58147674/245602).

If you want to try it out, I suggest you go to the Miniconda [installers page](https://docs.conda.io/en/latest/miniconda.html), get the version for Python 3 and go from there. Note that this installs not just the `conda` tool but also a suitable version of Python 3 to use with it.

The details
-----------

Python and the package installer `pip` have changed a lot over time. Try the following:

    $ python2 --version
    Python 2.7.12
    $ python --version
    Python 2.7.12
    $ python3 --version
    Python 3.5.2
    $ pip2 --version
    pip 8.1.1 from /usr/lib/python2.7/dist-packages (python 2.7)
    $ pip --version
    pip 8.1.1 from /usr/lib/python2.7/dist-packages (python 2.7)
    $ pip3 --version
    pip 8.1.1 from /usr/lib/python3/dist-packages (python 3.5)

Depending on your system you may find you either don't have Python installed at all or that you just have either Python 2 or 3 installed. Similarly, depending on your system `python` and `pip` may be linked to the Python 2 or 3 versions.

This is all a bit confusing.

More important than this though is how brittle package installation is with Python.

Without venvs packages generally get installed to one of two locations - a system wide location and a user specific location.

You can see where these locations are like so:

    $ python3 -m site
    sys.path = [
        '.',
        '/usr/lib/python35.zip',
        '/usr/lib/python3.5',
        '/usr/lib/python3.5/plat-x86_64-linux-gnu',
        '/usr/lib/python3.5/lib-dynload',
        '~/.local/lib/python3.5/site-packages',
        '/usr/local/lib/python3.5/dist-packages',
        '/usr/lib/python3/dist-packages',
    ]
    USER_BASE: '~/.local' (exists)
    USER_SITE: '~/.local/lib/python3.5/site-packages' (exists)
    ENABLE_USER_SITE: True

The `USER_SITE` value gives the location for packages installed for the current user, the system location isn't so clear but in this situation it's `/usr/lib/python3/dist-packages`.

You can install a package with `--user` and see that it ends up in a user specific location:

    $ pip3 install --user foobar
    Collecting foobar
    Installing collected packages: foobar
    Successfully installed foobar-0.1.0

    $ pip3 show foobar
    Name: foobar
    Installer: pip
    Location: ~/.local/lib/python3.5/site-packages
    ...

    $ ls ~/.local/lib/python3.5/site-packages/foobar
    README.md  main.py  __pycache__/

And you can install a package with `--system` and see it end up in a location shared by the entire system:

    $ sudo pip3 install --system foobar
    [sudo] password for joeblow: 
    Collecting foobar
    Installing collected packages: foobar
    Successfully installed foobar-0.1.0

    $ pip3 show foobar
    Name: foobar
    Installer: pip
    Location: /usr/local/lib/python3.5/dist-packages
    ...

    $ ls /usr/local/lib/python3.5/dist-packages/foobar
    README.md  main.py  __pycache__/

Note that the `sudo` had to be used to install to the system location. Don't do this. If you see any tutorial or StackOverflow answer suggesting this, just don't do it. Whatever about accidentally breaking your own projects by installing incompatible dependencies that's nothing compared to breaking system behavior by doing the same thing on a system wide basis.

As you can see `pip` downloads a specific version in both cases but the results end up in directories which aren't qualified by a version number.

Non-backward compatible changes are surprisingly common in the Python ecosystem so installing everything like this into one common area makes your Python setup extremely brittle. Not only do you risk breaking changes to your own projects but if you install into the system wide location you also risk breaking system behavior.

In short you should leave the system packages as they are - they've all been chosen to work together. If you do need to install something Python related for the system you should do this with the system specific installation tools, e.g. `apt`, rather than `pip`. However, in general you should _never_ need to do this - you should only ever be installing packages for your own projects.

Since version 20.0 (release late January, 2020) `pip` defaults to installing to a user specific location (see the the first item of the _Features_ section of the [20.0 release notes](https://pip.pypa.io/en/stable/news/#id48)). Debian and Ubuntu actually unilateraly defaulted to thie behavior far earler (see the 8.0.2-4 section of the [python-pip changelog](https://metadata.ftp-master.debian.org/changelogs/main/p/python-pip/unstable_changelog)).

TODO
----

Standardize on venv rather than virtual environment once you've introducted the term.

There have been various virtual environment schemes over the years - virtualenv - and various tools have builtin virtualenv support, e.g. pyevn virtualenv plugin and Python Poetry virtualenv support, but these days most people are using the standard Python 3 builtin venv support.

Go thru your other venv notes elsewhere and point them here.
