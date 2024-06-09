# SmartContractWallet

Using EIP/ERC-4337 Engineering a smart contract wallet through account abstraction such that to eliminate the use of EOA (externally owned accounts like metamask etc...)

## Having a Deterministic Address.

To have a deterministic Address, i used factory address + owner address + salt (any randomm int value) and combination of both give a deterministric address.
for e.g. as factory address is fixed hence for an owner with same value of salt his wallet address will be same.

```Solidity


    function createAccount(
        address owner,
        uint256 salt
    ) public returns (SmartContractWallet ret) {
        address addr = getAddress(owner, salt);
        uint256 codeSize = addr.code.length;

        // If the account is already created and deployed
        if (codeSize > 0) {
            return SmartContractWallet(payable(addr));
        }
        ret = SmartContractWallet(
            payable(
                new ERC1967Proxy{salt: bytes32(salt)}(
                    address(accountImplementation),
                    abi.encodeCall(SmartContractWallet.initialize, (owner))
                )
            )
        );
        getAccountAddress[owner].push(address(ret));
        allDeployedAddress.push(address(ret));
    }


    function getAddress(
        address owner,
        uint256 salt
    ) public view returns (address) {
        return
            Create2.computeAddress(
                bytes32(salt),
                keccak256(
                    abi.encodePacked(
                        type(ERC1967Proxy).creationCode,
                        abi.encode(
                            address(accountImplementation),
                            abi.encodeCall(
                                SmartContractWallet.initialize,
                                (owner)
                            )
                        )
                    )
                )
            );
    }
```

## UI component

<img width="1458" alt="image" src="https://github.com/PradeepSahhu/SmartContractsWallet/assets/94203408/924d600f-3b3e-48c8-91dd-51b06966c5a5">

As the factory address is same for every instance of Smart Contract Wallet hence, their is no need to take it as input.

Storing all the instances of a particular owner into an array of address such that i can connect to them later or use it to see which one's are made.

```Solidity
 address[] public allDeployedAddress;
mapping(address => address[]) public getAccountAddress;

```

here allDeployedAddress contains all the deployed smart contract wallet address and getAccountAddress contains all the wallet addresses of a particular owner.

<img width="1457" alt="image" src="https://github.com/PradeepSahhu/SmartContractsWallet/assets/94203408/e6e0852f-0a0e-49b8-a51c-582efa0ee690">

## Smart Wallet

Each smart wallet has certain state variables and functions. Some of the most important are:

- SendTransaction

```Solidity

 function transferTo(address _transferTo, uint amount) public {
        (bool callMsg, ) = _transferTo.call{value: amount}("");
        require(callMsg, "Transfer unsuccessfull");
    }
```

- Only Owner function and modifier. (efficient for gas optimization)

```Solidity

   modifier onlyOwner() {
        _onlyOwner();
        _;
    }

  function _onlyOwner() internal view {
        //directly from EOA owner, or through the account itself (which gets redirected through execute())
        require(
            msg.sender == owner || msg.sender == address(this),
            "only owner"
        );
    }
```

- Check Balance

```Solidity
    function checkBalance() public view returns (uint) {
        return address(this).balance;
    }
```

- Change , set, validate Password

```Solidity

      function setPassword(string memory _password) public onlyOwner {
        password = keccak256(abi.encode(_password));
    }

    function login(string memory _data) internal view returns (bool) {
        bytes32 message = keccak256(abi.encode(_data));
        require(message == password, "You are not the owner");
        return true;
    }

    function changeOwner(string memory _password) public {
        bool res = login(_password);
        require(res, "You are not authorized");
        owner = msg.sender;
    }

```

## UI Component

User Interface of each Wallet.

<img width="1467" alt="image" src="https://github.com/PradeepSahhu/SmartContractsWallet/assets/94203408/f47ac205-b252-4fbd-ae88-62c02fb06ad2">

### Send Component

<img width="1419" alt="image" src="https://github.com/PradeepSahhu/SmartContractsWallet/assets/94203408/307ed4c8-93de-4c67-97d4-5d00aaf71af5">

### Receive Component (QRCode & clipboard address)

<img width="1418" alt="image" src="https://github.com/PradeepSahhu/SmartContractsWallet/assets/94203408/db1be44c-661f-4c67-8836-d85b561218c7">

### Validate User.

<img width="1291" alt="image" src="https://github.com/PradeepSahhu/SmartContractsWallet/assets/94203408/252473c3-0da4-494c-84ff-40068ace40e6">
