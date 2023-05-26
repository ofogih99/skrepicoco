# skrepicoco
lecture 7
pragma solidity 0.8.0;

import "./BokkyPooBahsDateTimeLibrary.sol";

contract BirthdayPayout {

// store name,address teammates
// get name, addresses of teammates
// send ether to address teammates on command only by owner

    string _name;

    address _owner;

    Teammate[] public _teammates;

    struct Teammate {
        string name;
        address account;
        uint256 salary;
        uint256 birthday;
    }

    constructor() public {
        _name="max";
        _owner = msg.sender;
    }

    function addTeammate(address account,string memory name, uint256 salary,uint256 birthday) public onlyOwner {
        require(msg.sender != account,"Cannot add oneself");
        Teammate memory newTeammate = Teammate(name,account,salary, birthday);
        _teammates.push(newTeammate);
        emit NewTeammate(account,name);
    }

    function findBirthday() public onlyOwner returns(uint256){
        // it is a good idea to check whether therea are any teammates in the database
        require(getTeammatesNumber()>0,"No teammates in the database");
        // what are loops: https://www.youtube.com/watch?v=GwcisLY5avc
        // so basically we iterate over every Teammate in the _teammates array...
        for(uint256 i=0;i<getTeammatesNumber();i++){
        // and check their birthday
            if(checkBirthday(i)){
            // if the birthday is today - we return the user
                return uint256(i);
            }
        }
        // this will revert the transaction if no teammates with birthday are found
        revert("Noone found");
    }

    // this function is instead of sendToTeammate
    function birthdayPayout(uint256 index) public onlyOwner {
    	// send some money to the teammate
        sendToTeammate(index);
        // and emit a HeppyBirthday event(just in case)
        emit HappyBirthday(_teammates[index].name,_teammates[index].account);
    }

    function getDate(uint256 timestamp) view public returns(uint256 year, uint256 month, uint256 day){
        (year, month, day) = BokkyPooBahsDateTimeLibrary.timestampToDate(timestamp);
    }

    function checkBirthday(uint256 index) view public returns(bool){
        // day & month of today's date == day & month of birthday teammate
        //block.timestamp = 1671560040
        //birthday = 877126202
        //BokkyPooBahsDateTimeLibrary.timestampToDate();
        uint256 birthday = getTeammate(index).birthday;
        (, uint256 birthday_month,uint256 birthday_day) = getDate(birthday);
        uint256 today = block.timestamp;
        (, uint256 today_month,uint256 today_day) = getDate(today);

        if(birthday_day == today_day && birthday_month==today_month){
            return true;
        }
        return false;
    }

    function getTeammate(uint256 index) view public returns(Teammate memory){
        return _teammates[index];
    }

    function getTeam() view public returns(Teammate[] memory){
        return  _teammates;
    }

    function getTeammatesNumber() view public returns(uint256){
        return _teammates.length;
    }

    function sendToTeammate(uint256 index) public onlyOwner{
        payable(_teammates[index].account).transfer(_teammates[index].salary);
    }

    function deposit() public payable{

    }

    modifier onlyOwner{
        require(msg.sender == _owner,"Sender should be the owner of contract");
        _;
    }

    event NewTeammate(address account, string name);

    event HappyBirthday(string name, address account);
}
