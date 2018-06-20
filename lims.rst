LIMS API
========


Data structures
---------------

.. code:: python

    @dataclass
    class LIMSProposal:
    """
    """
        pass
        #TDB

    @dataclass
    class LIMSLaboratory:
    """
    """
        pass
        #TDB

    @dataclass
    class LIMSSession:
    """
    """
        pass
        #TDB



API Functions
-------------


.. code:: python

    from queue import DataCollectionTask

    def ping() -> bool:
        """
        Check if the LIMS server is responding

        :returns: True if responding False otherwise
        :rtype: bool
        """
        pass


    def get_proposal_by_username(str:username) -> LIMSProposal:
        """
        :param str username: username
        :returns: The proposal belonging to username
        :rtype: LIMSProposal
        """
        pass



    def get_proposal(str:code, str:number) -> LIMSProposal:
        """
        :param str code: Proposal code
        :param str number: Proposal number
        :returns: The proposal belonging to the proposal with code <code>
                  and number <number>
        :rtype: LIMSProposal
        """
        pass


   
    def get_local_contact(str: session_id) -> LIMSPerson:
        """
        :param str session_id: session id
        :returns: The local contact for session with session id <session_id>
        """
        pass


    def get_session(str: proposal_id) -> LIMSSession:
        """
        :param str proposal_id: proposal id
        :returns: The current session, if any, for proposal with id <proposal_id>
        """
        pass


    def get_sample_list(str: session_id) > List[LimsSample]:
        """
        :param str session_id: session id
        :returns: List of samples for session with id <session_id>
        :rtype: List[LimsSample]
        """
        pass


    def get_sample(str: sample_id) -> LimsSample:
        """
        :param str sample_id: 
        :returns: Sample with id <sample_id>
        :rtype: LimsSample
        """
        pass


    def get_task_url(str: task_id) -> str:
        """
        :param task_id: task id
        :returns: The URL for task with id <task_id>
        :rtype: str
        """
        pass


    def get_sample_url(str: sample_id) -> str:
        """
        :param str sample_id: sample id
        :returns: The URL for sample with id <sample_id>
        :rtype: str
        """
        pass


    def get_thumbnail(str: image_id) -> (str:name, Bytes: data):
        """
        :param str task_id: image_id
        :returns: Tuple (image name, binary data) of thumbnail with id <image_id>
        :rtype: Tuple[str, Bytes]
        """
        pass


    def get_data_collection_list(str: proposal_id, str:session_id) -> List[DataCollectionTask]:
        """
        :param str proposal_id: proposal id
        :param str session_id: session id
        :returns: List of DataCollectionTask items in session <session_id>
                  for proposal <proposal_id>
        """
        pass


    def get_data_collection(str: datacollection_id) -> DataCollectionTask:
        """
        :param str datacollection_id: datacollection id
        :returns: DataCollectionTask with id <datacollection_id>
        :rtype: DataCollectionTask
        """
        pass


    def get_quality_indicator_plot(str: task_id) -> (str:name, Bytes: data):
        """
        :param task_id: task id
        :returns: The quality indicator plot, Tuple (image name, binary data),
                  for task with id <task_id>
        :rtype: Tuple(str, Bytes)
        """
        pass
