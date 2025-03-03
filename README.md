# Snake Game in C++

A classic **Snake Game** implemented in C++ using data structures like queues and cross-platform libraries for real-time input and cursor manipulation. The game features multiple difficulty levels and a simple, text-based interface.

---

## Features

* **Real-Time Movement**: The snake moves in real-time based on user input without requiring the Enter key.
* **Multiple Difficulty Levels**: Choose from Easy, Medium, or Hard to adjust the snake's speed.
* **Cross-Platform**: Works on Windows, Linux, and macOS.
* **Simple Interface**: Text-based graphics using `*` for the snake and `o` for the food.
* **Cursor Management**: The blinking cursor is hidden during gameplay for a better user experience.

---
## Game Controls

* **w**: Move Up
* **a**: Move Left
* **s**: Move Down
* **d**: Move Right
* **x**: Exit the Game

---
## Select Difficulty

Choose a level when prompted:

**1**: Easy (Slow Speed)

**2**: Medium (Medium Speed)

**3**: Hard (Fast Speed)

---

## Code Structure
**main.cpp**: Contains the main game logic, including input handling, rendering, and game mechanics.

### Data Structures:
queue<Point>: Used to store the snake's body.

Point: Represents a coordinate on the game board.

Cross-Platform Support

* **Uses conio.h and windows.h for Windows**

* **Uses termios.h and fcntl.h for Linux/macOS**

---
## Dependencies
C++ Compiler: Ensure you have a C++ compiler installed (e.g., g++ or clang++).

---
### Cross-Platform Libraries:

* **Windows: conio.h, windows.h**

* **Linux/macOS: unistd.h, termios.h, fcntl.h**

---
## Future Improvements
* **Add a scoring system to track the player's progress.**
* **Introduce obstacles or walls for increased difficulty.**
* **Improve the user interface with colors or better graphics.**
* **Add a high-score system to save and display top scores.**

---
## How to Use

1. **Clone the Repository**
   ```bash
   git clone https://github.com/your-username/snake-game-cpp.git
   cd snake-game-cpp
