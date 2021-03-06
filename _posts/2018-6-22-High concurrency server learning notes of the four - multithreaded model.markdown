---
layout: post
title: 高并发服务器学习笔记之四：多线程模型
date: 发布于2018-06-22 14:47:15 +0800
categories: 一步步打造高并发服务器
tag: 4
---

* content
{:toc}

该模型和多进程模型的思想类似，只是把进程换成了线程，因为线程的创建比进程创建开销小。但这并不是说多线程就一定比多进程优秀，进程和线程都有各自的优缺点，具体请自行查阅线程和进程相关的内容，完整代码[戳这里](https://github.com/zhangn1989/MyRPC)​​​​​​​

<!-- more -->

    
    
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <errno.h>
    
    #include <pthread.h>
    #include <semaphore.h>
    #include <unistd.h>
    #include <sys/types.h>          /* See NOTES */
    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <netinet/ip.h> /* superset of previous */
    #include <arpa/inet.h>
    
    #include "public_head.h"
    
    #define LISTEN_BACKLOG 50
    
    sem_t sem;
    
    void *thread_func(void *arg)
    {
        int acceptfd = *(int *)arg;
    
        //sem += 1
        sem_post(&sem);
    
        int i = 0;
        ssize_t readret = 0;
        char read_buff[256] = { 0 };
        char write_buff[256] = { 0 };
       
    	while (1)
    	{
    		memset(read_buff, 0, sizeof(read_buff));
    		readret = read(acceptfd, read_buff, sizeof(read_buff));
    		if (readret == 0)
    			break;
    
    		printf("thread id:%lu, recv message:%s\n", pthread_self(), read_buff);
    
    		memset(write_buff, 0, sizeof(write_buff));
    		sprintf(write_buff, "This is server send message:%d", i++);
    		write(acceptfd, write_buff, sizeof(write_buff));
    	}
        printf("\n");
        close(acceptfd);
        return NULL;
    }
    
    int main(int argc, char ** argv)
    {
        int sockfd = 0;
        int acceptfd = 0;
        socklen_t client_addr_len = 0;
        struct sockaddr_in server_addr, client_addr;
    
        char client_ip[16] = { 0 };
    
        pthread_t tid;
    
        memset(&server_addr, 0, sizeof(server_addr));
        memset(&client_addr, 0, sizeof(client_addr));
    
        if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
            handle_error("socket");
    
        server_addr.sin_family = AF_INET;
        server_addr.sin_port = htons(9527);
        server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
        if(bind(sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0)
        {
    		char buff[256] = { 0 };
            close(sockfd);
    		strerror_r(errno, buff, sizeof(buff));
            handle_error("bind");
        }
    
        if(listen(sockfd, LISTEN_BACKLOG) < 0)
        {
            close(sockfd);
            handle_error("listen");
        }
    
        sem_init(&sem, 0, 0);
    
    
        while(1)
        {
            client_addr_len = sizeof(client_addr);
            if((acceptfd = accept(sockfd, (struct sockaddr *)&client_addr, &client_addr_len)) < 0)
            {
                perror("accept");
                continue;
            }
           
            memset(client_ip, 0, sizeof(client_ip));
            inet_ntop(AF_INET,&client_addr.sin_addr,client_ip,sizeof(client_ip)); 
            printf("client:%s:%d\n",client_ip,ntohs(client_addr.sin_port));
    
            if(pthread_create(&tid, NULL, thread_func, &acceptfd) != 0)
            {
    			handle_warning("pthread_create");
                close(acceptfd);
                continue;
            }
    
    		//wait thread start
            //if(sem == 0) block
            //wait for sem != 0
            //unblock, sem -= 1
            sem_wait(&sem);
        }
        
        sem_destroy(&sem);
        close(sockfd);
    
        return 0;
    }
    

