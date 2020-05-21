Scheme for SICP
===============

After googling about I eventually decided on Racket with its SICP package after reading the "Scheme for SICP" section of ["An opinionated guide to scheme implementations"](https://wingolog.org/archives/2013/01/07/an-opinionated-guide-to-scheme-implementations).

SICP was written for [MIT Scheme](https://www.gnu.org/software/mit-scheme/) - but MIT Scheme seems rather unloved - it's last release was 2 years ago and the one previous to that 5 years ago.

**Update:** MIT Scheme seems to have come back to life and they've been pushing out fairly frequent releases since 10.1 came out in late 2018.

Setup
-----

Install the Racket PPA for the latest Racket version.

The Ubuntu PPA is linked to from the Racket [download site](https://download.racket-lang.org/).

    $ sudo add-apt-repository ppa:plt/racket
    $ sudo apt-get update

You can see what packages the new PPA provide:

    $ cat /var/lib/apt/lists/ppa.launchpad.net_plt_racket_ubuntu_dists_*_main_binary-amd64_Packages

Now install everything:

    $ sudo apt-get install racket racket-doc

You can see what's been added by the above two (plus `racket-common` which is also pulled in):

    $ dpkg-query -L racket
    $ dpkg-query -L racket-common
    $ dpkg-query -L racket-doc

The `racket-doc` package installs `/usr/share/doc/racket/index.html` and its related content.

This is just the version of the documentation that you find at the Racket [docs site](https://docs.racket-lang.org/) that corresponds to your installed version of Racket.

You can open the above `index.html` file directly or trigger it to be opened from the command line:

    $ raco docs

Or from the Racket REPL:

    $ racket
    > help

Now let's get the DrRacket IDE setup with the "sicp" package:

    $ drracket 

Go to _File / Package Manager..._ and enter "sicp" in the _Package Source_ field and hit _Install_.

Once completed go to _Language / Choose Language..._ and select the first option "The Racket Language" (if it's not already selected by default). There's no separate option for SICP - it's not really a separate language (despite having its own `#lang` directive - see later).

None of the above steps need to be repeated when opening DrRacket after this.

For more details on the "sicp" package see Racket [SICP manual](http://docs.racket-lang.org/sicp-manual/).

Note: the "sicp" package that you installed and any other settings end up under `~/.racket`.

Usage
-----

In the _upper_ pane of the main DrRacket window change `#lang racket` to `#lang sicp` and hit Run.

You can now enter `(inc 42)` in the lower REPL pane (and get an instant result) or enter it in the upper editor pane and hit Run again to get a result.

---

For a Racket quick start see ["Quick: An Introduction to Racket with Pictures"](https://docs.racket-lang.org/quick/).

For a DrRacket quick start see the ["DrRacket Interface Essentials"](https://docs.racket-lang.org/drracket/interface-essentials.html).
