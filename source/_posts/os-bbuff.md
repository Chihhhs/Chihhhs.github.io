---
title: OS Bounded Buffer
date: 2024-08-26 13:57:36
tags: os
excerpt: Simulation Producer and Consumer
---

## INTRO

1. 共享記憶體的解決方案 (Share memory Solution)
    使用一個共享的Buffer（bounded buffer），這個Buffer有固定大小，生產者將數據放入Buffer，消費者從中取出數據。為了保證生產者和消費者不會同時訪問Buffer，使用互斥鎖 (mutex) 和兩個條件變量 (condition variables)，一個用於通知Buffer未滿 (not full)，另一個用於通知Buffer非空 (not empty)。

2. 生產者 (Producer)
    生產者的工作是將數據放入Buffer中，當Buffer滿時，它會等待消費者取走一些數據以騰出空間。

3. 消費者 (Consumer)
    消費者的工作是從Buffer中取出數據，當Buffer為空時，它會等待生產者放入新的數據。

4. Main Funtion
    1. 初始化 mutex
    2. `prod_thread, cons_thread` 保存新建 Thread ID , 創建生產者跟消費者 Thread
    3. 等待 Thread 完成
    4. 釋放它們佔用的資源 (destory)

5. LIB Function (pthread.h)
    1. pThread_t: `typedef __darwin_pthread_t pthread_t;`
    2. `pthread_mutex_lock` , `pthread_mutex_unlock` , `pthread_cond_wait` , `pthread_cond_signal`
    [](https://www.ibm.com/docs/zh-tw/aix/7.3?topic=p-pthread-mutex-lock-pthread-mutex-trylock-pthread-mutex-unlock-subroutine)

### IMPL

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define BUFFER_SIZE 5  // 緩衝區大小

int buffer[BUFFER_SIZE];  // share memory buffer
int count = 0;            // data amount in buffer
int in = 0;               // producer write position
int out = 0;              // consumer read position

pthread_mutex_t mutex;  // mutex lock for buffer
pthread_cond_t not_full;  // not full condition variable for buffer
pthread_cond_t not_empty; // not empty condition variable for buffer


void* producer(void* param){
    int item;
    while(1){
        item = rand() % 100; //隨機生成item
        pthread_mutex_lock(&mutex); // lock
        while(count == BUFFER_SIZE){
            //如果緩衝區已滿，等待not_full條件
            pthread_cond_wait(&not_full, &mutex);
        }
        // to buffer
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        count++;
        printf("Producer produced: %d\n", item);

        // buffer not empty signal
        pthread_cond_signal(&not_empty);
        pthread_mutex_unlock(&mutex); // unlock
        sleep(1); 
    }
}

void* consumer(void* param){
    int item;
    while (1) {
        pthread_mutex_lock(&mutex); // lock
        while (count == 0) {
            // if buffer is empty, wait for not_empty condition
            pthread_cond_wait(&not_empty, &mutex);
        }
        // get data from buffer
        item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        count--;
        printf("Consumer consumed: %d\n", item);

        // notify producer buffer not full
        pthread_cond_signal(&not_full);
        pthread_mutex_unlock(&mutex); // unlock

        sleep(1); // simulate consumption time
    }
}


int main(void){
    pthread_t producer_thread, consumer_thread;

    // intialize mutex and condition variables
    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&not_full, NULL);
    pthread_cond_init(&not_empty, NULL);

    // create producer and consumer threads
    pthread_create(&producer_thread, NULL, producer, NULL);
    pthread_create(&consumer_thread, NULL, consumer, NULL);

    // wait for threads to finish
    pthread_join(producer_thread, NULL);
    pthread_join(consumer_thread, NULL);

    // destroy mutex and condition variables
    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&not_full);

    return 0;
}
```
