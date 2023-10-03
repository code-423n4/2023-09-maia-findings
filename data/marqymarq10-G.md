https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L447-L453
BranchBridgeAgent.sol : line: 447 
Switch for loop to while loop
Example:
``		
        uint256 i = 0;
        uint256 len =  deposit.tokens.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }

``

