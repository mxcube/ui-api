LOGIN API
=========

Data structures
---------------

@dataclass
class LoginResult():
    """
    """
    code: str
    msg: str


class AuthType(Enum):
    """
    """
    LDAP = 0
    LIMS = 1


class LoginType(Enum):
    """
    """
    USER = 0
    PROPOSAL = 1


@dataclass
class SessionInfo():
    """
    """
    synchrotron_name: str
    beamline_name: str
    login_type: LoginType
    login_result: LoginResult
    proposal: LimsProposal
    session: LimsSession
    local_contact: LimsPerson
    user: LimsPerson
    laboratory: LimsProposal


API Functions
-------------

.. code:: python

    def authenticate() -> SessionInfo:
        """
        """
        pass

    def signout():
        """
        """
        pass

    def current_session_info() -> SessionInfo:
        """
        """
        pass

    def login_type() -> LoginType:
        """
        """
        pass

    def set_current_proposal(str:code, str:number):
        """
        Sets the current proposal to the proposal with code <code>
        and number <number>

        :param str code: Proposal code
        :param str number: Proposal number
        """
