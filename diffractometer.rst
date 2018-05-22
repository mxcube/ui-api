Diffractometer API
~~~~~~~~~~~~~~~~~~

**API Functions**

.. code:: python

    # ActuatorData as defined in beamline
    from beamline import ActuatorData

    # Decision Needed !
    # We need to discuss wether this method is needed or not
    # could ovelap with beamline.get_actuators
    def get_motors() -> Dict[str, ActuatorData]:
    """
    :returns: A dictionary with all available motors where the key
              is the motor name and the value the ActuatorData tuple
    :rtype dict:
    """
        pass


    # Decision Needed !
    def get_motor(name:str) -> ActuatorData:
    """
    :returns: The ActuatorData for motor identified by name
    :rtype: ActuatorData
    """
        pass


    # Decision Needed !
    def set_motor_value(name:str, value:Any) -> bool:
    """
    Tries to move the motor identified by name to value

    (Errors and progress of movement is passed asynchronously
     via the available signaling mechanism)
    
    :returns: True if motion was started False otherwise
    """
        pass


    # Move phase to beamline API ?
    def get_full_state(include_motors=False):
    """
    Returns the full state of the diffractometer:

    {"current_phase": phase
    """
        pass
