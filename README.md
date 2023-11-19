#include "raylib.h"
#include <stack>

// Define game states
enum class GameState
{
    Playing,    // The main gameplay state
    GameOver,   // The game is over, and players can restart
};

std::stack<GameState> gameStates;  // Stack to manage game states

// Ball class
class Ball
{
public:
    Ball(float startX, float startY, float initialSpeedX, float initialSpeedY, float initialRadius)
        : x(startX), y(startY), speedX(initialSpeedX), speedY(initialSpeedY), radius(initialRadius) {}

    // Update ball position
    void Update(float deltaTime)
    {
        x += speedX * deltaTime;
        y += speedY * deltaTime;

        // Bounce off top and bottom boundaries
        if (y < 0 || y > GetScreenHeight())
        {
            speedY *= -1;
        }
    }

    // Draw the ball
    void Draw()
    {
        DrawCircle((int)x, (int)y, radius, WHITE);
    }

    // Check collision with a paddle
    bool CheckCollisionWithPaddle(const Rectangle& paddleRect)
    {
        return CheckCollisionCircleRec(Vector2{ x, y }, radius, paddleRect);
    }

    float x;
    float y;
    float speedX;
    float speedY;
    float radius;
};

// Paddle class
class Paddle
{
public:
    Paddle(float startX, float startY, float initialWidth, float initialHeight, float initialSpeed)
        : x(startX), y(startY), width(initialWidth), height(initialHeight), speed(initialSpeed) {}

    // Update paddle position
    void Update(float deltaTime)
    {
        if (IsKeyDown(KEY_W))
        {
            y -= speed * deltaTime;
        }
        if (IsKeyDown(KEY_S))
        {
            y += speed * deltaTime;
        }

        // Ensure the paddle stays within the screen boundaries
        y = Clamp(y, height / 2, GetScreenHeight() - height / 2);
    }

    // Draw the paddle
    void Draw()
    {
        DrawRectangleRec(GetRect(), WHITE);
    }

    // Get the paddle's collision rectangle
    Rectangle GetRect()
    {
        return Rectangle{ x - width / 2, y - height / 2, width, height };
    }

    float x;
    float y;
    float width;
    float height;
    float speed;
};

// Push a new game state onto the stack
void PushGameState(GameState state)
{
    gameStates.push(state);
}

// Pop the current game state from the stack
void PopGameState()
{
    if (!gameStates.empty())
    {
        gameStates.pop();
    }
}

// Get the current game state
GameState GetCurrentGameState()
{
    if (!gameStates.empty())
    {
        return gameStates.top();
    }
    return GameState::Playing;
}

int main()
{
    // Initialize the game window
    InitWindow(800, 600, "Pong");
    SetWindowState(FLAG_VSYNC_HINT);

    // Create the ball and paddles
    Ball ball(GetScreenWidth() / 2.0f, GetScreenHeight() / 2.0f, 5, 5, 5);
    Paddle leftPaddle(50, GetScreenHeight() / 2, 10, 100, 500);
    Paddle rightPaddle(GetScreenWidth() - 50, GetScreenHeight() / 2, 10, 100, 500);

    const char* winnerText = nullptr;

    // Push the initial game state onto the stack
    PushGameState(GameState::Playing);

    while (!WindowShouldClose())
    {
        // Get the current game state
        GameState currentState = GetCurrentGameState();

        switch (currentState)
        {
        case GameState::Playing:
            // Update game elements
            ball.Update(GetFrameTime());
            leftPaddle.Update(GetFrameTime());
            rightPaddle.Update(GetFrameTime());

            // Check ball collisions with paddles
            if (ball.CheckCollisionWithPaddle(leftPaddle.GetRect()) || ball.CheckCollisionWithPaddle(rightPaddle.GetRect()))
            {
                ball.speedX *= -1.1f;
                ball.speedY = (ball.y - rightPaddle.y) / (rightPaddle.height / 2) * -ball.speedX;
            }

            // Check if the ball crosses the left boundary
            if (ball.x < 0)
            {
                winnerText = "Right Player Wins!";
                // Push the "GameOver" state onto the stack
                PushGameState(GameState::GameOver);
            }

            // Check if the ball crosses the right boundary
            if (ball.x > GetScreenWidth())
            {
                winnerText = "Left Player Wins!";
                // Push the "GameOver" state onto the stack
                PushGameState(GameState::GameOver);
            }
            break;

        case GameState::GameOver:
            if (IsKeyPressed(KEY_SPACE))
            {
                // Pop the "GameOver" state from the stack and reset the game
                PopGameState();
                winnerText = nullptr;

                ball.x = GetScreenWidth() / 2;
                ball.y = GetScreenHeight() / 2;
                ball.speedX = 5;
                ball.speedY = 5;
            }
            break;
        }

        // Begin drawing the game
        BeginDrawing();
        ClearBackground(BLACK);

        // Draw game elements
        ball.Draw();
        leftPaddle.Draw();
        rightPaddle.Draw();

        // Display the winner text (if any)
        if (winnerText)
        {
            int textWidth = MeasureText(winnerText, 60);
            DrawText(winnerText, GetScreenWidth() / 2 - textWidth / 2, GetScreenHeight() / 2 - 30, 60, YELLOW);
        }

        // Display the FPS
        DrawFPS(10, GetScreenHeight() - 30);
        EndDrawing();
    }

    // Close the game window when finished
    CloseWindow();

    return 0;
}
