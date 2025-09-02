# ₿CHess
**[Note]** *This is an early version that operated on a single board. The version demoed at BLISS '25 was updated in various ways, including Factory contracts to create more boards, locking down the game to two specific addresses, BCH betting, etc. Some other logic/functions were also changed, but this early version shows the general logic behind the design. Will release the updated version later.
These contracts were also designed to fit into the pre-2025 VM limits.* 

Chess game state stored with moves validated by contracts on Bitcoin Cash. Allows resetting the game after a king has actually been captured, rather than on checkmate.

## ChessMaster
The ChessMaster contract tracks the current turn number and whether a king has been captured. ChessMaster is included in every transaction (turn), and if a king has been captured then it doesn't allow further moves to occur.

## Squares
Each square of the chessboard is a utxo on the Square contract.

Each square keeps a copy of:
- its new game starting state (color/piece, permanent)
- its x/y coordinate (permanent)
- its current color/piece occuping the square (can change)
  
move() removes (color/piece) from the source square and puts it on the destination square, checking that it isn't occupied by its own team already, and overwrites any enemy piece on it.

checkEmpty() checks the square to verify it has no pieces currently on it. This is used to verify a piece that moves multiple squares (Queen/Rook/Bishop) aren't trying to pass through an occupied tile. 

Squares need to be added as sequential inputs as they verify each other to enforce straight/diagonal paths. This prevents creating an invalid (chess rule) zig-zagging path by selecting incorrect squares but still ending up at the correct destination square. 

E.g. Bishop moving 4 tiles
- input3: SourceSquare move()
- input4: IntermediateSquare checkMove()
- input5: IntermediateSquare checkMove()
- input6: DestinationSquare move()

## PieceLogic
Each piece type (pawn/knight/bishop/rook/queen/king) has its own contract with logic that enforces the allowed x/y movement rules for that piece. When a piece moves its logic contract must be included.

## Game Reset 
Resetting the game can only be performed after a King has been captured. 

## Stalemates & Checks
Checks are not accounted for, so a player can move their King into a threatened space which will cause them to lose if it gets captured. 

Stalemates are also not accounted for, and can be resolved simply by moving the King into a threatened space to be captured. Alternatively, a draw() function could be added for when both players agree to a draw.

## Other
Preview state, things are missing/not locked down but basic functionality is working so far. Does not currently have castling rule, but Piece contracts themselves are fairly small. Squares contract is maxed out in current design.

Could easily add betting/payouts by locking down the game to two addresses (anyone can call each turn currently) and adding a fund() to the ChessMaster, letting it hold the BCH bet then paying it out to the winning address (could save the dead king color, or have the payout verify the still-alive king).
