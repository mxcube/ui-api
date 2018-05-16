General API
===========

  Functionality (Procedures, Actuators) used generally across the interface.

    # NB I propose using the classes here globally across the interface.
    #    We need to decide if we want a single namespace for e.g. procedures,
    #    or we want to have nested dictionaries to divide them by component.



API Functions
-------------

.. code:: python

   from typing import *

   from enum import Enum

   **# PROCEDURES**

    class ProcedureData(NamedTuple):
        """
        Describes a "generic" beamline procedure, procedures that are not
        necessarily present on all beamlines.

        where:

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


    def get_procedures() -> OrderedDict[str,ProcedureData]:
        """
        :returns: an OrderedDict name:ProcedureData, each describing a procedure,
                  such as: realignment, anneal

        # NB we may want nested dictionaries if procedures are divided by component

        :rtype: Tuple[ProcedureData]
        """
        pass


    def run_procedure(name:str, *args, **kwargs) -> bool:
        """
        Runs procedure identified by name with mandatory arguments args
        and optional keyword arguments kwargs

        (Errors and progress that occurs is passed asynchronously via the
         available signalling mechanism.)

        # NB progress handling is still unresolved!

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

  **# ACTUATORS**

    class GenericActuatorData(NamedTuple):
        """
        Describes a generic actuator, in practice any attribute that could be
        modifiable on some beamline, that takes time to settle, and that can
        be said to have a state.

        The generic actuator (as given here) is shorthand description of several
        NamedTuples that have the same attributes with different parameter type,
        as indicated by the TYP (pseudo)parameter.

        Supported TYP are: float, Tuple[float, float], str, and Enum.
        More could be added at need.

        Using the same class to deal with continuous-value and enumerated floats.
        as well as settable and frozen attributes, allows you to use the
        same code and machinery on different beamlines, where things are
        implemented in different ways.

        # NB upper_limit and lower_limit are given separately to make it easier
        #    to support the pair-of-floats type.

        # NB value_list, if not empty, gives the allowed values.
        #    For TYP float a set_value will default to the closest value in the
        #    values_list.
        # In all other cases setting a disallowed value should throw ValueError.
        """

        name:str            # A unique name that identifies this actuator
        value:Optional[TYP] # The current position - could be None is some states.
        msg:Optional[str]   # A message string, explaining state or value
        state:ActuatorState # The state of the actuator
        upper_limit:Optional[TYP]   # Upper limit
        lower_limit:Optional[TYP]   # Lower limit
        value_list:Tuple[TYP]        # Tuple of allowed values


    def get_actuators() -> Dict[str, ActuatorData]:
        """
        :returns: A dictionary with all available actuators where the key
                  is the actuator name and the value the ActuatorData tuple
        :rtype dict:
        """
        pass


    def get_actuator(name) -> ActuatorData:
        """
        :returns: The ActuatorData object identified by the given name
        :rtype: ActuatorData
        """
        pass


    def set_actuator_value(name, value:Any) -> bool:
        """
        Tries to set the actuator identified by name to value.
        Setting a disallowed value will raise ValueError, with one exception:
        if the actuator takes a float value and has a non-empty values_list,
        the value will be set to the nearest value in the list.

        (Errors and progress of movement is passed asynchronously
         via the available signalling mechanism)

        :returns: True if motion was started False otherwise
        """
        pass

  **State/Value enumerations**

      class ActuatorState(Enum):

        # This enumeration should be limited to what the UI needs to know,
        # not what the motors might want to tell. These values may need fixing.
        # Why do we need MOVESTARTED, for instance?

        NOTINITIALIZED = 0  # Actuator has not yet been set up. value is None
        UNUSABLE = 1        # Actuator is not functional. value is None
        READY = 2           # Actuator is functional and ready to accept new moves.
        MOVESTARTED = 3     # Move has started. Why do we need this?
        MOVING = 4          # Actuator is moving and does not accept move orders.
                            # Value is defined but unstable.
        ONLIMIT = 5         # Value is on limit. Actuator accepts move orders (?)
        FROZEN = 6          # Actuator is functional, but cannnot be moved.
                            # value is defined, and may be modified by HO level.
                            # Needed for e.g. wavelength on non-tunable beamlines,
                            # machine_current, fill_mode.

      class TwoStateValue(Enum):

        # There are two states, with aliases, the ACTIVE/IN/CLOSED state
        # and the INACTIVE/OUT/OPEN
        # As a mnemonic, you could say that 1 is for when the object is
        # 'doing its job' (shutter closed, beamstop and frontlight in, ...)
        # That means that for collection you need beamstop IN, and frontlight OUT
        #
        # NB Do we need an (oxymoronic) third state, like UNUSABLE?

        INACTIVE = 0
        OUT = 0
        OPEN = 0

        ACTIVE = 1
        IN = 1
        CLOSED = 1


Specific Actuators:
-------------------

**Type float:**

    *Centring motors*

        Omega, Kappa, Phi, AlignmentX,  AlignmentY,  AlignmentZ,
        CentringX, CentringY, CentringZ,  Focus

        # NB, the capitalized case is to distinguish them from the current
        #     lower-case names ('Phi' matches 'kappa_phi', not 'phi')
        #
        # NB There are too many motor names here, and maybe we should shorten
        #    the list. It should be OK, though if the same motor is pointed to
        #    by two different names. The problems is how the names match.
        #    If you do not have 'CentringZ', which motor should you use during
        #    centring?
        #    Is 'Focus' always the same as 'AlignmentX'?
        #    Does 'CentringZ' always match the same alignment motor
        #    or does it depend on beamline geometry?

    *Detector motors*

        detector_distance, two_theta, detector_horizontal, detector_vertical

        # detector_distance may be the only common one,
        # but the others are clearly defined.
        # NB horizontal and vertical are kept separate, in case some beamlines
        #    have one without the other.

    *Beamline parameters*

        resolution, energy, wavelength, transmission

        # NB energy/wavelength and resolution/detector_distance are both needed.

    *Light intensity*

        frontlight_intensity, backlight_intensity

        # NB are these continuous-value floats, or rather values_list or on/off?


**Type Tuple[float, float]:**

    *Standard parameters*

        beam_size, beam_position

        # These should be generally supported

    *Expert parameters*

        aperture, slits, beam_definer

        # These are included to standardize the names, but the specific
        # beam-defining motors are likely to vary between beamlines.


**Type str:**

    *Enumerated strings*

        zoom, phase, centring_method, beam_shape

        # These must all have a values_list or enum
        # NB we should standardise the vocabulary. An enum??

**Type TwoState**

    fast_shutter, safety_shutter, beamstop, capillary, frontlight, backlight

    # These are all two-state actuators. Their value will be of type
    # TwoStateValue. Their state is set to ActuatorState, though one could
    # limit it to a subset: NOTINITIALIZED, UNUSABLE, READY, MOVING, FROZEN
    #
    # NB it is open whether multistate objects should be handled similarly
    #    or should be done as string enums?
    #
    # NB If e.g. a beamstop has multiple positions, how should we treat it?

**Immovable actuators**

    machine_current:float
    photon_flux:float
    fill_mode:str,
    beam_divergence:Tuple[float,float]

    # These are in practice unsettable by the UI, but I think it makes sense
    # to treat them as actuators anyway.
    # 1) they would be updated in the UI in much the same way, with
    #    'value_changed' signals, and some relevant states
    #    (NOTINITIALIZED, UNUSABLE, FROZEN, possibly MOVING)
    # 2) They are equivalent to actuators that are fixed only on some beamlines
    #    such as wavelength.


BeamInfo:
---------

    # These functions are Beam specific.
    # They no longer belong with the rest of the file, which is generic,
    # but are left here for comparison.
    #
    # get_beam_info would be one example of a component-specific multi-value getter.


    class BeamInfoData(NamedTuple):
        """
        Describes the beam

        position: Beam position on the microscope view
        shape: Beam shape defined by BeamShape, i.e ELLIPSE, RECTANGLE
        beam_size: (Horizontal, Vertical) size in microns
        available_beam_sizes: list of tuples (float, float)
        """

        position: tuple(float, float)
        shape: BeamShape
        vertical_size: float
        horizontal_size: float
        available_beam_sizes: list


    def get_beam_info() -> BeamInfoData:
        """
        :returns: Information regarding the beam
        :rtype: BeamInfoData
        """
        pass

    # NB This should be a procedure
    def prepare_beamline_for_sample():
        """
        Prepares the beamline for mounting a new sample
        """
       pass


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
    | procedureValueChanged     | procedure_value_changed_handler       |
    +---------------------------+---------------------------------------+
    | actuatorStateChanged      | actuator_state_changed                |
    +---------------------------+---------------------------------------+
    | actuatorValueChanged      | actuator_value_changed_handler        |
    +---------------------------+---------------------------------------+

  .. code:: python

    def procedure_state_changed_handler(ProcedureData) -> None:
        """Triggered when a procedure changes state"""
        pass

    def procedure_value_changed_handler(ProcedureData) -> None:
        """Triggered when a procedure changes value, i.e. progres"""
        # NB This will not work - there is no 'value' in ProcedureData
        #    We need to make some changes to allow for this use case
        pass


    def actuator_state_changed_handler(ActuatorData) -> None:
        """Triggered when an actuator changes state"""
        pass


    def actuator_value_changed_handler(ActuatorData) -> None:
        """Triggered when an actuator changes value, i.e. movement"""
        pass
