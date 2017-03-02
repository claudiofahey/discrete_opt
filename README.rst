
Contributor Installation
------------------------


conda install Sphinx numpydoc
sphinx-quickstart

pip install -e
pip install twine

python setup.py sdist bdist_wheel
twine upload dist/*

