# Persona Assertion Token (PASSporT)

## Building the Draft

Formatted text and HTML versions of the draft can be built using `make`.

```sh
$ make
```

This requires that you have the necessary software installed.  There are several
other tools that are enabled by make, check the Makefile for details, including
links to the software those tools might require.


## Installation and Setup

Mac users will need to install
[Xcode](https://itunes.apple.com/us/app/xcode/id497799835) to get `make`, see
[this answer](http://stackoverflow.com/a/11494872/1375574) for instructions.

Windows users will need to use [Cygwin](http://cygwin.org/) to get `make`.

All systems require [xml2rfc](http://xml2rfc.ietf.org/).  This
requires [Python](https://www.python.org/).  The easiest way to get
`xml2rfc` is with `pip`.

Using a `virtualenv`:

```sh
$ virtualenv --no-site-packages venv
# remember also to activate the virtualenv before any 'make' run
$ source venv/bin/activate
$ pip install xml2rfc
```

To your local user account:

```sh
$ pip install --user xml2rfc
```

Or globally:

```sh
$ sudo pip install xml2rfc
```

xml2rfc depends on development versions of [libxml2](http://xmlsoft.org/) and
[libxslt1](http://xmlsoft.org/XSLT).  These packages are named `libxml2-dev` and
`libxslt1-dev` (Debian, Ubuntu) or `libxml2-devel` and `libxslt1-devel` (RedHat,
Fedora).

If you use markdown, you will also need to install `kramdown-rfc2629`,
which requires Ruby and can be installed using the Ruby package
manager, `gem`:

```sh
$ gem install kramdown-rfc2629
```

Some other helpful tools are listed in `config.mk`.
