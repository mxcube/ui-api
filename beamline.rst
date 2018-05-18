Beamline API
============

Functionality that incorporates several different instruments or
that does not necessarily belong to one particular instrument.


List of Actuators:
------------------

Below a list of all generic actuators across the UI (see general.rst),
separated by data type.

NB The actuators could be subdivided by component (beamline, diffractometer, etc.),
but since the actuators can be accessed through the get_actuator(), get_actuators(),
and set_actuator_value() top-level functions, there is no need to locate them
within individual components. Component-specific access, such as

detector.set_detector_distance(value)

value = detector.get_detector_distance()

could be added for convenience, but are basically superfluous,
given that you can do

set_actuator_value('detector_distance', value)

value = get_actuator('detector_distance').value


**Actuators - Type float:**

    *Centring motors*

        Omega, Kappa, Phi, AlignmentX,  AlignmentY,  AlignmentZ,
        CentringX, CentringY, CentringZ,  Focus

        # NB the capitalized case is to distinguish them from the current lower-case names
             ('Phi' matches 'kappa_phi', not 'phi')

        # NB There are too many motor names here, and maybe we should shorten the list.
             It should be OK, though if the same motor is pointed to
             by two different names. The problems is how the names match.
             If you do not have 'CentringZ', which motor should you use during
             centring?
             Is 'Focus' always the same as 'AlignmentX'?
             Does 'CentringZ' always match the same alignment motor
             or does it depend on beamline geometry?

    *Detector motors*

        detector_distance, two_theta, detector_horizontal, detector_vertical

        detector_distance may be the only common one,
        but the others should be defined, so we have agreed names for the
        cases they are needed.

        # NB horizontal and vertical are kept separate,
             in case some beamlines have one without the other.

    *Beamline parameters*

        resolution, energy, wavelength, transmission

        # NB energy/wavelength and resolution/detector_distance are both needed.

    *Light intensity*

        frontlight_intensity, backlight_intensity

        The IN/OUT switching is covered below.

        # NB are these continuous-value floats,
             or rather allowed_values, or on/off?


**Actuators - Type Tuple[float, float]:**

    *Standard parameters*

        beam_size, beam_position

        These should be generally supported

    *Expert parameters*

        aperture, slits, beam_definer

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

**Actuators - Type TwoState**

    fast_shutter, safety_shutter, beamstop, capillary, frontlight, backlight

    These are all two-state actuators. Their value will be of type
    TwoStateValue. Their state is set to ActuatorState, though one could
    limit it to a subset: NOTINITIALIZED, UNUSABLE, READY, MOVING, FROZEN

    # NB If e.g. a beamstop has multiple positions, how should we treat it?
         It is open whether multistate objects (n> 2) should be handled similarly
         or should be done as TYP==str?

**Immovable actuators**

    - machine_current:float

    - photon_flux:float

    - fill_mode:str,

    - beam_divergence:Tuple[float,float]

    This is extending the concept of 'actuator' a bit.

    These are in practice unsettable by the UI, but they will be displayed,
    their values will be accessed and updated with value_changed signals,
    and they will have some similar states.
    (NOTINITIALIZED, UNUSABLE, FROZEN, possibly MOVING).
    I think it makes sense to treat them as actuators anyway, also because
    this is the way we want to treat normally settable actuators that are
    fixed only on some specific beamline (e.g. wavelength on a non-tunable
    beamline).


BeamInfo:
---------

.. code:: python

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

    # NB This should be a procedure
    def prepare_beamline_for_sample():
        """
        Prepares the beamline for mounting a new sample
        """
        pass
