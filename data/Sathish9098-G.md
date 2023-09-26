FALSE	Remove or replace unused state variables	

Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it's assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot	


45:       bytes32 private _PERMIT_TYPEHASH_DEPRECATED_SLOT;																							
			112:      string private baseURI;	


FALSE	State variables can be packed into fewer storage slots	

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper		

																						
			/// @audit Variable ordering with 7 slots instead of the current 8:																							
			///           mapping(32):operators, uint256(32):totalFees, uint256(32):maxDebtRate, uint256(32):minDebtRate, uint256(32):debtRateAgainstEthMarket, uint256(32):debtStartPoint, user-defined(20):accrueInfo, bool(1):_isEthMarket																							
			46:       mapping(address => mapping(address => bool)) public operators;


Using storage instead of memory for structs/arrays saves gas	

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct	

																							
			File: tapioca-bar-audit/contracts/markets/bigBang/BigBang.sol																							
			513:          IBigBang.AccrueInfo memory _accrueInfo = accrueInfo;																							
			525:          Rebase memory _totalBorrow = totalBorrow;	


FALSE	<x> += <y> costs more gas than <x> = <x> + <y> for state variables	

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)																								
																										
			446:          totalFees += balance;																							
			204:          dso_supply -= mintedInWeek[week - 1];	


FALSE	Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement	

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }																								
																										
			/// @audit require() on line 142																							
			146:                  balanceOf[msg.sender] = srcBalance - amount; // Underflow is checked																							
			/// @audit require() on line 167																							
			181:                  balanceOf[from] = srcBalance - amount; // Underflow is checked																							
			/// @audit require() on line 174																							
			177:                      allowance[from][msg.sender] = spenderAllowance - amount; // Underflow is checked		

FALSE	private functions not called by the contract should be removed to save deployment gas																									
																										
			631       function _executeViewModule(																							
			632           Module _module,																							
			633           bytes memory _data																							
			634:      ) private view returns (bytes memory returnData) {	


FALSE	internal functions not called by the contract should be removed to save deployment gas	

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword																								
			254       function _getAmountForBorrowPart(																							
			255           uint256 borrowPart																							
			256:      ) internal view returns (uint256) {																							
																																																																																																																	


Don't compare boolean expressions to boolean literals	

if (<x> == true) => if (<x>), if (<x> == false) => if (!<x>)	
																							
			175:              isSingularityMasterContractRegistered[mc] == true,																							
			183:              isBigBangMasterContractRegistered[mc] == true,																							
			322:              isSingularityMasterContractRegistered[mcAddress] == false,																							
			344:              isBigBangMasterContractRegistered[mcAddress] == false,


FALSE	Multiple if-statements with mutually-exclusive conditions should be changed to if-else statements

	If two conditions are the same, their blocks should be combined																								
			264           if (borrowPartDecimals > 18) {																							
			265               borrowPartScaled = borrowPart / (10 ** (borrowPartDecimals - 18));																							
			266           }																							
			267           if (borrowPartDecimals < 18) {																							
			268               borrowPartScaled = borrowPart * (10 ** (18 - borrowPartDecimals));																							
			269:          }																							
			272           if (collateralPartDecimals > 18) {																							
			273               collateralPartInAssetScaled =																							
			274                   collateralPartInAsset /																							
			275                   (10 ** (collateralPartDecimals - 18));																							
			276           }																																																																																																																					
						

Empty blocks should be removed or emit something	

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if (x) {...} else if (y) {...} else {...} => if (!x) { if (y) {...} else {...} }). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.																								
			

267:                  {} catch Error(string memory reason) {																							
			282:                  {} catch Error(string memory reason) {																							
			298:                  {} catch Error(string memory reason) {																							
																										
Superfluous event fields	


block.timestamp and block.number are added to event information by default so adding them manually wastes gas																								
			138:      event ProtocolWithdrawal(IMarket[] markets, uint256 timestamp);

Division by two should use bit shifting	<x> / 2 is the same as <x> >> 1. 

While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of [20 gas](https://gist.github.com/IllIllI000/ec0e4e6c4f52a6bca158f137a3afd4ff) due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two																								
			

150:              uint256 x = y / 2 + 1;																							
			153:                  x = (y / x + x) / 2;																							
																										
Structs can be packed into fewer storage slots by truncating timestamp bytes	

By using a uint32 rather than a larger type for variables that track timestamps, one can save gas by using fewer storage slots per struct, at the expense of the protocol breaking after the year 2106 (when uint32 wraps). If this is an acceptable tradeoff, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings																								
																										
			/// @audit Variable ordering with 3 slots instead of the current 4:																							
			///           uint256(32):amount, uint256(32):claimed, uint32(4):latestClaimTimestamp, bool(1):revoked																							
			32        struct UserData {																							
			33            uint256 amount;																							
			34            uint256 claimed;																							
			35            uint256 latestClaimTimestamp;																							
			36            bool revoked;																							
			37:       }		


																					
Multiple accesses of a mapping/array should use a local variable cache	

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata																								
																										
			/// @audit activeSingularities[_singularity] on line 180																							
			187:          activeSingularities[_singularity].totalDeposited += _amount;																							
			/// @audit activeSingularities[_singularity] on line 221																							
			247:          activeSingularities[_singularity].totalDeposited -= lockPosition.amount;																							
			/// @audit activeSingularities[singularity] on line 264																							
			267:          activeSingularities[singularity].poolWeight = weight;																							
			/// @audit activeSingularities[singularity] on line 283																							
			287:          activeSingularities[singularity].sglAssetID = assetID;																							
			/// @audit activeSingularities[singularity] on line 287	


keccak256() should only need to be called on a specific string literal once	

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once																								
																										
			366                   keccak256(																							
			367                       "onERC1155Received(address,address,uint256,uint256,bytes)"																							
			368:                  )	


Stack variable used as a cheaper cache for a state variable is only used once	

If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend	
																							
			160:          uint256 cachedEpoch = epoch;																							
																										
Consider using bytes32 rather than a string	

Using the bytes types for fixed-length strings is more efficient than having the EVM have to incur the overhead of string processing. Consider whether the value needs to be a string. A good reason to keep it as a string would be if the variable is defined in an interface that this project does not own.																								
																										
			8:       string public _name;																							
			9:       string public _symbol;		

																					
																										
The result of function calls should be cached rather than re-calling the function	

The instances below point to the second+ call of the function within a single function																								
																										
																										
			/// @audit market.oracle() on line 918																							
			930:          (, info.oracleExchangeRate) = IOracle(market.oracle()).peek(																							
			/// @audit market.oracleData() on line 919																							
			931:              market.oracleData()																							
			/// @audit market.oracle() on line 918																							
			933:          info.spotExchangeRate = IOracle(market.oracle()).peekSpot(																							
			/// @audit market.oracleData() on line 919																							
			934:              market.oracleData()


State variables can be packed into fewer storage slots by truncating timestamp bytes	

By using a uint32 rather than a larger type for variables that track timestamps, one can save gas by using fewer storage slots per struct, at the expense of the protocol breaking after the year 2106 (when uint32 wraps). If this is an acceptable tradeoff, if variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper																								
																										
	/// @audit Variable ordering with 3 slots instead of the current 4:																									
	///           uint256(32):MAX_DELAY, mapping(32):schedule, uint32(4):delay, bool(1):paused																									
	17:       uint256 private MAX_DELAY = 4 weeks;	

Calculations should be memoized rather than re-calculating them																									
																										
	/// @audit BytesLib.toUint128() on line 256																									
	257:          price = BytesLib.toUint128(_msg, 41);																									
	/// @audit BytesLib.toUint128() on line 286																									
	289:          amount = BytesLib.toUint128(_msg, 81);																									
	/// @audit BytesLib.toUint128() on line 299																									
	301:          amount = BytesLib.toUint128(_msg, 81);																									
	/// @audit BytesLib.toUint128() on line 396																									
	397:          amount = BytesLib.toUint128(_msg, 73);	

Don't use _msgSender() if not supporting EIP-2771	

Use msg.sender if the code does not implement [EIP-2771 trusted forwarder](https://eips.ethereum.org/EIPS/eip-2771) support																								
																										
Using globals directly is cheaper than assigning them to variables	

Using the global directly saves 5 gas																								
																										
	118:         address liquidityPool = msg.sender;																									
	149:         address liquidityPool = msg.sender;																									
	428:         address liquidityPool = msg.sender;

State variables should be cached in stack variables rather than re-reading them from storage

State variables only set in the constructor should be declared immutable

Is there any possibility refactoring saves gas 	like external calls in the last expensive operations in the last 

Don't emit event 

Combine events 

use assembly for loops 

																																																																																															
																																																																																																																	