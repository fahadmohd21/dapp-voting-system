# dapp-voting-system
// SPDX-License-Identifier: MIT<br>
pragma solidity ^ 0.8.0;<br>
contract VotingSystem {<br>
    address public owner;<br>
    enum ElectionState { NOT_STARTED, ONGOING, ENDED } <br>
    ElectionState public electionState; <br>
    struct Candidate {<br>
        string name;<br>
        string proposal;<br>
        uint256 votes;
    }

    struct Voter {<br>
        string name;<br>
        address delegate;<br>
        bool hasVoted;<br>
        address votedForCandidate;
    }
    mapping(address => bool) public isAdmin;<br>
    mapping(address => bool) public isVoter;<br>
    mapping(address => Candidate) public candidates;<br>
    mapping(address => Voter) public voters;<br>

    uint256 public numCandidates;<br>
    uint256 public numVoters;<br>

    modifier onlyOwner() {<br>
        require(msg.sender == owner, "Only the owner can call this function");
        _;
    }

    modifier onlyAdmin() {<br>
        require(isAdmin[msg.sender], "Only admin can call this function");
        _;
    }

    modifier onlyBeforeElectionStart() {<br>
        require(electionState == ElectionState.NOT_STARTED, "Election has already started");
        _;
    }

    modifier onlyDuringElection() {<br>
        require(electionState == ElectionState.ONGOING, "Election is not ongoing");
        _;
          }

    modifier onlyAfterElectionEnd() { <br>
        require(electionState == ElectionState.ENDED, "Election is not ended yet");
        _;
    }

    constructor() {<br>
        owner = msg.sender;<br>
        isAdmin[owner] = true;<br>
        electionState = ElectionState.NOT_STARTED;
    }

    function addCandidate(string memory _name, string memory _proposal) external onlyAdmin onlyBeforeElectionStart {<br>
        address candidateAddress = address(uint160(uint256(keccak256(abi.encodePacked(msg.sender, block.number)))));<br>
        candidates[candidateAddress] = Candidate(_name, _proposal, 0);<br>
        numCandidates++;
    }

    function addVoter(address _voter, string memory _name) external onlyAdmin onlyBeforeElectionStart {<br>
        isVoter[_voter] = true;<br>
        voters[_voter] = Voter(_name, address(0), false, address (0));<br>
        numVoters++;
    }

    function startElection() external onlyOwner onlyBeforeElectionStart {<br>
        electionState = ElectionState.ONGOING;
    }

    function displayCandidateDetails(address candidateAddress) external view returns (address, string memory, string memory) {<br>
    Candidate storage candidate = candidates[candidateAddress];<br>
    return (candidateAddress, candidate.name, candidate.proposal);
    }
    function showWinner() external view onlyAfterElectionEnd returns (string memory, address, uint256) {<br>
    require(numCandidates > 0, "No candidates available");<br>
    address winnerAddress;<br>
    uint256 maxVotes = 0;<br>

    for (uint256 i = 0; i < numCandidates; i++) {<br>
        address candidateAddress = address(uint160(uint256(keccak256(abi.encodePacked(owner, i)))));<br>
        uint256 votes = candidates[candidateAddress].votes;<br>
        if (votes > maxVotes) {<br>
            maxVotes = votes;<br>
            winnerAddress = candidateAddress;
        }
    }

    Candidate storage winner = candidates[winnerAddress];<br>
    return (winner.name, winnerAddress, maxVotes);
}
    function delegateVotingRight(address delegateTo, address voterAddress) external onlyDuringElection {<br>
        require(isVoter[voterAddress], "Not a valid voter");<br>
        require(!voters[voterAddress].hasVoted, "Voter has already voted");<br>
        voters[voterAddress].delegate = delegateTo;
    }
    function castVote(address candidateAddress, address voterAddress) external onlyDuringElection {<br>
    require(isVoter[voterAddress], "Not a valid voter");<br>
    require(!voters[voterAddress].hasVoted, "Voter has already voted");<br>
    require(voters[voterAddress].delegate == address(0), "Voter has delegated the vote");<br>

    candidates[candidateAddress].votes++;<br>
    voters[voterAddress].hasVoted = true;<br>
    voters[voterAddress].votedForCandidate = candidateAddress;
}

    function endElection() external onlyOwner onlyDuringElection {<br>
        electionState = ElectionState.ENDED;
    }

   function showElectionResults(address candidateAddress) external view onlyAfterElectionEnd returns (address, string memory, uint256) {<br>
    Candidate storage candidate = candidates[candidateAddress];<br>
    return (candidateAddress, candidate.name, candidate.votes);
}
    

    function viewVoterProfile(address voterAddress) external view returns (string memory, address, bool) {<br>
        Voter storage voter = voters[voterAddress];<br>
        return (voter.name, voter.votedForCandidate, voter.delegate != address(0));
    }
}

      
