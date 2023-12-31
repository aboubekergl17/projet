#include <pthread.h>
#include <semaphore.h>
#include <stdlib.h>
#include <stdio.h>

#define MAX_THREADS 10
#define MATRIX_SIZE 10

int rows1, cols1, rows2, cols2;
int buffer[MATRIX_SIZE];
int count = 0;

pthread_mutex_t mtx;
sem_t semFull, semEmpty;

int matrixB[MATRIX_SIZE][MATRIX_SIZE];
int matrixA[MATRIX_SIZE][MATRIX_SIZE];
int matrixC[MATRIX_SIZE][MATRIX_SIZE];

void *matrixMultiplier(void *args)
{
    int rowOrCol = *(int *)args;
    int result;

    for (int i = 0; i < (rowOrCol == 1 ? cols2 : rows1); i++)
    {
        result = 0;
        for (int j = 0; j < (rowOrCol == 1 ? rows2 : cols1); j++)
        {
            if (rowOrCol == 1)
                result += matrixB[rowOrCol][j] * matrixA[j][i];
            else
                result += matrixB[j][rowOrCol] * matrixA[i][j];
        }

        sem_wait(&semEmpty);
        pthread_mutex_lock(&mtx);
        buffer[count] = result;
        count++;
        pthread_mutex_unlock(&mtx);
        sem_post(&semFull);
    }

    free(args);
    pthread_exit(NULL);
}

void *resultCollector(void *args)
{
    int row = *(int *)args;
    int result;

    for (int col = 0; col < cols2; col++)
    {
        sem_wait(&semFull);
        pthread_mutex_lock(&mtx);
        count--;
        result = buffer[count];
        pthread_mutex_unlock(&mtx);
        sem_post(&semEmpty);

        matrixC[row][col] = result;
    }

    free(args);
    pthread_exit(NULL);
}

int main()
{
    rows1 = cols1 = rows2 = cols2 = 4;

    for (int i = 0; i < rows1; i++)
    {
        for (int j = 0; j < cols1; j++)
        {
            matrixB[i][j] = rand() % 100;
        }
    }

    for (int i = 0; i < rows2; i++)
    {
        for (int j = 0; j < cols2; j++)
        {
            matrixA[i][j] = rand() % 100;
        }
    }

    pthread_t producers[MAX_THREADS], consumers[MAX_THREADS];
    pthread_mutex_init(&mtx, NULL);
    sem_init(&semFull, 0, 0);
    sem_init(&semEmpty, 0, MATRIX_SIZE);

    printf("Matrix B:\n");
    for (int i = 0; i < rows1; i++)
    {
        for (int j = 0; j < cols2; j++)
        {
            printf("%d ", matrixA[i][j]);
        }
        printf("\n");
    }

    printf("Matrix C:\n");
    for (int i = 0; i < rows1; i++)
    {
        for (int j = 0; j < cols2; j++)
        {
            printf("%d ", matrixB[i][j]);
        }
        printf("\n");
    }

    for (int i = 0; i < rows1; i++)
    {
        int *a = malloc(sizeof(int));
        *a = i;
        pthread_create(&producers[i], NULL, matrixMultiplier, (void *)a);
    }

    for (int i = 0; i < cols2; i++)
    {
        int *a = malloc(sizeof(int));
        *a = i;
        pthread_create(&consumers[i], NULL, resultCollector, (void *)a);
    }

    for (int i = 0; i < rows1; i++)
    {
        pthread_join(producers[i], NULL);
    }

    for (int i = 0; i < cols2; i++)
    {
        pthread_join(consumers[i], NULL);
    }

    printf("Matrix A:\n");
    for (int i = 0; i < rows1; i++)
    {
        for (int j = 0; j < cols2; j++)
        {
            printf("%d ", matrixC[i][j]);
        }
        printf("\n");
    }

    pthread_mutex_destroy(&mtx);
    sem_destroy(&semFull);
    sem_destroy(&semEmpty);

    return 0;
}
