OPM Documentation
=======

See http://opm.readthedocs.org/


Compile the doc
-----------------------------------

* Install Sphinx :

``
	apt-get install python-sphinx
``

* Build :

``
	cd opm-doc
	sphinx-build -b html . /tmp/opm-doc/
``

Sphinx Theme
------------------------------------------------------------

Install the [https://github.com/snide/sphinx_rtd_theme](Read Te Doc) theme

``
        pip install sphinx_rtd_theme
``

And then add the following lines to the ``conf.py`` file:

``
import sphinx_rtd_theme
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
``
