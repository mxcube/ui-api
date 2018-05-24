Queue API
==================

This proposal is based on the mxcube3 routes/Queue.py


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
    class QueueFullState:
    """
    State of queue
    """
    current_sample: SampleTask
    centring_method: mxcube.CENTRING_METHOD
    auto_mount_next: SampleTask
    auto_add_diff_plan: bool
    num_snapshots: int
    group_folder: str
    queue: OrderedDict[LocationStr:SampleTask]
    sample_list: ??? # Is this necessary when you have the queue OrderedDict?
    queue_status: QueueState

    @dataclass
    class TaskNode:
        """
        Data for Task or sample node

        NB should probably be harmonised with queue_entry and queue_model
        """

        type: typing.ClassVar = 'Task'
        label: typing.ClassVar = 'Task'

        sample_ID: str
        node_ID: int = None
        parent_node_ID: int = None  # NB parent need not be a Sample, can be Interleave
        task_index: int = None
        checked: bool = False
        state: int = READY
        parameters: field(default_factory=dict)

    @dataclass
    class CentringTask(TaskNode):
        """
        Data for Centring node

        # NB it must be possible to add centrings to the queue,
        and to pass in motor positions as parameters,
        even if this will not happen in that way from the UI.
        """

        type: typing.ClassVar = 'Centring'
        label: typing.ClassVar = 'Centring'

    @dataclass
    class CharacterisationTask(TaskNode):
        """
        Data for Characterisation node
        """

        type: typing.ClassVar = 'Characterisation'
        label: typing.ClassVar = 'Characterisation'

        lims_result_data: dict = None
        diffraction_plan: DataCollectionTask = None

    @dataclass
    class DataCollectionTask(TaskNode):
        """
        Data for Data Collection node
        """

        type: typing.ClassVar = 'DataCollection'
        label: typing.ClassVar = 'Data Collection'

        lims_result_data: dict = None

    @dataclass
    class EnergyScanTask(TaskNode):
        """
        Data for EnergyScan node
        """

        type: typing.ClassVar = 'EnergyScan'
        label: typing.ClassVar = 'Energy Scan'

    @dataclass
    class InterleavedTask(TaskNode):
        """
        Data for Interleaved node
        """

        type: typing.ClassVar = 'Interleaved'
        label: typing.ClassVar = 'Interleaved'

        # tasks contain the interleaved wedges
        tasks: list = field(default_factory=list)

      @dataclass
      class MeshScanTask(TaskNode):
        """
        Data for Data Collection node
        """

        type: typing.ClassVar = 'MeshScan'
        label: typing.ClassVar = 'Mesh Scan'

        lims_result_data: dict = None

    @dataclass
    class SampleTask(TaskNode):
        """
        Data for Sample node
        """

        type: typing.ClassVar = 'Sample'
        label: typing.ClassVar = 'Sample'

        name:str = '?'
        code:str = '?'
        protein_acronym:str = '?'

        tasks: list = field(default_factory=list)

        # NB default_prefix, default_sub_dir, and others if desired are stored in parameters

    @dataclass
    class WorkflowTask(TaskNode):
        """
        Data for Workflow node
        """

        type: typing.ClassVar = 'Workflow'
        label: typing.ClassVar = 'Workflow'

        lims_result_data: dict = None

    @dataclass
    class XRFScanTask(TaskNode):
        """
        Data for XRFScan node
        """

        type: typing.ClassVar = 'XRFScan'
        label: typing.ClassVar = 'XRF Scan'


API Functions
-------------

These are the functions that make up the queue API,
including the task-specific functions to put
specific tasks on the queue.

# Queue administration functions

.. code:: python

    def start():
        """
        Start the queue running
        """
        pass

    def abort():
        """
        Abort the queue
        """
        pass

    def stop():
        """
        If a task is running, abort the task and pause the queue
        If no task is running, abort the queue
        """
        pass

    def pause():
        """
        Pause the queue
        """
        pass

    def unpause():
        """
        Unpause the queue
        """
        pass

    def clear():
        """
        Clear the queue
        """
        pass


    # NB do we need a similar or replacement function that takes a node_id as input?
    def execute_entry_with_id(sample_location:LocationStr, task_index:int):
        """
        Execute the entry at position (sample_location, task index) in queue

        :param LocationStr sample_location: sample_location
        :param int task_index: task index of task within sample at sample_location
        """
        pass

    # NB do we need a similar or replacement function that takes a list of node_ids as input?
    def delete_items(item_positions:List[Tuple[int, int]]):
        """
        Delete items in item_positions from queue

        :param List[Tuple[int, int]] item_positions: lit of (parent_node_id, task_index) tuples

        """
        pass

    def set_enabled_items(node_ids:List[int], is_enabled:bool=False):
        """
        en/disable queue nodes in node_ids list.

        :param List[int] node_ids: node_ids of nodes to en/disable
        :param bool is_enabled: value of is_enabled to set
        """
        pass

    def toggle_enabled(node_id:int):
        """
        Toggle enabled status for node node_id and recursively of contents.
        """
        pass

    def move_task_item(sample_location:LocationStr, from_task_index:int, to_task_index:int):
        """
        Move Sample task item in execution order from from_task_index to to_task_index

        :param LocationStr sample_location: sample_location
        :param int from_task_index: index of task to move
        :param int to_task_index: position to move task to
        """
        pass

    def swap_task_item(sample_location:LocationStr, from_task_index:int, to_task_index:int):
        """
        Swap Sample task item in execution order from from_task_index to to_task_index

        :param LocationStr sample_location: sample_location
        :param int from_task_index: index of task to swap
        :param int to_task_index: position to swap task to
        """
        pass

    def set_sample_order(order:List[LocationStr]):
        """
        reset sample order in queue

        :param List[LocationStr] order: New sample order
        """

# Queue action functions

.. code:: python

    def get_full_state() -> QueueFullState:
        """
        get complete state information for queue
        """
        pass

    def get_queue() -> OrderedDict[LocationStr:SampleTask]:
        """
        Get Ordered dictionary representation of Queue

        :returns: Ordered dict of sample_location:SampleTask
        :rtype OrderedDict[LocationStr:SampleTask]:
        """
        pass

    def add_nodes(tasks: List[TaskNode]):
        """
        Add queue nodes to queue, in position determined by their contents
        """

    def add_node(parent_node_id:int,task_node:TaskNode):
        """
        Add queue node to queue as child of task node.

        Dispatches to appropriate function depending on node type
        """
        new_task_node = create_node(parent_node_id, task_node.type)
        update_parameters(new_task_node, **get_default_parameters(task_node.type))
        update_parameters(new_task_node, **task_node.parameters)

    def create_node(parent_node_id, node_type):
        """ Create empty node of type node_type under parent defined by parent_node_id,
        and add it to the queue.
        """

    def get_default_parameters(node_type) -> dict:
        """
        Dispatcher function getting default parameter values for each task type.
        The get_default_xyz_parameters functions are part of the interface.
        Their return dict is not specified explictly, but is defined by the fact
        that update_xyz_parameters(node_id, **default_xyz_parameters)
        must create a completely populated default instance of the task.
        """
        if node_type == 'Sample':
            return get_default_sample_parameters()
        elif node_type == 'Characterisation':
            return get_default_characterisation_parameters()
        elif node_type == 'DataCollection':
            return get_default_datacollection_parameters()
        elif node_type == 'EnergyScan':
            return get_default_energy_scan_parameters()
        elif node_type == 'XRFScan':
            return get_default_xrf_scan_parameters()
        elif node_type == 'Workflow':
            return get_default_workflow_parameters()
        elif node_type == 'GphlWorkflow':
            return get_default_gphl_workflow_parameters()
        elif node_type == 'Centring':
            return get_default_centring_parameters()
        elif node_type == 'MeshScan':
            return get_default_mesh_scan_parameters()
        else:
            raise ValueError("Unknown node type: %s" % node_type)

    def update_parameters(task_node, **kwargs) -> dict:
        """
        Dispatcher function updating parameter values for each task type.

        The update_xyz_parameters functions are part of the interface.
        They must be defined so that all standard parameters are explicit,
        with a **kwargs argument to allow for beamline-specific arguments.
        This is necessary because it gives an official specification for
        which parameters are supported, which you would not get by
        simply passing a dictionary of unspecified content.

        All parameters must default to None and remain unchanged if no other
        value is given; this allows you to set all parameters to default and
        subsequently change only a few, if desired.
        """

        node_type = task_node.type
        node_id = task_node.node_id
        if node_type == 'Sample':
            update_sample_parameters(node_id, **kwargs)
        elif node_type == 'Characterisation':
            update_characterisation_parameters(node_id, **kwargs)
        elif node_type == 'DataCollection':
            update_datacollection_parameters(node_id, **kwargs)
        elif node_type == 'EnergyScan':
            update_energy_scan_parameters(node_id, **kwargs)
        elif node_type == 'XRFScan':
            update_xrf_scan_parameters(task_node, **kwargs)
        elif node_type == 'Workflow':
            update_workflow_parameters(node_id, **kwargs)
        elif node_type == 'GphlWorkflow':
            update_gphl_workflow_parameters(node_id, **kwargs)
        elif node_type == 'Centring':
            update_centring_parameters(node_id, **kwargs)
        elif node_type == 'MeshScan':
            update_mesh_scan_parameters(node_id, **kwargs)
        else:
            raise ValueError("Unknown node type: %s" % node_type)

    def set_queue_tasks(tasks: List[TaskNode]):
        """
        Make new queue and call add_nodes(tasks) on it

        :param  List[TaskNode] tasks: list of TaskNode
        """
        pass

    def update_lims_data_for_task(node_id:int):
        """
        Update lims data for task

        Renamed from 'get_lims_data_for_task', as it is not a getter function

        :param int node_id: node_id
        """
        pass

# Task-specific functions

.. code:: python

    def get_default_sample_parameters() -> dict:
        pass

    def get_default_characterisation_parameters() -> dict:
        pass

    def get_default_data_collection_parameters() -> dict:
        pass

    def get_default_energy_scan_parameters() -> dict:
        pass

    def get_default_mesh_scan_parameters() -> dict:
        pass

    def get_default_xrf_scan_parameters() -> dict:
        pass

    def get_default_workflow_parameters() -> dict:
        pass

    def get_default_gphl_workflow_parameters() -> dict:
        pass

    def get_default_centring_parameters() -> dict:
        pass


    # Functions to update task (queue_model_object) parameters
    #
    # NB All standard parameters must be added as param:typ=None
    # to each function
    # **kwargs is to support beamline-specific parameters
    def update_centring_parameters(node_id, ..., **kwargs)
    def update_characterisation_parameters(node_id, ..., **kwargs)
    def update_data_collection_parameters(node_id, ..., **kwargs)
    def update_energy_scan_parameters(node_id, ..., **kwargs)
    def update_gphl_workflow_parameters(node_id, ..., **kwargs)
    def update_mesh_scan_parameters(node_id, ..., **kwargs)
    def update_sample_parameters(node_id, ..., **kwargs)
    def update_workflow_parameters(node_id, ..., **kwargs)
    def update_xrf_scan_parameters(task_node, ..., **kwargs)


# Top Queue level attribute getter/setters

.. code:: python

    def get_group_folder() -> str:
        """
        getter for group_folder attribute

        :returns: group folder path
        :rtype: str
        """
        pass

    def set_group_folder(path:str):
        """
        setter for group_folder attribute

        :param str path: group folder path
        """
        pass

    def set_num_snapshots(count:int=4):
        """
        setter for num_snapshots attribute

        :param int count: number of snapshots to acquire
        """
        pass

    def set_auto_mount(auto_mount:bool):
        """
        set auto_mount attribute

        :param bool auto_mount:  If true automatically mount next sample
        """
        pass

    def set_auto_add_diffplan(auto_add:bool):
        """
        set auto_add_diffplan attribute

        :param bool auto_add:  If true automatically diffraction plan to queue
        """
        pass


# Unknown status (beamline specific??), requires discussion

.. code:: python

    def create_diff_plan(sample_location:LocationStr) -> DataCollectionTask:
        """
        :param LocationStr sample_location: location of queue sample to create plan for
        """
        pass

    def serialize
        """
        UNNECESSARY: seems to be an alias for get_queue
        """
