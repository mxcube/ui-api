# ui-api
Repository for specifying the API used between the MXCuBE user interface and its backend.
See [Background information](https://github.com/mxcube/HardwareRepository/issues/139) for more details.

The API consist of a set of functions and provides a "thin layer" for the MXCuBE user interface to access the underlying beamline control layer, today implemented as HardwareObjects.

The implementation of this specification will be done Python 2.7.x, migration to Python 3 will very likely occur in the future, but a time for the task have not yet been established. To facilitate the implementation process, the specification is written using __[Python3 with type hints](https://docs.python.org/3/library/typing.html)__. The API documentation is generated using __Sphinx__ and source code documentation should be written in [Sphinx compatible format](http://www.sphinx-doc.org/en/1.5.1/domains.html#the-python-domain)

The functions of the API will be divided into groups which are considered belonging together and each group will be documented in a separate file. The syntax used for these files is __reStructuredText__ since it integrates well with existing documentation tools used for Python. Each file corresponds to a specific part of the available "namespace" so each file can roughly be considered to become a python file in a concrete implementation. Data structures that are shared across the API are defined in [data_structures.rst](data_structures.rst), otherwise each .rst file corresponds to a category of functionality: [beamline.rst](beamline.rst]), [diffractometer.rst](diffractometer.rst), [lims.rst](lims.rst), [login.rst](login.rst), [processing.rst](processing.rst), [queue.rst](queue.rst), [remote_access.rst](remote_access.rst), [sample_changer.rst](sample_changer.rst), [workflow.rst](workflow.rst).

There are two types of functions those that are to be called directly by the client (UI) refereed to as __API functions__ and __signla handlers__ that are called by the underlying signal handling system.

The architecture is of client/server nature which means that some data structures are exchanged between client (UI) and server, its thus important that those data structures are easily serializable (to atleast JSON format).

A basic framework that provides access to the underlying control layer and for sending asynchronous data is assumed to exist, if needed. The control layer is accessed through a object called __"beamline"__ the mechanism for sending asynchronous data is referred to as __"async.emit"__.

Usage example:
--------------

Its up to the layer using this API to handle any errors that occur during execution and perform the necessary actions, see the example of how it could be done in for instance MXCuBE3

    @mxcube.route("/mxcube/api/v0.1/sample_changer/mount", methods=["POST"])
    def mount_sample_clean_up():
        try:
            location = request.get_json()
            common_ui_api.sample_changer.mount(location)
        except Exception:
            return Response(status=409)
        else:
            return Response(status=200)


__Example for setting beam size:__
```
def set_beam_size(vertical_size: float, horizontal_size: float) -> None:
"""
Sets the beam size.

:param float vertical_size: The vertical beam size in microns
:param float horizontal_size: The horizontal beam size in microns
:returns: None
"""
```
