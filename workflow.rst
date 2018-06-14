Workflow API
=====================

This proposal is based on the mxcube3 routes/workflow.py and the GPhL workflow


Data structures
---------------

NB the dataclasses are Python 3.7 (see manual), but backported to at least 3.6

They are the best way to specify data structures, because they are typed,
compact to write and (unlike NamedTuple) support subtyping.
They have a todict function for serialisation.
The same functionality could be done (more laboriously) with normal classes
or with multiple (non-inheriting) NamedTuples.
They are used as a good way of specifying functionality, without prejudice
to the implementation we shall eventually decide on.

.. code:: python

    @dataclass
    class WorkflowData:
        """
        Data specifying of a single workflow
        """
        name: str               # Workflow identifier
        display_name: str=None  # Heading/name to display
        requires: str=None      # requirement for workflow to run
        documentation: str=None # Documentation/tooltip
        parameters: dict=None   # Parameters passed to Workflow engine to set up UI
                                # May be control parameters or initial values for UI data entry.

    
    # Standard supported types of data, used to  widget, datatype, etc., with the corresponding DataClass
    # Additional types can be used. E.g. GPhL will have a selection_table type to display and select indexing solutions
    STANDARD_DISPLAY_TYPES = {'int':NumericFieldData, 
                              'float':NumericFieldData, 
                              'line':StringFieldData,
                              'text':StringFieldData,
                              'message':StringFieldData
                              'text_selector':SelectorFieldData,
                              'file':FieldData,
                              'bool':FieldData}
    
    
    @dataclass
    class FieldData:
        """
        Data specifying a data entry field for the workflow data dialog
        """
        name: str               # parameter name
        display_type: str       # String specifying the kind of data to display. Determines both the data type and the widget to use
        display_name: str=None  # Label/name to display
        value: object=None      # (Starting) value of parameter, to display
        tool_tip: str=None      # Short documentation string, for display or popup
        required: bool=False    # Is a return value mandatory?
        parameters: dict=None   # Dictionary for additional parameters
    
    
    @dataclass
    class NumericFieldData(FieldData):
        """
        Data specifying a numeric data entry field for the workflow data dialog
        """
        unit: str               # Suffix for data field, used to show the unit.
        upper_bound: float      # Upper bound for permitted values
        lower_bound: float      # Lower bound for permitted values
        
    @dataclass
    class StringFieldData(FieldData):
        """
        Data specifying a string data entry field for the workflow data dialog
        """
        read_only: bool          # Is field read-only? Yes/no.
        
    @dataclass
    class SelectorFieldData(FieldData):
        """
        Data specifying a string selector (e.g. pulldown) data entry field for the workflow data dialog
        """
        allowed_values: List[str] # List of allowed string values.
        
        

Functions
---------

.. code:: python

    def set_values_map(workflow_engine:str, parameters:Dict[str, object]):
        """ Return call from the 'workflowParametersDialog' signal. 
        Set queried values for workflow engine to parameters dict. 
        
        :param str workflow_engine: Name of workflow engine. value one of 'EDNA', 'GPhL'
        :param Dict[str, object] parameters: Parameters to pass
        """
        pass
        
    def get_available_workflows(workflow_engine:str) -> OrderedDict[str, WorkflowData]:
        """ Get available workflow information for workflow_engine. 
        
        :param str workflow_engine: Name of workflow engine. value one of 'EDNA', 'GPhL'
        :returns: Ordered dictionary of workflow_name and correspoinding WorkflowData
        :rtype: OrderedDict[str, WorkflowData]
        """
        pass
    
    
        

Signal handlers:
----------------

    Functions with the following signatures have to be provided by the specific
    UI Layer 

    +---------------------------+---------------------------------------+
    | Signal Name               | Handler                               |
    +===========================+=======================================+
    | workflowParametersDialog  | workflow_parameters_dialog            |
    +---------------------------+---------------------------------------+

.. code:: python

    def workflow_parameters_dialog(display_name:str, workflow_engine:str, fields:List[FieldData]) -> None:
        """Triggered when a workflow parameters are queried. 
        Pops up a query dialog for workflow_engine, which calls set_values_map when done."""
        pass
