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
        kwargs: OrderdDict[str:Any] of arguments with their defaults
        display_name: the name to display in the user interface e.g. "Quick Realign"
        tool_tip: the tool tip
        messages: any initial messages to display
        category: Classification of procedures into types, for UI, e.g. 'Action', 'Query', ...  Optional
        """

        name: str   # NB This assumes that only one instance of a command can run at any time
        state: CmdState
        kwargs: OrderedDict[str, Any]
        display_name: Optional[str] # defaults to name
        tool_tip: Optional[str]
        messages: Tuple[str]
        category: Optional[str]


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

    class ActuatorData(NamedTuple):
        """
        ActuatorData pass configuration, state and value for an Actuator.
        The type of value, uppper_limit, lower_limit, and allowed_values
        depends on the specific actuator. For now, supported types are:
        float, Tuple[float, float], str, and Enum,
        but more could be added at need.

        Using the same structure to deal with continuous-value and enumerated
        floats, as well as settable and frozen attributes, allows you to use
        the same code and machinery on different beamlines, where things are
        implemented in different ways.

        # NB upper_limit and lower_limit are given separately to make it easier
        #    to support the pair-of-floats type.

        # NB allowed_values, if not empty, gives the allowed values.
        #    For type float a set_value must default to the closest value in
        #    the allowed_values.
        # In all other cases setting a disallowed value should throw ValueError.
        """

        name:str            # A globally unique name that identifies the actuator
        value               # The current position - could be None is some states.
        msg:Optional[str]   # A message string, explaining state or value
        state:ActuatorState # The state of the actuator
        upper_limit         # Upper limit
        lower_limit         # Lower limit
        allowed_values      # Tuple of allowed values

State/Value enumerations
------------------------

.. code:: python

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
