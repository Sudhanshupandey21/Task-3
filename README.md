# Task-3


#include <ncurses.h>
#include <stdlib.h>
#include <time.h>

#define MAX_SNAKE_LENGTH 100

typedef struct {
    int x;
    int y;
} Point;

typedef struct {
    Point body[MAX_SNAKE_LENGTH];
    int length;
    int direction;
} Snake;

typedef struct {
    Point position;
} Food;

void initializeGame(Snake *snake, Food *food);
void generateFood(Food *food, Snake *snake);
void draw(Snake *snake, Food *food);
void update(Snake *snake, Food *food, int *gameOver);
int checkCollision(Snake *snake);

int main() {
    Snake snake;
    Food food;
    int gameOver = 0;
    int ch;

    initscr();
    clear();
    noecho();
    cbreak();
    timeout(100); // Snake speed
    keypad(stdscr, TRUE);
    curs_set(0);

    initializeGame(&snake, &food);

    while (!gameOver) {
        ch = getch();
        switch(ch) {
            case KEY_UP:
                if (snake.direction != KEY_DOWN)
                    snake.direction = KEY_UP;
                break;
            case KEY_DOWN:
                if (snake.direction != KEY_UP)
                    snake.direction = KEY_DOWN;
                break;
            case KEY_LEFT:
                if (snake.direction != KEY_RIGHT)
                    snake.direction = KEY_LEFT;
                break;
            case KEY_RIGHT:
                if (snake.direction != KEY_LEFT)
                    snake.direction = KEY_RIGHT;
                break;
            case 'q':
                gameOver = 1;
                break;
        }

        update(&snake, &food, &gameOver);
        draw(&snake, &food);
    }

    endwin();
    return 0;
}

void initializeGame(Snake *snake, Food *food) {
    snake->length = 1;
    snake->body[0].x = COLS / 2;
    snake->body[0].y = LINES / 2;
    snake->direction = KEY_RIGHT;

    generateFood(food, snake);
}

void generateFood(Food *food, Snake *snake) {
    int validPosition = 0;
    while (!validPosition) {
        validPosition = 1;
        food->position.x = rand() % COLS;
        food->position.y = rand() % LINES;

        for (int i = 0; i < snake->length; i++) {
            if (food->position.x == snake->body[i].x && food->position.y == snake->body[i].y) {
                validPosition = 0;
                break;
            }
        }
    }
}

void draw(Snake *snake, Food *food) {
    clear();
    mvprintw(food->position.y, food->position.x, "O");
    for (int i = 0; i < snake->length; i++) {
        mvprintw(snake->body[i].y, snake->body[i].x, "X");
    }
    refresh();
}

void update(Snake *snake, Food *food, int *gameOver) {
    Point next = snake->body[0];

    switch(snake->direction) {
        case KEY_UP:
            next.y--;
            break;
        case KEY_DOWN:
            next.y++;
            break;
        case KEY_LEFT:
            next.x--;
            break;
        case KEY_RIGHT:
            next.x++;
            break;
    }

    if (next.x < 0 || next.x >= COLS || next.y < 0 || next.y >= LINES) {
        *gameOver = 1;
        return;
    }

    for (int i = 0; i < snake->length; i++) {
        if (next.x == snake->body[i].x && next.y == snake->body[i].y) {
            *gameOver = 1;
            return;
        }
    }

    for (int i = snake->length; i > 0; i--) {
        snake->body[i] = snake->body[i - 1];
    }

    snake->body[0] = next;

    if (next.x == food->position.x && next.y == food->position.y) {
        snake->length++;
        if (snake->length < MAX_SNAKE_LENGTH) {
            generateFood(food, snake);
        }
    }
}

int checkCollision(Snake *snake) {
    Point head = snake->body[0];

    if (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= LINES) {
        return 1;
    }

    for (int i = 1; i < snake->length; i++) {
        if (head.x == snake->body[i].x && head.y == snake->body[i].y) {
            return 1;
        }
    }

    return 0;
}
