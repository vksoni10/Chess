Java Swing Chess Client (Local, Online & AI)
This project is a full‑featured Java chess application built with Swing and Chesslib, supporting local two‑player games, online multiplayer over sockets, and single‑player mode against the Stockfish chess engine. The GUI includes piece selection, legal move highlighting, strict move validation using Chesslib’s legal move generator, and full game state detection (check, checkmate, stalemate, draw).

Features
Three game modes

Local Two Player (hotseat): Two players alternate on the same machine, with turn‑based enforcement.

Online Two Player: Play over the network using a simple TCP socket protocol (client–server).

Single‑Player vs AI: Play against the Stockfish engine with configurable thinking time.

Rules and validation

Uses Chesslib for:

Legal move generation (MoveGenerator.generateLegalMoves).

Board state and FEN handling (Board, getFen()).

Game state checks (isKingAttacked, isMated, isStaleMate, isDraw, isRepetition, isInsufficientMaterial).

Every user move is validated by:

Building a Move object from the selected source and destination (plus promotion piece when needed).

Generating all legal moves for the current position.

Applying the move only if it exists in the legal move list.

Supports all standard chess rules:

Normal piece movement and captures.

Castling (both sides).

En passant.

Pawn promotion (with user choice of promotion piece).

Check, checkmate, stalemate, and draw detection (50‑move rule, repetition, insufficient material).

GUI and UX (Swing)

8×8 chessboard using a JButton grid.

Piece display:

PNG icons loaded from an images/ directory (e.g. wK.png, bQ.png).

Falls back to Unicode chess glyphs if icons are missing.

Input modes:

Click‑to‑move:

First click selects a piece (only if it belongs to the side to move).

Second click chooses a destination square and triggers validation.

Selection and legal move highlighting:

Selected piece is highlighted (e.g. colored border).

All legal destination squares for the selected piece are highlighted.

Status and info labels:

Show current mode (local / online / AI).

Show move number, side to move, half‑move clock.

Indicate check, checkmate, stalemate, and draw conditions.

AI integration (Stockfish)

External Stockfish executable is used as the engine.

The client passes the current position (FEN) to Stockfish and requests a best move with a configurable thinking time (e.g. 500 ms, 1000 ms, 2000 ms, 3000 ms).

After the player’s legal move:

The engine move is queried asynchronously (so the UI remains responsive).

The returned move string is converted to a Move, validated against Chesslib’s legal moves, and then applied to the board.

Human is White by default in AI mode (easy to change in code).

Online multiplayer

Simple line‑based text protocol over TCP sockets:

COLOR:white / COLOR:black – assign side to each client.

START – signal that the game has begun.

YOUR_TURN – notify the client that it may move.

MOVE:e2e4 – communicate moves.

RESET – reset the game to the initial position.

Client:

Connects to a server using Socket.

Reads incoming messages via BufferedReader and dispatches them in handleServerMessage.

When the local player makes a legal move, the client:

Applies it locally.

Sends MOVE:<uci> to the server.

Waits for YOUR_TURN before accepting the next move.

How the Core Logic Works
Board representation
Uses com.github.bhlangonijr.chesslib.Board to store:

Piece placements.

Side to move.

Castling rights, en passant square.

Half‑move counter and full‑move number.

The GUI board:

A 2D array squares[8][8] of JButtons.

A helper method converts between (row, col) indices and algebraic notation ("A1" .. "H8"), taking into account optional board flipping in local two‑player mode.

Move selection and validation
First click – selecting a piece

If no square is currently selected (selectedSquare == null):

The client checks the underlying Board to see if:

There is a piece on the clicked square, and

The piece belongs to the correct side (based on isMyTurn, playerColor, and game mode).

If valid:

selectedSquare is set to that square.

The selected square is highlighted.

highlightLegalMoves(selectedSquare) is called:

Generates all legal moves for the current position.

Filters moves whose from square equals selectedSquare.

Highlights each destination square in the GUI.

Second click – choosing a destination

If selectedSquare is already set:

Clicking the same square again cancels the selection (clears highlights and resets selectedSquare).

Clicking a different square:

Builds a move string like "E2E4". If the move is a pawn move to the last rank, a promotion dialog opens and a promotion piece (q, r, b, n) is appended to the string.

Creates a Move instance from that string and the correct side.

Calls MoveGenerator.generateLegalMoves(game) to get all legal moves for the current position.

Compares the candidate move with the legal moves using .equals().

If it matches one of them:

game.doMove(move) is invoked to update the Chesslib board.

updateBoard() redraws the GUI board based on the new state.

updateGameInfo() refreshes the labels with the current move number, side to move, etc.

checkGameState() checks for check, checkmate, stalemate, or draw and updates status messages.

Turn flags (isMyTurn) and, if desired, board orientation (isBoardFlipped) are updated.

If not legal:

The client shows “Illegal move! Try again.” and clears selection and highlights.

Game state and end conditions
After each applied move, checkGameState() uses Chesslib methods:

isKingAttacked() and isMated() to detect:

Check (king in check but not mated).

Checkmate (game ends; winner determined by opposite of getSideToMove()).

isStaleMate() to detect stalemate.

isDraw(), plus specific helpers such as isRepetition(), isInsufficientMaterial(), and getHalfMoveCounter() to determine draw reasons (e.g. repetition, insufficient material, 50‑move rule).

When a terminal state is reached:

gameEnded is set to true.

A dialog informs the user of the result (e.g. “Checkmate! White wins.” or “Draw by stalemate.”).

Further moves are ignored until the game is reset.

Project Structure (typical)
src/

client/ChessClient.java – main GUI and game controller.

engine/StockfishEngine.java – wrapper for launching and communicating with the Stockfish executable using a UCI‑like interface.

server/ChessServer.java – simple TCP server for coordinating online games (if present in your project).

images/ – PNG images for white/black pieces, named like wK.png, wQ.png, wR.png, wB.png, wN.png, wP.png and the corresponding black pieces.

lib/ – external libraries such as chesslib.jar.

Getting Started
Prerequisites
Java 8+ installed.

Chesslib JAR on the classpath.

Stockfish engine binary downloaded (e.g. from the official Stockfish repository or releases) and placed where StockfishEngine expects it (commonly the project root, named stockfish or stockfish.exe, configurable in code).

Running the client
Add Chesslib and any other required JARs to your classpath.

Compile:

bash
javac -cp .:lib/chesslib.jar src/client/ChessClient.java
Run:

bash
java -cp .:lib/chesslib.jar client.ChessClient
Choose a mode from the dropdown:

Local Two Player to play on the same machine.

Online Two Player to connect to a running server.

Single‑Player vs AI to play against Stockfish.

Running online mode
Start your ChessServer (if included) on a machine accessible to clients.

Launch ChessClient on each player’s machine and choose Online Two Player mode.

Ensure both clients connect to the correct host and port used by the server (for example localhost:8000 for local testing).

Customization
Change piece graphics: Replace images in the images/ folder with your own designs, keeping the same filenames.

Adjust colors: Modify the light and dark square colors and highlight colors in ChessClient.java.

Tune AI: Modify default aiThinkTime values or add additional presets for difficulty.

Extend protocol: Add more message types to the online protocol (e.g. chat, draw offers, resign).

Credits and License
Chess rules and FEN/PGN support are provided by Chesslib.

Engine analysis is powered by Stockfish, a free, strong UCI chess engine.

This project is a Java Swing client that integrates these tools into a complete chess playing experience.

(Insert your preferred open‑source license here, e.g. MIT, Apache‑2.0, or GPL‑3.0, making sure it is compatible with the licenses of Chesslib and Stockfish.)
