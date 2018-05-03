Its up to the layer utilizing this API to handle any errors that occur during execution 
and perform the necessary actions, see the example of how it could be done in for instance
MXCuBE3

**Usage example:**

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


Sample changer API
~~~~~~~~~~~~~~~~~~

**Signal handlers:**

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
+---------------------+---------------------------------+

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
   

   def sc_contents_update_handler() -> None:
   """
   Triggered when the contents have been updated
   """
   pass


   def sc_command_update_handler(state_list, cmd_state, message) -> None:
   """
   Triggered when any of the command states have been updated
   """
   pass
   
   
   def sc_error_handler(error_code, message) -> None:
   """
   Triggered on any error
   """
   pass


**API Functions**

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


    def get_loaded_sample() -> SampleTuple:
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


    def unmount_sample(location:LocationStr) -> bool:
    """
    Un-mounts mounted sample to location, un mounts the sample
    to where it was last mounted from if nothing is passed

    :param LocationStr location: location
    :returns: True if un-mount successful otherwise False
    :rtype: bool
    """
    pass


    def get_available_commands() -> List[List]:
    """
    Retrieves a List (of Lists)  of sample changer specific commands, The List
    has the following structure:

    ["Command-Category_1", [
        ["cmd_1", "DisplayName", "Comment/Help-text"],
        ...
        ["cmd_n-1", "DisplayName", "Comment/Help-text"], 
        ["cmd_n", "DisplayName", "Comment/Help-text"], 
    ] 
    ], 
    ["Command-Category_n-1", [
        ["cmd_1", "DisplayName", "Comment/Help-text"],
        ...
        ["cmd_n-1", "DisplayName", "Comment/Help-text"], 
        ["cmd_n", "DisplayName", "Comment/Help-text"],
    ]
    ], 
    ["Command-Category_n",  [
        ...
    ] 
    ],  


    :Example:
    cmd_list = [
    ["Actions",  [
        ["home", "Home", "Actions"],
        ["defreeze", "Defreeze gripper", "Actions"],
        ["reset_sample_number", "Reset sample number", "Actions"],                   
        ["change_gripper", "Change Gripper", "Actions"],
        ["abort", "Abort", "Actions"],
      ]
    ]


    :returns: List, of lists, on the format described above
    :rtype List:
    """
    pass


    def get_command_state() -> List[List]:
    """
    Returns a list with the currently available commands, which might depend on
    sample changer state, and a status message:

    [{"cmd_1": True if available else False,
      "cmd_n": True if available else False,
      "cmd_n-1": True if available else False},
     message]

     Where message is a str


    :rtype: List
    """
    pass

    
    def get_full_state() -> Dict:
    """
    :returns: A dictionary containing the complete state of the sample changer

    The returned dict has the following format:

    {'state': GenericSampleChanger.SampleChangerState 
     'loaded_sample': LocationStr
     'contents': SampleNode
     'cmds': "as returned by get_available_commands",
     'commands_state': "as returned by get_command_state}",
     'msg': "user message if any"
    }

    :rtype: dict
    """


    def exec_command(cmd_name: str, *args, **kwargs) -> bool:
    """
    Executes the command cmd_name (one of the commands returned by 
    get_available_commands) with the args *args and **kwargs:

    :returns: True on successful execution otherwise False
    :rtype: bool
    """
    
