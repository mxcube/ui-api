Sample changer API
==================


API Functions
-------------

These are the functions that make up the sample changer API

.. code:: python

    from sample_changer.GenericSampleChanger import (SampleChangerState, Sample)
    from typing import NewType

    # Mostly to facilitate documentation (to avoid repetition) but could also
    # be used in actual code.
    #
    # The ':' separated string that is used to describe the sample location i.e
    # cell:basket:sample (Should really be part of GenericSampleChanger)
    #
    LocationStr = NewType('LocationStr', str)

    class SampleNode(NamedTuple):
    """
    Represents a sample in a hierarchical layout that corresponds to the
    physical representation of the sample changer.

    Where state is one of the samples given by: SampleChangerState.tostring()
    """
        name: str
        state: str
        id: str
        selected: bool
        children: SampleNode


    class SampleTuple(NamedTuple):
    """
    Represents a sample in the sample list, a flat representation of all samples

    Where state is one of the samples given by: SampleChangerState.tostring()
    """
        id: str
        name: str
        location: LocationStr
        code: str
        lodable: bool
        state: str


    def to_sample_tuple(s: Sample) -> \
        SampleTuple[str, str, str, str, bool, str]:
    """
    Example using existing GenericSampleChanger to create a SampleTuple
    """
    return SampleTuple(s.getAddress(),
                       s.getAddress(), "Sample-%s" % s.getAddress(),
                       s.getID(),
                       True,
                       SampleChangerState.tostring(s.getState()))


    def get_sample_list() -> List[SampleTuple]:
    """
    :returns: the sample changer content often refered to as the "sample list"
    :rtype: List[SampleTuple]
    """
    pass


    def get_state() -> str:
    """
    :returns: the sample changer state, one of the strings defined in
              SampleChangerState

    :rtype: str (GenericSampleChanger.SampleChangerState)
    """
    pass


    def get_current_sample() -> SampleTuple:
    """
    :returns: the sample that is currenly loaded by the sample changer
    :rtype: SampleTuple
    """
    pass


    def get_sc_contents() -> SampleNode:
    """
    :returns: the hierarchical layout of the sample changer, with containers
              and samples.
    """
    pass


    def select_location(location:LoctionStr) -> bool:
    """
    Selects the sample at the given location

    :param LocationStr location: location
    :returns: True if location was selected otherwise False
    :rtype: bool
    """
    pass


    def scan_location(location:LocationStr) -> bool:
    """
    Scan the given location for contents

    :param LocationStr location: location
    :returns: True if any new content found otherwise False
    :rtype: bool
    """
    pass


    def mount_sample(location:LocationStr) -> bool:
    """
    Mounts sample from location

    :param LocationStr location: location
    :returns: True if mount successful otherwise False
    :rtype: bool
    """
    pass


    def unmount_current_sample(location:LocationStr=None) -> bool:
    """
    Un-mounts mounted sample to location, un mounts the sample
    to where it was last mounted from if nothing is passed

    :param LocationStr location: location
    :returns: True if un-mount successful otherwise False
    :rtype: bool
    """
    pass


    def get_full_state() -> Dict:
    """
    :returns: A dictionary containing the complete state of the sample changer

    The returned dict has the following format:

    {'state': GenericSampleChanger.SampleChangerState
     'loaded_sample': LocationStr
     'contents': SampleNode
     'procedures': "as returned by get_procedures",
     'msg': "user message if any"
    }

    :rtype: dict
    """

Specific procedures
-------------------

There will be a number of procedures that are beamline-specific, or that use
different parameters on different beamlines. These can be handled with the
get_procedures, get_procedure, run_procedure, and stop_procedure functions
(see the Procedures section under general for details). Procedures that
are used in the same way on different beamlines should preferably be promoted
to be part of the API, possibly with additional, beamline-specific parameters.

Possible example procedures are:
home, abort, defreeze, reset_sample_number, change_gripper,

Signal handling
---------------

Functions with the following signatures have to be provided by the specific UI Layer in order
to handle the corresponding signals. These functions could simply be implemented in a file
called for instance sc_signals.py or just signals.py and be attached automatically to the
corresponding signal name

    +---------------------+---------------------------------+
    | Signal Name         | Handler                         |
    +=====================+=================================+
    | stateChanged        | sc_state_changed_handler        |
    +---------------------+---------------------------------+
    | loadedSampleChanged | sc_loaded_sample_changed_handler|
    +---------------------+---------------------------------+
    | contentsUpdated     | sc_contents_update_handler      |
    +---------------------+---------------------------------+
    | cmdStateChanged     | sc_cmd_state_update_handler     |
    +---------------------+---------------------------------+
    | scError             | sc_error_handler                |
    +---------------------+---------------------------------+`

.. code:: python

   def sc_state_changed_handler(old_state:SampleChangerState,
                                new_state:SampleChangerState) -> None:
   """
   Triggered when the sample changer state changes
   """
   pass

   def loaded_sample_changed_handler(sample:Sample) -> None:
   """
   Triggered when a sample have been loaded
   """
   pass


   def sc_contents_update_handler(sample_node:SampleNode) -> None:
   """
   Triggered when sample_node or its contents have been updated.
   """
   pass


   def sc_procedure_update_handler(procedures:Tuple[str, ,...], message) -> None:
   """
   Triggered when the states of one or more procedures have been updated

   Note that get_procedures will get the entire set of procedures and their states
   """
   pass


   def sc_error_handler(error_code, message) -> None:
   """
   Triggered on any error
   """
   pass

**Usage example:**

Its up to the layer using this API to handle any errors that occur during execution
and perform the necessary actions, see the example of how it could be done in for instance
MXCuBE3

.. code:: python

    @mxcube.route("/mxcube/api/v0.1/sample_changer/mount", methods=["POST"])
    def mount_sample_clean_up():
        try:
            location = request.get_json()
            common_ui_api.sample_changer.mount(location)
        except Exception:
            return Response(status=409)
        else:
            return Response(status=200)
