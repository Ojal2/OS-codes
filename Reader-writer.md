```C

#include <pthread.h>
#include <semaphore.h>
#include <stdio.h>

sem_t rw_mutex;    // Used by both readers and writers
sem_t mutex;       // Used by readers to update read count
int read_count = 0;  // Number of readers currently reading

void *writer(void *wno) {
    sem_wait(&rw_mutex);  // Writer enters critical section

    printf("Writer %d is writing\n", (*((int *)wno)));

    sem_post(&rw_mutex);  // Writer exits critical section
}

void *reader(void *rno) {
    sem_wait(&mutex);  // Reader updates read_count
    read_count++;
    if (read_count == 1) {
        sem_wait(&rw_mutex);  // If this is the first reader, block writers
    }
    sem_post(&mutex);

    printf("Reader %d is reading\n", (*((int *)rno)));

    sem_wait(&mutex);  // Reader finishes reading
    read_count--;
    if (read_count == 0) {
        sem_post(&rw_mutex);  // If this is the last reader, allow writers
    }
    sem_post(&mutex);
}

int main() {
    pthread_t read[10], write[5];  // Create threads for readers and writers
    sem_init(&rw_mutex, 0, 1);     // Initialize semaphore for writer access
    sem_init(&mutex, 0, 1);        // Initialize semaphore for read_count update

    int reader_num[10], writer_num[5];
    for (int i = 0; i < 10; i++) {
        reader_num[i] = i + 1;
    }
    for (int i = 0; i < 5; i++) {
        writer_num[i] = i + 1;
    }

    // Create reader and writer threads
    for (int i = 0; i < 5; i++) {
        pthread_create(&write[i], NULL, writer, (void *)&writer_num[i]);
    }
    for (int i = 0; i < 10; i++) {
        pthread_create(&read[i], NULL, reader, (void *)&reader_num[i]);
    }

    // Join threads
    for (int i = 0; i < 5; i++) {
        pthread_join(write[i], NULL);
    }
    for (int i = 0; i < 10; i++) {
        pthread_join(read[i], NULL);
    }

    // Destroy semaphores
    sem_destroy(&rw_mutex);
    sem_destroy(&mutex);

    return 0;
}

