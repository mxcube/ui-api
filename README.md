# ui-api
Repository for specifying the API used between the MXCuBE user interface and its backend

See [Background information](https://github.com/mxcube/HardwareRepository/issues/139) for more details.

The API consist of a set of functions and provides a "thin layer" for the MXCuBE user interface to access the underlying beamline control layer, today implemented as HardwareObjects.

The implementation of this specification will be done Python 2.7.x, migration to Python 3 will very likely occur in the future, but a time for the task have not yet been established. To facilitate the implementation process, the specification is written using __[Python3 with type hints](https://docs.python.org/3/library/typing.html)__. The API documentation is generated using __Sphinx__ and source code documentation should be written in [Sphinx comaptible format](http://www.sphinx-doc.org/en/1.5.1/domains.html#the-python-domain)

Example for setting beam size:
```
def set_beam_size(vertical_size: float, horizontal_size: size) -> None:
```
A basic framework that provides access to the underlying control layer and for sending asynchronous data is assumed to exist, if needed. The control layer is accessed through a object called __"beamline"__ the mechanism for sending asynchronous data is referred to as __"async.emit"__.

The functions of the API will be divided into groups which are considered belonging together and each group will be documented in a separate file. The syntax used for these files is __reStructuredText__ since it integrates well with existing documentation tools used for Python.
