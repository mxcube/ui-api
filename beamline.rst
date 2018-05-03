Beamline API
~~~~~~~~~~~~

Functionality that incorporates several different instruments or that does
not necessarily belong to one particular instrument.



**API Functions**

.. code:: python

   class CmdTuple(NamedTuple):
    """
    Describes a "generic" beamline procedure, where:

    cmd_id: the string identifying the command, i.e. 'QUICK_REALIGN'
    running: True if the procedure is running otherwise False
    args: arguments to pass to the procedure
    display_name: the name to display in the user interface i.e. "Quick Realign"
    tool_tip: the tool tip
    messages: any initial messages to display
    """
        cmd_id: str
        state: CmdState
        args: tuple
        display_name: str
        tool_tip: str
        messages: list


    def get_procedures() -> List[CmdTuple]:
    """
    :returns: a list of tuples each describing a beamline procedure,
              such as: realignment, anneal

    :rtype: List[CmdTuple]
    """
        pass

        
    def run_procedure(cmd_id:str, args) -> bool:
    """
    Runs procedure identified by cmd_id passes args as its arguments
    
    (Errors and progress that occurs is passed asynchronously via the
     available signaling mechanism.)
    """
        
    
    def stop_procedure(cmd_id:str) -> bool:
    """
    Stops a running procedure identified by cmd_id

    (Signal emitted if stopped or on timeout waiting for procedure to stop)
    """


    class Actuator(NamedTuple):
    """
    Describes an actuator commonly a motor, that is considered to be
    "globally" available on the beamline. Such actuators could be shutters
    of different kinds, i.e: fast shutter, safety shutter, other in-out
    devices; beam stop, detector cover or even motors such as omega.

    name: A unique name that identifies this actuator
    value: The current position
    msg: A message string, explaining state or value
    state: The state of the actuator defined by ActuatorState
           (MOVING, READY, ERROR ...)
    """

        name: str
        value: float
        msg: str
        state: ActuatorState



    def get_actuators() -> Dict[str, Actuator]:
    """
    :returns: A dictionary with all available actuators where the key
              is the actuator name and the value the Actuator tuple
    :rtype dict:            
    """
        pass


    def get_actuator(name) -> Actuator:
    """
    :returns: The Actuator object identified by the given name
    :rtype: Actuator
    """
       pass


    def set_actuator(name, value) -> bool:
    """
    Tries to move the actuator identified by name to value

    (Errors and progress of movement is passed asynchronously
     via the available signaling mechanism)
    
    :returns: True if motion was started False otherwise
    """


    class BeamInfo(NamedTuple):
    """
    Describes the beam

    position: Beam position on the microscope view
    shape: Beam shape defined by BeamShape, i.e ELLIPSE, RECTANGLE
    vertical_size: Vertical size in microns
    horizontal_size: Horizontal in microns
    available_beam_sizes: list of tuples (float, float)
    """

        position: tuple(float, float)
        shape: BeamShape
        vertical_size: float
        horizontal_size: float
        available_beam_sizes: list
    

    def get_beam_info() -> BeamInfo:
        pass


    def set_beam_size(vertical_size:float, horizontal_size:float) -> bool:
    """
    Sets the beam size to vertical_size, horizontal_size the tuple
    (vertical_size, horizontal_size) must exist in available_beam_sizes
    returned by get_beam_info()

    (Errors setting beam size passed asynchronously via signals)
    """
        pass


    def prepare_beamline_for_sample():
    """
    Prepares the beamline for mounting a new sample
    """
       pass


**Signal handlers:**

Functions with the following signatures have to be provided by the specific UI Layer in order
to handle the various errors, state changes or simply progress messages that are sent by the
actions initiated by the functions above. These are the generic signals that can be sent by
a procedure or actuator, each of which can have their own specific signals that have to
be handled separately (should be documented with the corresponding procedure or actuator)

+---------------------------+---------------------------------------+
| Signal Name               | Handler                               |
+===========================+=======================================+
| procedureStateChanged     | procedure_state_changed_handler       |
+---------------------------+---------------------------------------+
| procedureValueChanged     | procedure_value_changed_handler       |
+---------------------------+---------------------------------------+
| actuatorStateChanged      | actuator_state_changed                |
+---------------------------+---------------------------------------+
| actuatorValueChanged      | actuator_value_changed_handler        |
+---------------------------+---------------------------------------+

.. code:: python

   def procedure_sate_changed_handler(CmdTuple) -> None:
   """
   Triggered when a procedure changes state
   """
   pass

   
   def procedure_value_changed_handler(CmdTuple) -> None:
   """
   Triggered when a procedure changes value, i.e. progress
   """
   pass


   def actuator_sate_changed_handler(CmdTuple) -> None:
   """
   Triggered when an actuator changes state
   """
   pass

   
   def actuator_value_changed_handler(CmdTuple) -> None:
   """
   Triggered when an actuator changes value, i.e. movement
   """
   pass
