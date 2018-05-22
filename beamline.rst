Beamline API
============

Functionality that are considered to belong to the beamlone as a whole,
incorporating several different instruments or that does not necessarily belong
to one particular instrument.


List of Actuators:
------------------

Below a list of actuators that, from an instrumentation perspective, are
considered to belong to the beamline as a whole. Such actuators include, but
are not limited to, actuators that have an effect on properties directly
associated with the beamline. Such properties are for instance resolution,
energy and resolution.


**Actuators - Type float:**

    *Centring motors*

        omega, kappa, kappa_phi, alignment_ver, alignment_hor, alignment_depth,
        centring_vertical, centring_horizontal, centring_depth, focus

    *Detector motors*

        detector_distance, two_theta, detector_horizontal, detector_vertical

        detector_distance may be the only common one, but the others are
        defined for future needs.

    *Beamline parameters*

        resolution, energy, wavelength, transmission

    *Light intensity*

        frontlight_intensity, backlight_intensity


**Actuators - Type Tuple[float, float]:**

    *Standard parameters*

        beam_size, beam_position

    *Expert parameters (optional)*

        aperture, slits

        #
        # Question: How exactly would these work ?
        #

        These are included only to standardize the names. The specific
        beam-defining motors are likely to vary between beamlines,
        and many beamlines will not support these in the user interface,
        while others would use more or different actuators.


**Actuators - Type str:**

    *Enumerated strings*

        zoom, phase, centring_method, beam_shape

        These must all have a allowed_values or enum

        # NB we should standardise the vocabularies as well as the names.
             Enums?? How should we deal with beamline-specific sets?

        # The underlaying "object" that provides the functionality could provide
        # a suitable enum to go with the object ?


**Actuators - Type TwoState**

    fast_shutter, safety_shutter, beamstop, capillary, frontlight, backlight

    These are all two-state actuators. Their value will be of type
    TwoStateValue. Their state is set to ActuatorState, though one could
    limit it to a subset: NOTINITIALIZED, UNUSABLE, READY, MOVING, FROZEN

    # NB If e.g. a beamstop has multiple positions, how should we treat it?
         It is open whether multistate objects (n> 2) should be handled similarly
         or should be done as TYP==str?

    # A beamstop is traditionally in or out, if it has more states it have to be
    # of another type


**Immovable actuators**

    - machine_current:float

    - photon_flux:float

    - fill_mode:str

    - beam_divergence:Tuple[float,float]


API Functions
-------------

.. code:: python

    from dstruct import (ActuatorData, ProcedureData)


    def get_procedures() -> OrderedDict[str, ProcedureData]:
        """
        :returns: an OrderedDict name:ProcedureData, each describing a
                  procedure, such as: realignment, anneal
        """
        pass


    def get_procedure(name:str) -> ProcedureData:
        """
        :returns: The ProcedureData object identified by the given name
        """
        pass


    def run_procedure(name:str, **kwargs) -> bool:
        """
        Runs procedure identified by name with arguments kwargs

        (Errors and progress that occurs is passed asynchronously via the
         available signaling mechanism.)
        """
        pass


    def stop_procedure(name:str) -> bool:
        """
        Stops a running procedure identified by name

        (Signal emitted if stopped or on timeout waiting for procedure to stop)
        """
        pass


    def get_actuators() -> Dict[str, ActuatorData]:
        """
        :returns: A dictionary with all available actuators where the key
                  is the actuator name and the value the ActuatorData tuple
        """
        pass


    def get_actuator(name:atr) -> ActuatorData:
        """
        :returns: The ActuatorData object identified by the given name
        """
        pass


    def set_actuator_value(name:str, value:Any) -> bool:
        """
        Tries to set the actuator identified by name to value.

        Setting a disallowed value will raise ValueError.
        Setting a value of the wrong type will raise TypeError

        (Errors and progress of movement is passed asynchronously
         via the available signaling mechanism)

        :returns: True if motion was started False otherwise
        """
        pass


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
        This is an example of a domain-specific multi-value getter function

        :returns: Information regarding the beam
        """
        pass


    def prepare_beamline_for_sample():
        """
        Prepares the beamline for mounting a new sample
        """
        pass


Signal handlers:
----------------

    Functions with the following signatures have to be provided by the specific
    UI Layer in order

    to handle the various errors, state changes or simply progress messages that
    are sent by the actions initiated by the functions above. These are the
    generic signals that can be sent by a procedure or actuator, each of which
    can have their own specific signals that have to be handled separately
    (should be documented with the corresponding procedure or actuator)

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
