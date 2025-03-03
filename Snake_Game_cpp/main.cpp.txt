#include <iostream>
#include <queue>
#include <chrono>
#include <thread>
#ifdef _WIN32
#include <conio.h> // For _kbhit() and _getch() on Windows
#include <windows.h> // For cursor manipulation on Windows
#else
#include <unistd.h>
#include <termios.h>
#include <fcntl.h>
#endif

using namespace std;

const int width = 35;  // Width of the game board
const int height = 20; // Height of the game board

struct Point {
    int x, y;
};

// Cross-platform non-blocking input
bool kbhit() {
#ifdef _WIN32
    return _kbhit(); // Windows-specific
#else
    struct termios oldt, newt;
    int ch;
    int oldf;
    tcgetattr(STDIN_FILENO, &oldt);
    newt = oldt;
    newt.c_lflag &= ~(ICANON | ECHO);
    tcsetattr(STDIN_FILENO, TCSANOW, &newt);
    oldf = fcntl(STDIN_FILENO, F_GETFL, 0);
    fcntl(STDIN_FILENO, F_SETFL, oldf | O_NONBLOCK);
    ch = getchar();
    tcsetattr(STDIN_FILENO, TCSANOW, &oldt);
    fcntl(STDIN_FILENO, F_SETFL, oldf);
    if (ch != EOF) {
        ungetc(ch, stdin);
        return true;
    }
    return false;
#endif
}

// Cross-platform get character
char getch() {
#ifdef _WIN32
    return _getch(); // Windows-specific
#else
    struct termios oldt, newt;
    char ch;
    tcgetattr(STDIN_FILENO, &oldt);
    newt = oldt;
    newt.c_lflag &= ~(ICANON | ECHO);
    tcsetattr(STDIN_FILENO, TCSANOW, &newt);
    ch = getchar();
    tcsetattr(STDIN_FILENO, TCSANOW, &oldt);
    return ch;
#endif
}

// Hide the cursor
void hideCursor() {
#ifdef _WIN32
    HANDLE consoleHandle = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_CURSOR_INFO info;
    info.dwSize = 100;
    info.bVisible = FALSE;
    SetConsoleCursorInfo(consoleHandle, &info);
#else
    cout << "\033[?25l"; // ANSI escape code to hide cursor
#endif
}

// Show the cursor
void showCursor() {
#ifdef _WIN32
    HANDLE consoleHandle = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_CURSOR_INFO info;
    info.dwSize = 100;
    info.bVisible = TRUE;
    SetConsoleCursorInfo(consoleHandle, &info);
#else
    cout << "\033[?25h"; // ANSI escape code to show cursor
#endif
}

class Snake {
private:
    queue<Point> body; // Queue to store the snake's body
    Point food;       // Position of the food
    char direction;   // Current direction of the snake
    bool gameOver;    // Game over flag
    int speed;        // Speed of the snake (delay in milliseconds)

    // Generate food at a random position
    void generateFood() {
        food.x = rand() % width;
        food.y = rand() % height;
    }

    // Draw the game board
    void draw() {
        // Clear the screen (not using system("cls") for compatibility)
        cout << "\033[H\033[J"; // ANSI escape code to clear screen (works on most online compilers)

        // Draw the top border
        for (int i = 0; i < width + 2; i++)
            cout << "#";
        cout << endl;

        // Draw the game board
        for (int i = 0; i < height; i++) {
            for (int j = 0; j < width; j++) {
                if (j == 0)
                    cout << "#"; // Left border

                // Check if the current position is part of the snake's body
                bool isBodyPart = false;
                queue<Point> temp = body;
                while (!temp.empty()) {
                    Point p = temp.front();
                    temp.pop();
                    if (p.x == j && p.y == i) {
                        cout << "*"; // Draw the snake's body
                        isBodyPart = true;
                        break;
                    }
                }

                // If not part of the body, check if it's food
                if (!isBodyPart) {
                    if (i == food.y && j == food.x)
                        cout << "o"; // Draw the food
                    else
                        cout << " "; // Empty space
                }

                if (j == width - 1)
                    cout << "#"; // Right border
            }
            cout << endl;
        }

        // Draw the bottom border
        for (int i = 0; i < width + 2; i++)
            cout << "#";
        cout << endl;
    }

    // Handle user input
    void input() {
        if (kbhit()) { // Check if a key is pressed
            char key = getch(); // Get the pressed key
            switch (key) {
                case 'a':
                    if (direction != 'd')
                        direction = 'a'; // Move left
                    break;
                case 'd':
                    if (direction != 'a')
                        direction = 'd'; // Move right
                    break;
                case 'w':
                    if (direction != 's')
                        direction = 'w'; // Move up
                    break;
                case 's':
                    if (direction != 'w')
                        direction = 's'; // Move down
                    break;
                case 'x':
                    gameOver = true; // Exit the game
                    break;
            }
        }
    }

    // Update game logic
    void logic() {
        // Calculate the new head position based on the current direction
        Point newHead = body.back();
        switch (direction) {
            case 'a':
                newHead.x--;
                break;
            case 'd':
                newHead.x++;
                break;
            case 'w':
                newHead.y--;
                break;
            case 's':
                newHead.y++;
                break;
        }

        // Check for collision with walls
        if (newHead.x < 0 || newHead.x >= width || newHead.y < 0 || newHead.y >= height) {
            gameOver = true;
            return;
        }

        // Check for collision with the snake's body
        queue<Point> temp = body;
        while (!temp.empty()) {
            Point p = temp.front();
            temp.pop();
            if (p.x == newHead.x && p.y == newHead.y) {
                gameOver = true;
                return;
            }
        }

        // Add the new head to the body
        body.push(newHead);

        // Check if the snake eats the food
        if (newHead.x == food.x && newHead.y == food.y) {
            generateFood(); // Generate new food
        } else {
            body.pop(); // Remove the tail (snake doesn't grow)
        }
    }

public:
    Snake(int level) {
        direction = 'd'; // Initial direction: right
        gameOver = false;
        Point head = {width / 2, height / 2}; // Initial head position
        body.push(head);
        generateFood(); // Generate the first food

        // Set speed based on level
        switch (level) {
            case 1:
                speed = 200; // Easy: Slow speed
                break;
            case 2:
                speed = 100; // Medium: Medium speed
                break;
            case 3:
                speed = 50;  // Hard: Fast speed
                break;
            default:
                speed = 100; // Default to medium
                break;
        }
    }

    void run() {
        hideCursor(); // Hide the cursor before starting the game
        while (!gameOver) {
            draw();      // Draw the game board
            input();     // Handle user input
            logic();     // Update game logic
            this_thread::sleep_for(chrono::milliseconds(speed)); // Delay based on speed
        }
        showCursor(); // Show the cursor after the game ends
        cout << "Game Over!" << endl;
    }
};

int main() {
    int level;
    cout << "Select level:\n";
    cout << "1. Easy\n";
    cout << "2. Medium\n";
    cout << "3. Hard\n";
    cout << "Enter your choice (1-3): ";
    cin >> level;

    if (level < 1 || level > 3) {
        cout << "Invalid choice. Exiting..." << endl;
        return 0;
    }

    Snake snake(level);
    snake.run();
    return 0;
}