---
title: Python Development with Atom
date: 2017-09-07T05:55:59
feature_image: https://images.unsplash.com/photo-1517134191118-9d595e4c8c2b?ixlib=rb-0.3.5&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzczfQ&s=c52b0953fc1a1d157d35a4eba1fd7b59
tags:
  - development
  - tech
languages:
  - en
---

Over the years my Development Environments have changed quite a lot. When I started writing scripts in Bash and Perl, I used [VIM](https://vim.sourceforge.io/). As I used Linux on the Desktop, and my scripts were not that complex, it worked great at the time. After that, I tried [Komodo](https://www.activestate.com/komodo-ide) and ended up using [Eclipse](http://www.eclipse.org/) with the Perl plugins. Eclipse always felt a bit bloated and oversized and I was always happy when I did not have to use it.

For a long time I stuck to VIM and had one or two Terminal windows open on the side to run scripts and a browser to look up the documentation. Now that my scripts and modules get more complex, I wanted to take another look and see if anything has changed in the world of IDEs in the last couple of years.

This coincided with my switch from VIM to [Atom](https://atom.io/) as my default editor, so naturally I looked at Atom’s plugins first. And I liked what I found! I decided to start with the following packages:

* [linter-flake8](https://atom.io/packages/linter-flake8)
* [linter-python-pep8](https://atom.io/packages/linter-python-pep8)
* [autocomplete-python](https://atom.io/packages/autocomplete-python)
* [django-templates](https://atom.io/packages/django-templates)
* [python-debugger](https://atom.io/packages/python-debugger)

Since it is never a good idea to install multiple [Python](https://www.python.org/) versions and the required modules directly to the system, installing [Virtualenv](https://virtualenv.pypa.io/en/stable/) is a must! I also discovered [Virtualwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/index.html), which makes creating and managing virtual environments much easier!

Python on Mac OS X is quite old, and I do not want to mess it up by accident, so I always install Python and [pip](https://pypi.python.org/pypi/pip) via [Homebrew](https://brew.sh/).

#### Use pip to install virtualenv and the wrapper

```shell
pip install virtualenv virtualenvwrapper
```

#### Export Python and VirtualEnv settings (.bashrc)

```bash
export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.5
export WORKON_HOME="${HOME}/venv"
export PROJECT_HOME="${HOME}/devel"
source /usr/local/bin/virtualenvwrapper.sh
```

#### Install Atom packages with all requirements

```shell
$ pip install pep8 flake8 jedi flake8-docstrings
$ apm install linter-flake8 linter-python-pep8 autocomplete-python django-templates python-debugger
```

It is great to have a style guide directly in Atom! It is also awesome to be able to use the Python Debugger and go step by step through the code, or auto-complete functions, arguments and variables. All these little packages are a tremendous help. I think, I will stick to this for a while.
