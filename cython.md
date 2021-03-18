[Use Cython to get more than 30X speedup on your Python code](https://towardsdatascience.com/use-cython-to-get-more-than-30x-speedup-on-your-python-code-f6cb337919b6)

### setup.py example:
	from distutils.core import setup
	from Cython.Build import cythonize
	setup(ext_modules = cythonize('cy_script.pyx', language_level=3))

Run command to compile cpython script: `python setup.py build_ext --inplace`

