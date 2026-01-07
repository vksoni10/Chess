# ♟️ Java Swing Chess Client (Local, Online & AI)

This project is a **full-featured Java chess application** built with **Swing** and **Chesslib**, supporting local two-player games, online multiplayer over sockets, and single-player mode against the **Stockfish** chess engine.  
The application provides a rich GUI experience with **strict move validation**, **legal move highlighting**, and **complete game-state detection** (check, checkmate, stalemate, and all standard draw conditions).

---

## ✨ Features

### 🎮 Game Modes

- **Local Two Player (Hotseat)**  
  Two players alternate moves on the same machine with turn-based enforcement.

- **Online Two Player**  
  Play against another player over the network using a simple TCP client–server protocol.

- **Single Player vs AI**  
  Play against the Stockfish chess engine with configurable thinking time.

---

### ♜ Rules & Validation (Chesslib)

The game uses **Chesslib** for all core chess logic:

- Legal move generation  
  `MoveGenerator.generateLegalMoves(board)`
- Board state & FEN handling  
  `Board`, `getFen()`
- Game state checks  
  - Check detection: `isKingAttacked()`
  - Checkmate: `isMated()`
  - Stalemate: `isStaleMate()`
  - Draw detection:
    - `isDraw()`
    - `isRepetition()`
    - `isInsufficientMaterial()`
    - 50-move rule via `getHalfMoveCounter()`

**Every move is validated by:**
1. Creating a `Move` object from source → destination (plus promotion piece if needed).
2. Generating all legal moves for the current position.
3. Applying the move **only if it exists** in the legal move list.

Supported rules:
- Normal movement and captures
- Castling (king-side and queen-side)
- En passant
- Pawn promotion (user-selectable piece)
- Check, checkmate, stalemate, and all standard draw conditions

---

## 🖥️ GUI & UX (Swing)

### Board
- 8×8 chessboard built using a grid of `JButton`s.
- Optional board flipping for local two-player mode.

### Piece Display
- PNG icons loaded from the `images/` directory:
  - `wK.png`, `wQ.png`, `wR.png`, `wB.png`, `wN.png`, `wP.png`
  - `bK.png`, `bQ.png`, `bR.png`, `bB.png`, `bN.png`, `bP.png`
- Falls back to Unicode chess symbols if images are missing.

### Input & Feedback
- **Click-to-move** input system.
- Selected piece highlighting.
- Legal destination square highlighting.
- Status labels showing:
  - Current mode (Local / Online / AI)
  - Side to move
  - Move number
  - Half-move clock
  - Check / checkmate / draw messages

---

## 🤖 AI Integration (Stockfish)

- Uses an **external Stockfish executable** (UCI engine).
- Sends the current position (FEN) to Stockfish.
- Requests a best move with configurable thinking time:
  - 500 ms
  - 1000 ms
  - 2000 ms
  - 3000 ms (or more)

**AI flow:**
1. Player makes a legal move.
2. UI updates immediately.
3. Engine move is calculated asynchronously.
4. Engine move is validated and applied using Chesslib.

> By default, the human player is **White** in AI mode.

---

## 🌐 Online Multiplayer

### Protocol (TCP, line-based)

- `COLOR:white` / `COLOR:black` – assign side
- `START` – game begins
- `YOUR_TURN` – client may move
- `MOVE:e2e4` – move in UCI notation
- `RESET` – reset the game

### Client Behavior
- Connects to server using `Socket`.
- Reads messages via `BufferedReader`.
- Dispatches messages in `handleServerMessage(...)`.

When a player moves:
1. Move is validated locally.
2. Move is applied to the local board.
3. `MOVE:<uci>` is sent to the server.
4. Client waits for `YOUR_TURN` before allowing the next move.

---

## 🧠 Core Logic Overview

### Board Representation
Uses `com.github.bhlangonijr.chesslib.Board` to store:
- Piece placement
- Side to move
- Castling rights
- En passant square
- Half-move counter
- Full-move number

### GUI Mapping
- `JButton squares[8][8]` represents the board.
- Helper methods convert between `(row, col)` and algebraic notation (`"A1"` … `"H8"`).
- Supports optional board flipping.

---

### Move Selection & Validation

#### First Click – Select Piece
- Validates:
  - A piece exists on the square.
  - The piece belongs to the side to move.
- Highlights:
  - Selected square.
  - All legal destination squares.

#### Second Click – Destination
- Clicking the same square cancels selection.
- Clicking another square:
  - Builds a move string (e.g. `E2E4`).
  - Opens a promotion dialog if required.
  - Creates a `Move` instance.
  - Compares against all legal moves.

If legal:
- `board.doMove(move)` updates the board.
- GUI refreshes.
- Turn switches.

If illegal:
- Error dialog shown.
- Selection and highlights cleared.

---

## 🏁 Game State & End Conditions

After every move, the game checks for:
- Check
- Checkmate
- Stalemate
- Draw (repetition, insufficient material, 50-move rule)

When a terminal state is reached:
- The game is marked as ended.
- A dialog shows the result.
- Further moves are blocked until reset.

---

## 📁 Project Structure

```text
src/
├── client/
│   └── ChessClient.java        # Main GUI & controller
├── engine/
│   └── StockfishEngine.java    # Stockfish process wrapper
├── server/
│   └── ChessServer.java        # TCP server for online play
├── images/
│   ├── wK.png, wQ.png, ...
│   └── bK.png, bQ.png, ...
├── lib/
│   └── chesslib.jar
