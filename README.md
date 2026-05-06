# Chess System

A console-based Chess game implemented in Java, designed as an educational
project to practice Object-Oriented Programming, Layered Architecture, and
core Chess rules including special moves.

The game runs in the terminal with ANSI-colored output, accepts moves in
algebraic notation (e.g. `e2`, `e4`) and handles a full game until checkmate.

---

## Table of Contents

1. [Features](#features)
2. [Tech Stack](#tech-stack)
3. [How to Run](#how-to-run)
4. [How to Play](#how-to-play)
5. [Project Structure](#project-structure)
6. [Architecture Overview](#architecture-overview)
7. [Domain Model](#domain-model)
8. [Game Rules Implemented](#game-rules-implemented)
9. [Suggested Improvements](#suggested-improvements)
10. [Roadmap](#roadmap)
11. [Acknowledgments](#acknowledgments)

---

## Features

- Standard 8x8 chess board with all six pieces.
- Full move legality validation (own-color, blocked path, target reachable).
- **Check** and **Checkmate** detection.
- **Special moves**:
  - Castling (kingside and queenside).
  - En passant.
  - Pawn promotion (Bishop, Knight, Rook, Queen).
- Move highlighting (shows valid destinations for the selected piece).
- Captured pieces tracking, displayed by color.
- Turn counter and current-player indicator.
- ANSI-colored terminal rendering.

## Tech Stack

- **Language:** Java (compatible with JavaSE-25 in `.classpath`; runs on Java 11+).
- **Build:** Eclipse project (`.project` / `.classpath`).
- **Dependencies:** None (pure standard library).

## How to Run

### From Eclipse

1. Import the project: `File > Import > Existing Projects into Workspace`.
2. Right-click [src/application/Program.java](src/application/Program.java) and select `Run As > Java Application`.

### From the command line

```bash
# Compile
javac -d bin src/module-info.java src/boardgame/*.java src/chess/*.java src/chess/pieces/*.java src/application/*.java

# Run
java -cp bin application.Program
```

> **Tip:** For a proper terminal experience, run in a console that supports
> ANSI escape codes (Windows Terminal, iTerm2, GNOME Terminal). The Eclipse
> built-in console will print escape codes literally.

## How to Play

The board is displayed each turn. Enter coordinates in algebraic notation:

```
   a b c d e f g h
8  R N B Q K B N R
7  P P P P P P P P
6  - - - - - - - -
5  - - - - - - - -
4  - - - - - - - -
3  - - - - - - - -
2  P P P P P P P P
1  R N B Q K B N R

Source: e2
Target: e4
```

When prompted for **promotion**, enter one of: `B` (Bishop), `N` (Knight),
`R` (Rook), `Q` (Queen).

Piece glyphs:

| Letter | Piece  |
|--------|--------|
| K      | King   |
| Q      | Queen  |
| R      | Rook   |
| B      | Bishop |
| N      | Knight |
| P      | Pawn   |

## Project Structure

```
src/
├── application/
│   ├── Program.java          # Entry point — game loop
│   └── UI.java               # Console rendering & input parsing
├── boardgame/
│   ├── Board.java            # Generic NxM board
│   ├── Piece.java            # Abstract piece (board-agnostic)
│   ├── Position.java         # (row, column) coordinates
│   └── BoardException.java   # Board-layer error
├── chess/
│   ├── ChessMatch.java       # Game state, moves, check/mate, special moves
│   ├── ChessPiece.java       # Chess-aware piece (color, move count)
│   ├── ChessPosition.java    # Algebraic position (a1..h8)
│   ├── Color.java            # WHITE / BLACK
│   ├── ChessException.java   # Chess-layer error
│   └── pieces/
│       ├── King.java
│       ├── Queen.java
│       ├── Rook.java
│       ├── Bishop.java
│       ├── Knight.java
│       └── Pawn.java
└── module-info.java
```

## Architecture Overview

The project follows a **layered architecture** that separates a generic
board-game framework from the chess-specific rules:

| Layer       | Package                | Responsibility                                  |
|-------------|------------------------|-------------------------------------------------|
| UI          | `application`          | Console I/O, rendering, input parsing           |
| Domain      | `chess`, `chess.pieces`| Chess rules, match flow, piece movement         |
| Foundation  | `boardgame`            | Generic board, piece and position abstractions  |

Dependency direction: `application → chess → boardgame`. The `boardgame`
package has no knowledge of chess and could in principle support other board
games (checkers, reversi, etc.).

## Domain Model

```
                  ┌──────────────┐
                  │   Program    │  (entry point)
                  └──────┬───────┘
                         │ uses
                  ┌──────▼───────┐
                  │      UI      │  (rendering / input)
                  └──────┬───────┘
                         │ orchestrates
                  ┌──────▼────────────────────┐
                  │       ChessMatch          │
                  │  - turn, currentPlayer    │
                  │  - check / checkMate      │
                  │  - performChessMove(...)  │
                  └──────┬────────────────────┘
                         │ owns
                  ┌──────▼───────┐        ┌─────────────┐
                  │    Board     │◄───────┤  Position   │
                  └──────┬───────┘        └─────────────┘
                         │ contains
                  ┌──────▼───────┐
                  │ ChessPiece   │ (abstract — extends Piece)
                  └──────┬───────┘
                         │
       ┌───────┬─────────┼─────────┬─────────┬──────────┐
     King    Queen      Rook    Bishop    Knight       Pawn
```

## Game Rules Implemented

| Rule                              | Status |
|-----------------------------------|--------|
| Standard piece movement           | ✅     |
| Capture                           | ✅     |
| Turn alternation                  | ✅     |
| Self-check prevention             | ✅     |
| Check detection                   | ✅     |
| Checkmate detection               | ✅     |
| Castling (kingside / queenside)   | ✅     |
| En passant                        | ✅     |
| Pawn promotion                    | ✅     |
| Stalemate / draw by repetition    | ❌     |
| Fifty-move rule                   | ❌     |
| Move history / PGN export         | ❌     |

## Suggested Improvements

> The following list is a code-quality review based on common OOP/Clean Code
> guidelines. The current implementation is solid as a learning project; the
> items below are polishing steps for production-quality code.

### Architecture & SOLID

- **Split `ChessMatch`** into smaller collaborators (currently ~350 lines mixing
  state, move execution, check evaluation, special moves and initial setup):
  - `MoveExecutor` — `makeMove` / `undoMove`.
  - `CheckEvaluator` — `testCheck` / `testCheckMate`.
  - `BoardInitializer` — `initialSetup`.
  - `PromotionPolicy` — promotion replacement logic.
- **Replace `instanceof` chains** in `makeMove` / `undoMove` with polymorphic
  hooks on `ChessPiece` (e.g. `onMove`, `onUndoMove`) — this respects OCP and
  keeps each piece's special behavior inside its own class.
- **Decouple `Pawn` and `King` from `ChessMatch`** — they only need a small
  read interface (e.g. `MatchContext` exposing `getEnPassantVulnerable()` and
  `isInCheck()`), avoiding the circular reference.

### DRY

- Extract a `SlidingPiece` base class (or movement strategy) for `Rook`,
  `Bishop`, and `Queen` — the directional sliding loop is duplicated.
- Hoist the duplicated `canMove(Position)` helper from `King` and `Knight` to
  `ChessPiece` (or use the existing `isThereOpponentPiece` consistently).
- Replace the `(color == WHITE) ? BLACK : WHITE` pattern (used in `nextTurn`
  and `opponent`) with `Color#opponent()` on the enum.
- Loop over columns `'a'..'h'` in `initialSetup` to remove 16 repetitive pawn
  placements.

### Code Quality

- **Boolean getters** should use `is`/`has` prefix: `isInCheck()`,
  `isCheckmate()` instead of `getCheck()` / `getCheckMate()`.
- Replace `check = (testCheck(...)) ? true : false;` with the boolean directly.
- In `ChessMatch.performChessMove`, the promotion condition has confusing
  operator precedence — wrap each clause in parentheses:
  ```java
  if ((color == WHITE && target.getRow() == 0) ||
      (color == BLACK && target.getRow() == 7)) { ... }
  ```
- Replace promotion magic strings (`"B"`, `"N"`, `"R"`, `"Q"`) with an enum
  `PromotionPiece` and use a `Map<PromotionPiece, PieceFactory>` instead of an
  `if`-chain in `newPiece`.
- Rename cryptic locals: `mat` → `moves`, `aux` → `removed`, `posT1/posT2` →
  `kingsideRookPos` / `queensideRookPos`.
- **Bug:** `Piece#isThereAnyPossibleMove` uses `mat.length` for the inner loop
  bound — this only works for square boards. Use `mat[i].length`.
- The `module-info.java` has two empty `/** */` Javadoc blocks before the
  `module` declaration — clean up.

### Immutability

- Make `Position` immutable (a `record` in modern Java). The current API
  exposes `setRow` / `setColumn` / `setValues`, encouraging in-place mutation
  inside piece movement code, which makes the logic harder to follow.
- Mark `Color`, `ChessPosition` already-immutable fields as `final`.

### Error Handling

- In `replacePromotedPiece`, an invalid `type` silently returns the current
  piece. Either reject with a clear `ChessException` or move the validation
  upstream (the UI already validates — make this method demand a valid input
  instead of accepting any string).
- Avoid catching `RuntimeException` broadly in `UI.readChessPosition`; catch
  `NumberFormatException` / `IndexOutOfBoundsException` specifically.

### Testing

- **No tests exist.** Add unit tests for:
  - Piece movement (`possibleMoves` for each piece, including blocked paths
    and captures).
  - `ChessMatch.performChessMove` happy path and rejections (own-color piece,
    blocked target, self-check).
  - Special moves: castling preconditions (rook unmoved, no check, path
    clear), en passant window, promotion.
  - Check / checkmate scenarios (e.g. fool's mate, scholar's mate, smothered
    mate).
- Aim for ≥75% coverage with adversarial cases, not just happy paths.

### Documentation & Tooling

- Add Javadoc for `ChessMatch` public API and each piece's `possibleMoves`.
- Provide a Maven or Gradle build (`pom.xml` / `build.gradle`) for portable
  builds outside Eclipse.
- Add CI (GitHub Actions) running tests on push.

## Roadmap

- [ ] Stalemate / draw detection.
- [ ] Move history & PGN export.
- [ ] Two-player network mode.
- [ ] Simple AI opponent (minimax).
- [ ] GUI (JavaFX or Swing).

## Acknowledgments

This project follows the structure of the well-known Java OOP course exercise
on Chess, and was extended with castling, en passant and promotion special
moves while keeping a clean separation between the generic `boardgame`
framework and the chess-specific domain.
