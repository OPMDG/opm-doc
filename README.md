OPM Documentation
=================

This repository is the source of the [the official
documentation](http://opm.readthedocs.io/).


Compile the documentation
-------------------------

The documentation is written using Sphinx, using reStructuredText format.

In order to compile the documentation, you need sphinx.  On Debian/Ubuntu
distrubutions, you can install it using this command:

```bash
sudo apt-get install python-sphinx
```

If you're familiar with the `pip` command tool, you can simply install the
needed tools with this command:

```bash
pip install -r requirements
```

Once you have sphinx available, you can compile the documentation with the
following command:

```bash
make html
```

The documentation will be available in the `_buil/html/` directory.
