Installing node and npm
=======================

The easiest way to install node (and npm) is with [nvm](https://github.com/nvm-sh/nvm) (nvm allows easy version management and doesn't require root access). To install on Mac or Linux:

    $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

See the `0.35.3` in the URL? That was the current version at the time of writing, find the latest version [here](https://github.com/nvm-sh/nvm/tags).

This installation step updates your `~/.bashrc` so open a new terminal session, to pick up these changes,  or just force a reload of `~/.bashrc` like so:

    $ exec $SHELL

Now use nvm to install the latest version of node (and with it npm):

    $ nvm install node
    $ node --version
    v13.12.0
    $ npm --version
    6.14.4
