# Parallel-Programming-Guess-Game-in-C-Linux

Playing a guess game with processes and threads. Processes and threads will try to guess the number that is chosen by computer from the interval from 1 to 99. The closest guess gets the point. The possible numbers include from [1,99].

Use and create multiple processes and threads inside a process and gain some experience about basics of parallel programming. 


## a)	guess_process.c ; Game with 2 processes
## b)	guess_threads.c ; Game with 3 threads

### Pseudocode for (a):
1.	Main process fills an array of size 5 with random numbers from 1 to 99.
2.	Main process creates a new child process with fork() 
3.	Child process selects a number randomly
4.	Parent process will select a number randomly
5.	Then they will compare their numbers with the first element of the filled array (in the second iteration of the loop, they will compare with second elements, in the third iteration compare with third element of the array, and so forth)
6.	Score of the winning process to be increased
7.	Game will end after 5 guesses.

### Pseudocode for (b):
1.	Main process fills an array of size 5 with random numbers from 1 to 99.
2.	Main process creates 3 threads with pthread_create()
3.	Threads selects a number randomly
4.	Then they will compare their numbers with the first element of the filled array (in the second iteration of the loop, they will compare with second elements, in the third iteration compare with third element of the array, and so forth)
5.	Score of the winning threads will be increased 
6.	Game will end after 5 guesses.

## PART A

```C++
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <time.h>
#include <sys/types.h>
#include <sys/wait.h>
#define READ_END 0
#define WRITE_END 1

int main()
{
    pid_t pid;
    int fd[2];
    int randArray[5];
    int randNumW, childNum, parentNum, status;
    srand(time(NULL));
    printf("The game has launched\n");
    for (int i = 0; i < 5; i++)
    {
        randArray[i] = rand() % 99 + 1;
    }
    printf("Computer Choses [%d, %d, %d, %d, %d] randomly\n", randArray[0], randArray[1], randArray[2], randArray[3], randArray[4]);
    printf("Child process is created\n");
    printf("The game starts\n--\n");

    if (pipe(fd) == -1)
    {
        fprintf(stderr, "Pipe failed");
        return 1;
    }
    pid = fork();

    if (pid == 0)
    {
        wait(NULL);
        srand(time(NULL) ^ getpid());
        for (int i = 0; i < 5; i++)
        {
            randNumW = (rand() % 99) + 1;
            write(fd[WRITE_END], &randNumW, sizeof(randNumW));
        }
    }
    else if (pid > 0)
    {
        waitpid(pid, &status, 0);
        srand(time(NULL) ^ getpid());
        int homeScore = 0, awayScore = 0;
        for (int i = 0; i < 5; i++)
        {
            wait(NULL);
            parentNum = rand() % 99 + 1;

            read(fd[READ_END], &childNum, sizeof(childNum));

            printf("Turn %d, Parent guess: %d, Child guess: %d\n", i + 1, parentNum, childNum);
            if (abs(randArray[i] - childNum) < abs(randArray[i] - parentNum))
            {
                awayScore++;
                printf("Child win, Score: %d â€“ %d, %d is closer to %d than %d\nâ€“\n", homeScore, awayScore, childNum, randArray[i], parentNum);
            }
            else if (abs(randArray[i] - childNum) > abs(randArray[i] - parentNum))
            {
                homeScore++;
                printf("Parent win, Score: %d â€“ %d, %d is closer to %d than %d\n--\n", homeScore, awayScore, parentNum, randArray[i], childNum);
            }
            else
            {
                homeScore++;
                awayScore++;
                printf("Draw, Score: %d â€“ %d, %d is equal to %d\n--\n", homeScore, awayScore, childNum, parentNum);
            }
        }
        if (homeScore > awayScore)
            printf("--\nParent has won the game with score: %d â€“ %d .\n--\n", homeScore, awayScore);
        else
            printf("--\nChild has won the game with score: %d â€“ %d .\n--\n", homeScore, awayScore);

        printf("Child has terminated\n");
        printf("Parent waits child with wait()\n");
        printf("Parent has terminated\n");
    }

    return 0;
}
```

## PART B

```C++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

static char guessArr[15][5];
static int count = 0;
pthread_mutex_t lock;
void *myThread(void *arg)
{
    int argInt = *(int *)arg;
    pthread_mutex_lock(&lock);
    for (int i = 0; i < 5; i++)
    {
        guessArr[argInt][i] = (rand() % 99) + 1;
    }
    pthread_mutex_unlock(&lock);
    return NULL;
}
void *myThread2(void *arg)
{
    int argInt = *(int *)arg;
    pthread_mutex_lock(&lock);
    if(argInt != 2 || count != 4)
        printf("%d_%d, ", argInt + 1, guessArr[argInt][count]);
    else
        printf("%d_%d", argInt + 1, guessArr[argInt][count]);
    if(argInt == 2)
        count++;
    if(count == 5)
        count = 0;
    pthread_mutex_unlock(&lock);
    
    return NULL;
}

int main()
{
    pthread_t tid[3], tid2[3];
    int randArray[5];
    int randNumW, childNum, parentNum, status, thr1Score = 0, thr2Score = 0, thr3Score = 0;
    int index[3];

    srand(time(NULL));
    printf("The game has launched\n");
    for (int i = 0; i < 5; i++)
    {
        randArray[i] = rand() % 100;
    }
    printf("Computer Choses [%d, %d, %d, %d, %d] randomly\n",randArray[0],randArray[1],randArray[2],randArray[3],randArray[4]);
    printf("3 threads will be created\n");
    printf("The game starts\n--\n");
    for (int i = 0; i < 3; i++)
    {
        index[i] = i;
        pthread_create(&tid[i], NULL, myThread, &index[i]);
    }
    for (int i = 0; i < 3; i++)
        pthread_join(tid[i], NULL);

    for (int i = 0; i < 5; i++)
    {

        int thr1 = guessArr[0][i];
        int thr2 = guessArr[1][i];
        int thr3 = guessArr[2][i];
        int thr1Abs = abs(randArray[i] - thr1);
        int thr2Abs = abs(randArray[i] - thr2);
        int thr3Abs = abs(randArray[i] - thr3);

        printf("Turn %d, Guesses: 1.Thread: %d, 2.Thread: %d, 3.Thread: %d, \n–-\n", i + 1, thr1, thr2, thr3);
        if (thr1Abs < thr2Abs && thr1Abs < thr3Abs)
        {
            thr1Score++;
            printf("1.thread win, Score: %d – %d - %d, %d is closest to %d, \n–-\n", thr1Score, thr2Score, thr3Score, thr1, randArray[i]);
        }
        else if (thr2Abs < thr1Abs && thr2Abs < thr3Abs)
        {
            thr2Score++;
            printf("2.thread win, Score: %d – %d - %d, %d is closest to %d, \n-–\n", thr1Score, thr2Score, thr3Score, thr2, randArray[i]);
        }
        else if (thr3Abs < thr1Abs && thr3Abs < thr2Abs)
        {
            thr3Score++;
            printf("3.thread win, Score: %d – %d - %d, %d is closest to %d, \n-–\n", thr1Score, thr2Score, thr3Score, thr3, randArray[i]);
        }
        else if (thr1Abs == thr2Abs && thr1Abs == thr3Abs)
        {
            thr1Score++;thr2Score++;thr3Score++;
            printf("1.thread and 2.thread and 3.thread win, Score: %d – %d - %d, %d is closest to %d, \n–-\n", thr1Score, thr2Score, thr3Score, thr1, randArray[i]);
        }
        else if (thr1Abs == thr2Abs && thr1Abs < thr3Abs)
        {
            thr1Score++;
            thr2Score++;
            printf("1.thread and 2.thread win, Score: %d – %d - %d, %d is closest to %d, \n–-\n", thr1Score, thr2Score, thr3Score, thr1, randArray[i]);
        }
        else if (thr2Abs == thr3Abs && thr2Abs < thr1Abs)
        {
            thr2Score++;
            thr3Score++;
            printf("2.thread and 3.thread win, Score: %d – %d - %d, %d is closest to %d, \n-–\n", thr1Score, thr2Score, thr3Score, thr2, randArray[i]);
        }
        else if (thr1Abs == thr3Abs && thr1Abs < thr2Abs)
        {
            thr1Score++;
            thr3Score++;
            printf("1.thread and 3.thread win, Score: %d – %d - %d, %d is closest to %d, \n–-\n", thr1Score, thr2Score, thr3Score, thr1, randArray[i]);
        }
    }
    if (thr1Score == thr2Score && thr1Score == thr3Score)
        printf("–-\n1.Thread and 2.Thread and 3. Thread won the game with score: Score: %d – %d – %d.\n–-\n-–\n", thr1Score, thr2Score, thr3Score);
    else if (thr1Score == thr2Score && thr1Score > thr3Score)
        printf("–-\n1.Thread and 2.Thread won the game with score: Score: %d – %d – %d.\n–-\n-–\n", thr1Score, thr2Score, thr3Score);
    else if (thr1Score == thr3Score && thr1Score > thr2Score)
        printf("-–\n1.Thread and 3.Thread won the game with score: Score: %d – %d – %d.\n–-\n–-\n", thr1Score, thr2Score, thr3Score);
    else if (thr2Score == thr3Score && thr2Score > thr1Score)
        printf("–-\n2.Thread and 3.Thread won the game with score: Score: %d – %d – %d.\n–-\n-–\n", thr1Score, thr2Score, thr3Score);
    else if (thr1Score > thr3Score && thr1Score > thr2Score)
        printf("-–\n1.Thread won the game with score: Score: %d – %d – %d.\n-–\n–-\n", thr1Score, thr2Score, thr3Score);
    else if (thr2Score > thr1Score && thr2Score > thr3Score)
        printf("-–\n2.Thread won the game with score: Score: %d – %d – %d.\n-–\n–-\n", thr1Score, thr2Score, thr3Score);
    else if (thr3Score > thr1Score && thr3Score > thr2Score)
        printf("–-\n3.Thread won the game with score: Score: %d – %d – %d.\n-–\n–-\n", thr1Score, thr2Score, thr3Score);
    else
        printf("something forgotten!");
    printf("Guesses made by threads: [");
    for (int i = 0; i < 5; i++)
    {
        for (int i = 0; i < 3; i++)
        {
            index[i] = i;
            pthread_create(&tid2[i], NULL, myThread2, &index[i]);
        }
        for (int i = 0; i < 3; i++)
            pthread_join(tid[i], NULL);
    }

    printf("]\n-–\n-–\n");
    printf("1.Thread terminated\n2.Thread terminated\n3.Thread terminated\nThreads are joined by main process\nGame finished\n");
    pthread_exit(NULL);
    return 0;
}
```

## Example

- gcc guess_process.c -o guess_process
- ./guess_process
```
The game has launched
Computer choses [56, 1, 33, 90, 77] randomly
Child process is created
The game starts


Turn 1, Parent guess: 41, Child guess: 20
Parent win, Score: 1 – 0, 41 is closer to 56 than 20


Turn 2, Parent guess: 52, Child guess: 45
Child win, Score: 1 – 1, 45 is closer to 1 than 52


Turn 5, Parent guess: 3, Child guess: 9
Child win, Score: 2 – 3, 9 is closer to 77 than 3


Child has won the game with score: 2 – 3 .


Child has terminated
Parent waits child with wait()
Parent has terminated


```
- gcc guess_thread.c -o guess_thread –lpthread
- ./guess_thread
```
The game has launched
Computer choses [6, 99, 13, 49, 87] randomly
3 threads will be created 
The game starts


Turn 1, Guesses: 1.thread: 11, 2.thread: 66, 3.thread: 80,
1.thread win, Score: 1 – 0 - 0, 11 is closest to 6


Turn 5, Guesses: 1.thread: 9, 2.thread: 56, 3.thread: 13,
2.thread win, Score: 1 – 2 - 2, 56 is closest to 87


2.Thread and 3.Thread won the game with score: Score: 1 – 2 – 2.


Guesses made by threads: [2_66, 1_11, 3_80, …, 3_13, 1_9, 2_56]


1.Thread terminated
2.Thread terminated
3.Thread terminated
Threads are joined by main process
Game finished
```
