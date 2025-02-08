# Hangman-Game

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <time.h>

#define MAX_WORD_LENGTH 50
#define MAX_TRIES 6
#define MAX_SCORES 10

struct WordWithHint {
    char word[MAX_WORD_LENGTH];
    char hint[MAX_WORD_LENGTH];
};

struct Category {
    char name[MAX_WORD_LENGTH];
    struct WordWithHint words[10];
};

struct HighScore {
    char playerName[50];
    int score;
};

struct Category categories[] = {
    { "Sports", {
        { "football", "A game played with a ball and two goals." },
        { "tennis", "A game played with rackets and a net." },
        { "cricket", "A bat-and-ball game." },
        { "hockey", "A game played with sticks and a puck." },
        { "basketball", "A game with a hoop and ball." }
    }},
    { "Food", {
        { "pizza", "A popular Italian dish." },
        { "burger", "A sandwich with a patty." },
        { "pasta", "An Italian dish of noodles." },
        { "sushi", "A Japanese dish with rice and fish." },
        { "icecream", "A frozen sweet dessert." }
    }},
    { "Countries", {
        { "japan", "An island nation in Asia." },
        { "france", "Home of the Eiffel Tower." },
        { "australia", "A country and continent." },
        { "china", "The world's most populous country." },
        { "india", "A country with diverse cultures." }
    }},
    { "Colors", {
        { "red", "The color of passion and fire." },
        { "blue", "The color of the sky." },
        { "green", "The color of nature." },
        { "yellow", "A bright, cheerful color." },
        { "orange", "A mix of red and yellow." }
    }},
    { "Animals", {
        { "lion", "The king of the jungle." },
        { "tiger", "A large striped cat." },
        { "elephant", "A large mammal with a trunk." },
        { "dog", "Man's best friend." },
        { "cat", "A common pet that purrs." }
    }}
};

void displayMainMenu();
void playGame(const struct Category* categories, char *playerName, int *playerScore);
void displayWord(const char word[], const int guessed[]);
void drawHangman(int tries);
void displayHighScore();
void saveHighScore(const char* name, int score, int gameFinished);
void loadHighScore(struct HighScore highScores[], int* highScoreCount);

int main() {
    srand(time(NULL));

    char playerName[50];
    int playerScore = 0;

    printf("Welcome to the Hangman Game!\n");
    printf("Enter your name: ");
    fgets(playerName, sizeof(playerName), stdin);
    playerName[strlen(playerName)-1] = '\0';

    while (1) {
        displayMainMenu();
        int choice;
        printf("\nEnter your choice: ");
        scanf("%d", &choice);
        while ((getchar()) != '\n');

        switch (choice) {
            case 1:
                playGame(categories, playerName, &playerScore);
                break;
            case 2:
                displayHighScore();
                break;
            case 3:
                saveHighScore(playerName, playerScore, 1);
                printf("Exiting the game...\n");
                return 0;
            default:
                printf("Invalid choice, please try again.\n");
        }
    }
}

void displayMainMenu() {
    printf("\nMain Menu:\n");
    printf("1. Play Game\n");
    printf("2. View High Scores\n");
    printf("3. Exit\n");
}

void displayWord(const char word[], const int guessed[]) {
    for (int i = 0; i < strlen(word); i++) {
        if (guessed[word[i] - 'a']) {
            printf("%c ", word[i]);
        } else {
            printf("_ ");
        }
    }
    printf("\n");
}

void drawHangman(int tries) {
    printf("\n+---+\n");
    printf("  |   |\n");
    printf("  %s   |\n", (tries > 0) ? "O" : " ");  // Head
    printf(" %s%s%s  |\n", (tries > 2) ? "/" : " ", (tries > 1) ? "|" : " ", (tries > 3) ? "\\" : " ");  // Arms
    printf(" %s %s  |\n", (tries > 4) ? "/" : " ", (tries > 5) ? "\\" : " ");  // Legs
    printf("      |\n");
    printf("=========\n");
}

void playGame(const struct Category* categories, char *playerName, int *playerScore) {
    int categoryChoice;

    printf("\nChoose a category:\n");
    for(int i = 0; i < 5; i++) {
    printf("%d. %s\n",i + 1 , categories[i].name);
    }

    printf("Enter your choice (1-5): ");
    scanf("%d", &categoryChoice);
    while ((getchar()) != '\n');

    if (categoryChoice < 1 || categoryChoice > 5) {
        printf("Invalid choice. Returning to the main menu...\n");
        return;
    }

    const struct Category* selectedCategory = &categories[categoryChoice - 1];
    int correctGuesses = 0;
    int tries = 0;

    while (1) {
        const struct WordWithHint* wordData = &selectedCategory->words[rand() % 5];    // randomly chooses a word from selected category
        const char* word = wordData->word;  // stores word
        const char* hint = wordData->hint;  // stores hint

        int wordLength = strlen(word);
        int guessedLetters[26] = {0};
        correctGuesses = 0;
        tries = 0;

        printf("\nCategory: %s\nHint: %s\n", selectedCategory->name, hint);

        while (tries < MAX_TRIES) {
            printf("\nWord: ");
            displayWord(word, guessedLetters);
            drawHangman(tries);

            char guess;
            printf("Enter your guess: ");
            scanf(" %c", &guess);
            guess = tolower(guess);

            if (!isalpha(guess)) {
                printf("Invalid input. Please enter a letter.\n");
                continue;
            }

            if (guessedLetters[guess - 'a']) {
                printf("You already guessed that letter. Try again.\n");
                continue;
            }

            guessedLetters[guess - 'a'] = 1;
            int found = 0;

            for (int i = 0; i < wordLength; i++) {
                if (word[i] == guess) {
                    found = 1;
                    correctGuesses++;
                    (*playerScore)++;
                }
            }

            if (found) {
                printf("You're Right!\n");
                if (correctGuesses == wordLength) {
                    printf("\nCongratulations, %s! You guessed the word: %s\n", playerName, word);
                    break;
                }
            } else {
                printf("Wrong guess!\n");
                tries++;
            }
        }

        if (tries == MAX_TRIES) {
            drawHangman(tries);
            printf("Game over!\nThe correct word was: %s\n", word);
            saveHighScore(playerName, *playerScore, 1);
            break;
        }

        char playAgain;
        printf("\nDo you want to continue to the next word? (y/n): ");
        scanf(" %c", &playAgain);

        if (tolower(playAgain) != 'y') {
            saveHighScore(playerName, *playerScore, 1);
            break;
        }
    }
}

void saveHighScore(const char* name, int score, int gameFinished) {
    struct HighScore highScores[MAX_SCORES];
    int highScoreCount = 0;

    loadHighScore(highScores, &highScoreCount);

    int found = 0;
    for (int i = 0; i < highScoreCount; i++) {
        if (strcmp(highScores[i].playerName, name) == 0) {
            if (gameFinished && score > highScores[i].score) {
                highScores[i].score = score;
            }
            found = 1;
            break;
        }
    }

    if (!found && gameFinished) {
        if (highScoreCount < MAX_SCORES) {
            strcpy(highScores[highScoreCount].playerName, name);
            highScores[highScoreCount].score = score;
            highScoreCount++;
        }
    }

    FILE* file = fopen("highscore.txt", "w");
    if (!file) {
        printf("Error saving high score.\n");
        return;
    }

    for (int i = 0; i < highScoreCount; i++) {
        fprintf(file, "%s %d\n", highScores[i].playerName, highScores[i].score);
    }

    fclose(file);
}

void loadHighScore(struct HighScore highScores[], int* highScoreCount) {
    FILE* file = fopen("highscore.txt", "r");
    if (!file) {
        return;
    }

    while (fscanf(file, "%s %d", highScores[*highScoreCount].playerName, &highScores[*highScoreCount].score) == 2) {
        (*highScoreCount)++;
    }

    fclose(file);
}

void displayHighScore() {
    struct HighScore highScores[MAX_SCORES];
    int highScoreCount = 0;

    loadHighScore(highScores, &highScoreCount);

    printf("\nHigh Scores:\n");
    printf("=====================\n");
    for (int i = 0; i < highScoreCount; i++) {
        printf("%s: %d\n", highScores[i].playerName, highScores[i].score);
    }
    printf("=====================\n");
}
