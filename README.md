OPM Documentation
=======

See http://opm.readthedocs.org/


Compile the doc
-----------------------------------

* Install Sphinx :

``
	apt-get install python-sphinx
``

* Install the [https://github.com/snide/sphinx_rtd_theme](Read Te Doc) theme

``
        pip install sphinx_rtd_theme
``


* Build :

``
	cd opm-doc
	sphinx-build -b html . /tmp/opm-doc/
``

