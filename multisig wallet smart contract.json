# Solidity code for the MultiSigWallet contract:

pragma solidity ^0.8.0;

contract MultiSigWallet {
    uint private nonce;
    mapping(address => uint) private owners;
    uint private required;
    
    constructor(address[] memory _owners, uint _required) {
        require(_owners.length > 0 && _required > 0 && _required <= _owners.length, "Invalid arguments");
        
        for (uint i = 0; i < _owners.length; i++) {
            owners[_owners[i]] = 1;
        }
        
        required = _required;
    }
    
    function execute(address _to, uint _value, bytes memory _data, bytes[] memory _signatures) public {
        bytes32 txHash = keccak256(abi.encodePacked(nonce, _to, _value, _data));
        uint count = 0;
        
        for (uint i = 0; i < _signatures.length; i++) {
            address owner = recoverSigner(txHash, _signatures[i]);
            
            if (owners[owner] == 1) {
                owners[owner] = 2;
                count++;
            }
            
            if (count == required) {
                break;
            }
        }
        
        require(count == required, "Invalid signatures");
        
        (bool success,) = _to.call{value: _value}(_data);
        require(success, "Transaction execution failed");
        
        nonce++;
    }
    
    function recoverSigner(bytes32 _hash, bytes memory _signature) private pure returns (address) {
        require(_signature.length == 65, "Invalid signature length");
        bytes32 r;
        bytes32 s;
        uint8 v;
        
        assembly {
            r := mload(add(_signature, 32))
            s := mload(add(_signature, 64))
            v := byte(0, mload(add(_signature, 96)))
        }
        
        if (v < 27) {
            v += 27;
        }
        
        require(v == 27 || v == 28, "Invalid signature recovery");
        
        return ecrecover(_hash, v, r, s);
    }
}

# Solidity code for the MultiSigWalletFactory contract:

pragma solidity ^0.8.0;

import "./MultiSigWallet.sol";

contract MultiSigWalletFactory {
    mapping(uint256 => address) public multiSigWalletsCreatedTotal;
    mapping(address => uint256) public multiSigWalletsCreatedByUser;
    
   function deployMultiSigWithCreate(address[] memory _owners, uint256 _required) public {
    MultiSigWallet wallet = new MultiSigWallet(_owners, _required);
    multiSigWalletsCreatedTotal[uint256(wallet)] = address(wallet);
    multiSigWalletsCreatedByUser[msg.sender]++;
}

function deployMultiSigWithCreate2(address[] memory _owners, uint256 _required, bytes32 _salt) public {
    MultiSigWallet wallet = new MultiSigWallet(_owners, _required);
    address walletAddress = address(wallet);
    bytes memory code = type(MultiSigWallet).creationCode;
    assembly {
        let ptr := add(code, 0x20)
        mstore(ptr, _salt)
    }
    bytes32 saltHash = keccak256(abi.encodePacked(_salt));
    multiSigWalletsCreatedTotal[uint256(saltHash)] = walletAddress;
    multiSigWalletsCreatedByUser[msg.sender]++;
}

}
