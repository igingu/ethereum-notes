# Following [this tutorial](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737/)

```
pragma solidity ^0.4.24;

contract BasicToken {
  
  uint256 totalSupply_;
  mapping(address => uint256) balances;
  
  constructor(uint256 _initialSupply) public {
    totalSupply_ = _initialSupply;
    balances[msg.sender] = _initialSupply;
  }

  function totalSupply() public view returns (uint256) {
    return totalSupply_;
  }

  function transfer(address _to, uint256 _value) public returns (bool) {
    require(_to != address(0));
    require(_value <= balances[msg.sender]);
    balances[msg.sender]  -= _value;
    balances[_to] = balances[_to] + _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint256) {
    return balances[_owner];
  }
}
``` 
results in the opcodes under **opcodes.txt**

# Opcodes
* 0 gas
  * RETURN - return from contract call - Pop offset. Pop length. return memory[offset:offset+length]	
  * STOP - halts execution of contract
  * REVERT - Pop offset. Pop length. Reverts with return data = memory[offset:offset+length]
* 1 gas
	* JUMPDEST - Metadata that annotates a possible jump position
* 2 gas
  * CALLVALUE - Push msg.value
  * POP - Pops a (u)int256 and discards it
  * CALLER - Push msg.sender
  * CALLDATASIZE - Push len(msg.data)
* 3 gas
  * PUSH1 a - PUSH 1-byte value
  * PUSH2 a - PUSH 2-byte value
  * MSTORE - Pop offset. Pop value. memory[offset:offset+32] = value	
  * DUP1 - Push top (clone top of stack)
  * DUP4 - Push top, the fourth last value (clone fourth last value to top)
  * ISZERO - Pop a. Push (a == 0 ? 1 : 0)
  * MLOAD - Reads a (u)int256 from memory. Pop offset. Push value where value = memory[offset:offset+32]
  * ADD - Pop b. Pop a. Push a + b.
  * SWAP1 - Swaps top most two values. Pop b. Pop a. Push b. Push a. 
  * SWAP2 - Swaps top most value, with the second value in the stack, excepting the first one. (a, b, c) + SWAP2 = (c, b, a)
  * LT - Pop b. Pop a. Push (a < b ? 1 : 0)
  * CALLDATALOAD - Pop idx. Push msg.data[idx:idx+32]
* 10 gas
  * JUMPI - Pop destination. Pop condition. Jump to destination if condition is true.
* [A3 gas](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore)
  *  CODECOPY - Pop memoryStart index. Pop codeStart index. Pop length. memory[memoryStart:memoryStart+length] = address(this).code[codeStart:codeStart+length]
* [A7 gas](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore)
  * SSTORE - writed a (u)int256 value to storage. Pop key. Pop value. storage[key] = value;
* Unknown
  * SHA3 - Pop offset. Pop length. Push keccak256(memory[offset:offset+length]).

# Memory
First two words (0x00 and 0x20) are scratch space for hashing functions
Third word (0x40) is free memory pointer
Fourth word (0x60) should be zero permanently and be used as initial value for empty dynamic memory arrays

Storing mappings:
Assume the storage location of the mapping or array ends up being a slot **p** after applying the [storage layout rules](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding).
The value corresponding to a mapping key k is located at keccak256(h(k) . **p**) where . is concatenation and h is a function that is applied to the key depending on its type:

# Analysis of opcodes for deployment

PUSHn is the only EVM instruction composed of two or more bytes. That is why we miss instruction 001, because 001 is actually 0x80.
## Free memory pointer
---
000 PUSH1 80 // Push 0x80 (0x80)

002 PUSH1 40 // Push 0x40 (0x80, 0x40)

004 MSTORE // Store in memory, at position 0x40, 0x80 => Free memory pointer: Memory is free from position 0x80 onwards  ()

## Check that indeed it was non-payable (msg.value should be 0)
---
005 CALLVALUE // Push msg.value (msg.value)

006 DUP1 // Clone top (msg.value, msg.value)

007 ISZERO // (msg.value, msg.value == 0)

008 PUSH2 0010 ( msg.value, msg.value == 0, 0x0010)

011 JUMPI // if msg.value == 0 was false, doesn't jump (msg.value). Otherwise, jumps at 0x0010, which is 16.

012 PUSH1 00 // (msg.value, 0x00)

014 DUP1 // (msg.value, 0x00, 0x00)

015 REVERT // Reverts - because msg.value was not 0

## Copy contract code and retrieve constructor parameters
---
016 JUMPDEST // Annotation for possible jump location (msg.value)

017 POP  // Pop stack and discard ()

018 PUSH1 40 (0x40)

020 MLOAD // Pop offset. Push value, where value = memory[offset:offset+32]. Puts the free memory pointer on top, which was 0x80. (0x80)

021 PUSH1 20 // (0x80, 0x20)

023 DUP1 // (0x80, 0x20, 0x20)

024 PUSH2 0217 // (0x80, 0x20, 0x20, 0x0217)

027 DUP4 // (0x80, 0x20, 0x20, 0x0217, 0x80)

028 CODECOPY // Copies bytecode from executing contract to memory. memory[0x80:0x80+0x20] = bytecode[0x0217:0x0217+0x20] (0x80, 0x20). 0x0217 in hex = 535 in dec. Position 535 in bytecode is 

029 DUP2 // (0x80, 0x20, 0x80)

030 ADD // Pop 0x80. Pop 0x20. Push 0x80 + 0x20. (0x80, 0xA0)

031 PUSH1 40 // (0x80, 0xA0, 0x40)

033 SWAP1 // (0x80, 0x40, 0xA0)

034 DUP2 // (0x80, 0x40, 0xA0, 0x40)

035 MSTORE // Store in memory, at position 0x40, 0xA0. => The new free memory pointer, after copying bytecode from executing contract. (0x80, 0x40)

036 SWAP1 // (0x40, 0x80)

037 MLOAD // Push to stack value from position 0x80 from memory. (0x40, 0x2710)

## Constructor body
### totalSupply_ = 10000
---
038 PUSH1 00 // (0x40, 0x2710, 0x00)

040 DUP2 // (0x40, 0x2710, 0x00, 0x2710)

041 DUP2 // (0x40, 0x2710, 0x00, 0x2710, 0x00)

042 SSTORE // Pop key. Pop value. storage[key] = value. This initializes the totalSupply_ to 10000. (0x40, 0x2710, 0x00)\

043 CALLER // Push msg.sender. (0x40, 0x2710, 0x00, msg.sender)

044 DUP2 // (0x40, 0x2710, 0x00, msg.sender, 0x00)


045 MSTORE // memory[0x00] = msg.sender (0x40, 0x2710)

### concatenate mapping position with key
---
Compute where store ```balances[msg.sender] = _initialSupply;```, where in actual storage is msg.sender key in balances mapping. balances should start on the second storage slot (**so p = 1**) => we need to compute hash(h(msg.sender) . 0x01).

046 PUSH1 01 // (0x40, 0x2710, 0x00, 0x01) 

048 PUSH1 20 // (0x40, 0x2710, 0x00, 0x01, 0x20)

### balances[msg.sender] = totalSupply
---
050 MSTORE // memory[0x20] = 0x01 (0x40, 0x2710, 0x00)

051 SWAP2 // (0x00, 0x2710, 0x40)

052 SWAP1 // (0x00, 0x40, 0x2710)

053 SWAP2 // (0x2710, 0x40, 0x00)

054 SHA3 // Push keccak256(memory[0x00:0x00+0x40]). (0x2710, 0x36306db541fd1551fd93a60031e8a8c89d69ddef41d6249f5fdc265dbc8fffa2)

```balances[msg.sender] is 0x36306db541fd1551fd93a60031e8a8c89d69ddef41d6249f5fdc265dbc8fffa2```

```_initialSupply is 0x2710```

055 SSTORE // storage[0x36306db541fd1551fd93a60031e8a8c89d69ddef41d6249f5fdc265dbc8fffa2] = 0x2710 ()

## Copy runtime code to memory
---
056 PUSH2 01d1 // (0x01D1)
059 DUP1 // (0x01D1, 0x01D1)
060 PUSH2 0046 // ((0x01D1, 0x01D1, 0X0046)
063 PUSH1 00 // ((0x01D1, 0x01D1, 0X0046, 0X00)
065 CODECOPY // memory[0x00:0x00+0x01D1] = bytecode[0x0046:0x0046+0x01D1] (0x01D1)

The above copied code from position 70 to position 535 (0 - 70 is contract initialization code; 535-566 is constructor parameters).

## Return runtime code from transaction (actual code of the contract)
066 PUSH1 00 // (0x01D1, 0x00)
068 RETURN // Returns memory[0x00:0x00+0x01D1].
069 STOP

**The creation code gets executed in a transaction, which returns a copy of the runtime code, which is the actual code of the contract.**


# Analysis of opcodes for calling a method (totalBalance())

Calldata is an encoded chunk of hexadecimal numbers that contains information about what function of the contract we want to call, and it’s arguments or data. It consists of a “function id”, which is generated by hashing the function’s signature (truncated to the first leading four bytes) followed by the packed arguments data.

Calldata for totalBalance() = 0x18160DDD = keccak256("totalSupply()"), truncated to the first leading four bytes

## Free memory pointer
---
000 PUSH1 80 // (0x80)

002 PUSH1 40 // (0x40)

004 MSTORE // Make free memory pointer be 0x80. memory[0x40] = 0x80 ()

## Check calldata to have >= 4 bytes, since we don't have receive or fallback functions
005 PUSH1 04 // (0x04)

007 CALLDATASIZE // (0x04, 0x04)

008 LT // (0x00). Assuming our calldatasize had less than 4 bytes, LT would have been (0x01), and the jump would have happened, to 0x0056, which is 86 in dec. Stack would have become () => (0x00) => (0x00, 0x00) => REVERT because calldatasize < 4 would mean a fallback/receive function, but we don't have any, so we REVERT

009 PUSH2 0056 // (0x00, 0x0056)

012 JUMPI // Doesn't jump. ()

## Extract first four bytes of calldata
---
013 PUSH4 ffffffff // (0xFFFFFFFF)

018 PUSH29 0100000000000000000000000000000000000000000000000000000000 // (0xFFFFFFFF, 0x0100000000000000000000000000000000000000000000000000000000)

048 PUSH1 00 // (0xFFFFFFFF, 0x0100000000000000000000000000000000000000000000000000000000, 0x00)

050 CALLDATALOAD // ((0xFFFFFFFF, 0x000000000100000000000000000000000000000000000000000000000000000000, msg.data[0x00:0x20] == 0x18160DDD)
051 DIV // Dividing 0x18160DDD000..0000 by 0x000000000100000000000000000000000000000000000000000000000000000000 would give us the first four bytes. (0xFFFFFFFF,0x18160DDD)
052 AND // (0x18160DDD)

## Try to match "totalSupply()"
---
053 PUSH4 18160ddd // (0x18160DDD, 0x18160DDD)
058 DUP2 // (0x18160DDD, 0x18160DDD, 0x18160DDD)
059 EQ - LINE 3 // (0x18160DDD, 1)
060 PUSH2 005b // (0x18160DDD, 1, 0x005B)
063 JUMPI // Jumps at 0x005B which is 91. (0x18160DDD)

## Try to match "balanceOf(address)"
---
064 DUP1 -
065 PUSH4 70a08231 -
070 EQ -
071 PUSH2 0082 -
074 JUMPI -

## Try to match "transfer(address, uint256)"
---
075 DUP1 -
076 PUSH4 a9059cbb -
081 EQ -
082 PUSH2 00b0 -
085 JUMPI -
086 JUMPDEST -
087 PUSH1 00 -

## No match (and no fallback function), revert
--
089 DUP1 -
090 REVERT -


091 JUMPDEST -
092 CALLVALUE -
093 DUP1 -
094 ISZERO -
095 PUSH2 0067 -
098 JUMPI -
099 PUSH1 00 -
101 DUP1 -
102 REVERT -
103 JUMPDEST -
104 POP -
105 PUSH2 0070 -
108 PUSH2 00f5 -
111 JUMP -
112 JUMPDEST -
113 PUSH1 40 -
115 DUP1 -
116 MLOAD -
117 SWAP2 -
118 DUP3 -
119 MSTORE -
120 MLOAD -
121 SWAP1 -
122 DUP2 -
123 SWAP1 -
124 SUB -
125 PUSH1 20 -
127 ADD -
128 SWAP1 -
129 RETURN -
130 JUMPDEST -
131 CALLVALUE -
132 DUP1 -
133 ISZERO -
134 PUSH2 008e -
137 JUMPI -
138 PUSH1 00 -
140 DUP1 -
141 REVERT -
142 JUMPDEST -
143 POP -
144 PUSH2 0070 -
147 PUSH20 ffffffffffffffffffffffffffffffffffffffff -
168 PUSH1 04 -
170 CALLDATALOAD -
171 AND -
172 PUSH2 00fb -
175 JUMP -
176 JUMPDEST -
177 CALLVALUE -
178 DUP1 -
179 ISZERO -
180 PUSH2 00bc -
183 JUMPI -
184 PUSH1 00 -
186 DUP1 -
187 REVERT -
188 JUMPDEST -
189 POP -
190 PUSH2 00e1 -
193 PUSH20 ffffffffffffffffffffffffffffffffffffffff -
214 PUSH1 04 -
216 CALLDATALOAD -
217 AND -
218 PUSH1 24 -
220 CALLDATALOAD -
221 PUSH2 0123 -
224 JUMP -
225 JUMPDEST -