General API
===========

  Functions and classes used generally across the interface.

Procedures
----------

.. code:: python

    """
    This code describes "generic" beamline procedures.
    Some procedures and their parameters will be part of the interface,
    but individual beamlines can add their own procedures, as well as
    additional parameters (with defaults) to existing ones.
    """

    from typing import *
    from enum import Enum

    class ProcedureData(NamedTuple):
        """

        # NB TODO awaiting clarification on args / kwargs

        name: the string identifying the command, i.e. 'QUICK_REALIGN'
        state: CmdState
        args: tuple of arguments to pass to the procedure
        kwargs: dict[str:Any] of keyword arguments with their defaults
        display_name: the name to display in the user interface i.e. "Quick Realign"
        tool_tip: the tool tip
        messages: any initial messages to display
        """

        name: str   # NB This assumes that only one instance of a command can run at any time
        state: CmdState
        args: tuple[str, ...]
        kwargs: dict[str, Any]
        display_name: Optional[str] # defaults to name
        tool_tip: Optional[str]
        messages: Tuple[str]


    def get_procedures() -> OrderedDict[str, ProcedureData]:
        """
        :returns: an OrderedDict name:ProcedureData, each describing a procedure,
                  such as: realignment, anneal

        # NB we may want nested dictionaries if procedures are divided by component
        """
        pass


    def get_procedure(name) -> ProcedureData:
        """
        :returns: The ProcedureData object identified by the given name
        """
        pass


    def run_procedure(name:str, *args, **kwargs) -> bool:
        """
        Runs procedure identified by name with mandatory arguments args
        and optional keyword arguments kwargs

        (Errors and progress that occurs is passed asynchronously via the
         available signalling mechanism.)

        """
        pass

    def stop_procedure(name:str) -> bool:
        """
        Stops a running procedure identified by name

        (Signal emitted if stopped or on timeout waiting for procedure to stop)

        # NB we are assuming that names (like 'Quick_Realign') are unique,
        #    and that a given procedure cannot run in two parallel processes.
        """
        pass

    class CmdState(Enum):
        # NB do we need other / different states here?
        UNUSABLE = 0
        READY = 1
        RUNNING = 2

Actuators:
----------

.. code:: python

    """
    Describes a generic actuator. This can include proper actuators (IN/OUT),
    movers (e.g. alignment motors), settable values (e.g. wavelength, energy),
    and in some cases values that are not settable (on a particular beamline)
    such as machine_current, fill_mode, or energy (on a non-tunable beamline)
    """

    from typing import *
    from enum import Enum

    class GenericActuatorData(NamedTuple):
        """
        GenericActuatorData (as given here) is an abstract superclass - as it were.
        In practice we would need separate classes that differ in using
        different actual type specifications instead of the generic TYP.

        Supported TYP are: float, Tuple[float, float], str, and Enum.
        More could be added at need.

        Using the same structure to deal with continuous-value and enumerated
        floats, as well as settable and frozen attributes, allows you to use
        the same code and machinery on different beamlines, where things are
        implemented in different ways.

        # NB upper_limit and lower_limit are given separately to make it easier
        #    to support the pair-of-floats type.

        # NB allowed_values, if not empty, gives the allowed values.
        #    For TYP==float a set_value must default to the closest value in
        #    the allowed_values.
        # In all other cases setting a disallowed value should throw ValueError.
        """

        name:str            # A globally unique name that identifies the actuator
        value:Optional[TYP] # The current position - could be None is some states.
        msg:Optional[str]   # A message string, explaining state or value
        state:ActuatorState # The state of the actuator
        upper_limit:Optional[TYP]   # Upper limit
        lower_limit:Optional[TYP]   # Lower limit
        allowed_values:Tuple[TYP]   # Tuple of allowed values


    def get_actuators() -> Dict[str, ActuatorData]:
        """
        :returns: A dictionary with all available actuators where the key
                  is the actuator name and the value the ActuatorData tuple
        """
        pass


    def get_actuator(name) -> ActuatorData:
        """
        :returns: The ActuatorData object identified by the given name
        """
        pass


    def set_actuator_value(name, value:Any) -> bool:
        """
        Tries to set the actuator identified by name to value.
        Setting a disallowed value will raise ValueError, with one exception:
        if the actuator takes a float value and has a non-empty allowed_values,
        the value will be set to the nearest value in the list.

        Setting a value of the wrong type will raise TypeError

        (Errors and progress of movement is passed asynchronously
         via the available signalling mechanism)

        :returns: True if motion was started False otherwise
        """
        pass

State/Value enumerations
------------------------

.. code:: python

    from typing import *
    from enum import Enum

    class ActuatorState(Enum):
        """
        This enumeration should be limited to what the UI needs to know,
        not what the motors might want to tell. These values may need fixing.
        """

        NOTINITIALIZED = 0  # Actuator has not yet been set up. value is None
        UNUSABLE = 1        # Actuator is not functional. value is None
        READY = 2           # Actuator is functional and ready to accept new moves.
        MOVING = 3          # Actuator is moving and does not accept move orders.
                            # Value is defined but unstable.
        ONLIMIT = 4         # Actuator is READY but value is on limit.
        FROZEN = 5          # Actuator is functional, but cannnot be moved.
                            # value is defined, and may be modified by HO level.
                            # Needed for e.g. wavelength on non-tunable beamlines,
                            # machine_current, fill_mode.

    class TwoStateValue(Enum):
        """
        There are two states, with aliases, the ACTIVE/IN/CLOSED state
        and the INACTIVE/OUT/OPEN
        As a mnemonic, you could say that 0 is for when the object is
        'doing its job' (shutter closed, beamstop and frontlight in, ...)
        That means that for collection you need beamstop IN, and frontlight OUT

        The official state name is (IN)ACTIVE, the other names are aliases.

        NB Do we need an (oxymoronic) third state, like UNUSABLE?
        """

        INACTIVE = 0
        OUT = 0
        OPEN = 0

        ACTIVE = 1
        IN = 1
        CLOSED = 1

Signal handlers:
----------------

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
    | procedureProgress         | procedure_progress_handler            |
    +---------------------------+---------------------------------------+
    | actuatorStateChanged      | actuator_state_changed                |
    +---------------------------+---------------------------------------+
    | actuatorValueChanged      | actuator_value_changed_handler        |
    +---------------------------+---------------------------------------+

.. code:: python

    def procedure_state_changed_handler(ProcedureData) -> None:
        """Triggered when a procedure changes state"""
        pass

    def procedure_progress_handler(procedure_name:str, value: Any,
                                   message:str='') -> None:
        """Handles progress-messages from running procedures"""
        pass

    def actuator_state_changed_handler(ActuatorData) -> None:
        """Triggered when an actuator changes state"""
        pass

    def actuator_value_changed_handler(ActuatorData) -> None:
        """Triggered when an actuator changes value, i.e. movement"""
        pass
