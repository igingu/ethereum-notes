### What is this in solidity
This is the current contract: address(this); address(this).balance; this.mint.selector; this.externalFunction();

### To call another contract
* either have an interface of it
* either have it's code
* either use "call" and abi.encodeWithSignature for the function signature
* ```function examples(address _test) external {
        ITestInterface(_test).inc();
        uint count = ITestInterface(_test).count();
    }```

### Modifiers[](https://docs.soliditylang.org/en/latest/cheatsheet.html#modifiers "Permalink to this heading")
-   `pure` for functions: Disallows modification or access of state.
-   `view` for functions: Disallows modification of state.
-   `payable` for functions: Allows them to receive Ether together with a call.
-   `constant` for state variables: Disallows assignment (except initialisation), does not occupy storage slot.
-   `immutable` for state variables: Allows exactly one assignment at construction time and is constant afterwards. Is stored in code.
-   `anonymous` for events: Does not store event signature as topic. Normal events have 4 topics, one being the event signature, the others being the 3 possible index fields. Anonymous events have 4 index fields.
-   `indexed` for event parameters: Stores the parameter as topic.
-   `virtual` for functions and modifiers: Allows the function’s or modifier’s behaviour to be changed in derived contracts.
-   `override`: States that this function, modifier or public state variable changes the behaviour of a function or modifier in a base contract.

### Abstract vs Interface
Abstract contracts can have implementations for functions. Contract should be abstract if it has at least one unimplemented function.

### Optimizations
* Bytes are packed better in structs. Using uints other than uint256 only saves memory in a struct. They should be put next to each other.
* Remove things using delete: delete mapping[id]

### View and pure functions
View and pure functions are only free when called externally.

### Default state variables and functions visibility
State variables are internal by default, and functions are public by default.

### Declaring a variable public
If you declare a variable public (uint256 public number), it automatically created a getter function for it (contract.number());

### Virtual and override
Virtual specifies function can be overriden by child contract. Override specifies contract overrides base contract's function.

### Msg.sender vs tx.origin
Tx.origin refers to original external account that initialized the transaction.
Msg.sender refers to the immediate account (external or other contract) that invokes the function.

### EVM data stores
In the EVM, there are 3 places to store data:
* in the stack (PUSH1 0x60 => 0x6060 because PUSH1 has opcode 60 as well)
* in the RAM (MSTORE opcode => 0x52)
* in the disk storage (SSTORE opcode) which is the most expensive

### Arrays
* Fixed size arrays have size determined when the array is created/allocated (uint[5] fixedArray).
* Dynamic arrays are random access, variable-size data structures, not predetermined at the point of declaration (uint[] dynamicArray);
* bytes is similar to bytes1[], but it is packed tighter in memory. string is equal to bytes, but string does not allow length or index access. Always use bytes1 to bytes32 if length is limited, they are cheaper.

### Mappings 
* key can be any built-in type, bytes, strings or any contract or enum type but not reference types.
* can only be in storage.
* don't have length.

### Fallback vs Receive
Receive function executes when calldata in transaction is empty, and any ETH value is supplied.
Fallback function executes when calldata in transaction doesn't match any function signature (not even receive).
Receive must be external payable, doesn't have arguments or return anything.
Fallback must be external, doesn't have arguments or return anything.

### Function selectors
Function selectors are 4 bytes long. 

### Base fee
Fee necessary to include transaction on the blockchain.

### gasleft 
gasleft returns (uint256) returns amont of gas left.

### msg variable
* msg.value = amount of ETH sent in WEI
* msg.sender = caller of transaction
* msg.data = complete transaction calldata
* msg.sig = first 4 bytes of calldata (the function signature or selector)
msg.sig is the selector of the function we are currently in.

### tx variable 
* tx.origin = origin of transaction
* tx.gasPrice = gas price of transaction

### address type variables 
* address.balance
* address.code 
* address.codehash
* address payable.send (returns false on failure)
* address payable.transfer (throws on failure)

### new keyword
new is used to deploy and create a new smart contract instance. Returns new contract's address.

### Constructors and other misc
* Solidity constructors are optional. If not set, contracts have a default constructor. 
* Constructors can be internal or public. 
* Inheritance must be specified left-to-right, most base contract to most derived.
* Override without base virtual => TypeError. Putting virtual in base class, and removing override from derived class => TypeError.

### How to send ether 
* transfer(2300 gas, throws error) - address.transfer(msg.value)
* send(2300 gas, returns bool) - address.send(msg.value)
* call(forwards all gas or set gas, returns bool) - address.call{value: msg.value}("")
* recommended to use call + reentrancy guard

### type stuff
* `type(C).name` (`string`): the name of the contract
* `type(C).creationCode` (`bytes memory`): creation bytecode of the given contract, see [Type Information](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#meta-type).
-   `type(C).runtimeCode` (`bytes memory`): runtime bytecode of the given contract, see [Type Information](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#meta-type).
-   `type(I).interfaceId` (`bytes4`): value containing the EIP-165 interface identifier of the given interface, see [Type Information](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#meta-type).
-   `type(T).min` (`T`): the minimum value representable by the integer type `T`, see [Type Information](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#meta-type)
-   `type(T).max` (`T`): the maximum value representable by the integer type `T`, see [Type Information](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#meta-type)

### Value Types
* Signed Integers (int = int256, int8, int16, int32, - 2 ^ 255 - 2 ^ 255)
* Unsigned Integers (uint = uint256, uint8, uint32, - 2 ^ 256 + 1, 2 ^ 256 - 1)
* Boolean (false, true)
* Addresses 
	* (holds up to 20 bytes = 160 bits, the size of Ethereum addresses)
	* can do .balance
	* address payable can .send and .transfer
* Enums
	* enum BooleanChoice{ False, True } (0, 1 actually)
* Bytes 
	* 8 bit signed integers
	* bytesX, with 1 <= X <= 32

### Referece Types
* Fixe-sized arrays
* Dinamically-sized arrays
	* Array members (length, push, pop)
* Byte arrays (bytes type)
* String arrays (are dynamic, but don't have length, push, pop. Need to be converted to byte arrays to be iterated.)
* Structs  
* Mappings

### Salt

"When creating a contract, the address of the contract is computed from the address of the creating contract and a counter that is increased with each contract creation.

If you specify the option salt (a bytes32 value), then contract creation will use a different mechanism to come up with the address of the new contract: It will compute the address from the address of the creating contract, the given salt value, the (creation) bytecode of the created contract and the constructor arguments."

### Try catch 
* allows you to react to failed external calls or contract creation calls, can not use it for internal function calls
* catch Error(string memory reason) for revert and require
* catch Error (bytes memory reason) for assert

### Libraries
* embedded into the contract if all library functions are internal
* if there are functions other than internal, it must be deployed and linked

### Storage vs Memory vs Calldata
Very important: https://medium.com/coinmonks/ethereum-solidity-memory-vs-storage-which-to-use-in-local-functions-72b593c3703a
Data location can only be specified for string, array, struct or mapping types.

Storage is where all state variables are stored. Storage variables are mutable, their location is persistent (on the blockchain). They are arranged in a compact way, with multiple variables occupying the same storage slot if possible (variables are packed together in 32 bytes slots, outside of dynamically sized arrays and structs). Dynamic arrays and structs always occupy a new slot. Constant state variables are not stored, but replaced with their constant values after transformation into bytecode.

State variables in `storage` are arranged in a compact way — if possible, multiple values will occupy the same `storage` slot. Besides the special cases of dynamically sized arrays and structs, other variables are packed together in blocks of 32 bytes.

If these variables are less than 32 bytes, they will be combined to occupy the same slot. Otherwise, they will be pushed onto the next `storage` slot. The data is stored contiguously (ie, one after the other), starting from the `0` slot (to slots `1`, `2`, `3`, etc), in order of their declaration in the contract.

However, constant state variables are _not_ saved into a `storage` slot. Rather, they are injected directly into the contract bytecode — whenever those variables are read, the contract automatically switches them out for their assigned constant value.
Storage should be used for storage. Variables declared outside functions are by default storage.

Memory is reserved for variables inside the scope of a function, mutable within that function, only persist while the function is called. Solidity reserves 4 32 bytes slots for memory variables.

Arrays (bytes and strings) are by default in storage. I should use memory to declare them.

Calldata is an immutable, temporary location where function arguments are stored, behaves mostly like memory. Avoids unnecessary copies and makes sure data is unaltered.

For arrays in functions it is better to use calldata, saves gas. Calldata can only be used for function declaration, for external functions. Must be used for dynamic parameters of external function. Calldata is allocated by caller, memory is allocated by callee.

Assignments between storage or calldata, to memory always create a separate copy.
Assignments between memory to memory only create references.
Assignments between storage to storage also creates references.
Assignments between calldata and memory to storage create a copy.