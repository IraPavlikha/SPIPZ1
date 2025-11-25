# TCP/UDP Socket Tasks Documentation

## Task 1 — TCP Echo Server and Client

### server.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int listenfd, connfd, n;
    struct sockaddr_in servaddr;
    char buffer[1024];

    if((listenfd = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket");
        exit(1);
    }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(51000);

    if(bind(listenfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0) {
        perror("bind");
        close(listenfd);
        exit(1);
    }

    if(listen(listenfd, 5) < 0) {
        perror("listen");
        close(listenfd);
        exit(1);
    }

    printf("TCP Echo Server started on port 51000...\n");

    while(1) {
        if((connfd = accept(listenfd, (struct sockaddr*)NULL, NULL)) < 0) {
            perror("accept");
            continue;
        }

        while((n = read(connfd, buffer, sizeof(buffer)-1)) > 0) {
            buffer[n] = '\0';
            printf("Received: %s", buffer);

            if(write(connfd, buffer, n) < 0) {
                perror("write");
                break;
            }
        }

        close(connfd);
    }

    close(listenfd);
    return 0;
}
```

### client.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv) {
    int sockfd, n, i;
    char sendline[1000], recvline[1000];
    struct sockaddr_in servaddr;

    if(argc != 2) {
        printf("Usage: %s <IP-address>\n", argv[0]);
        exit(1);
    }

    bzero(sendline, sizeof(sendline));
    bzero(recvline, sizeof(recvline));

    if((sockfd = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket");
        exit(1);
    }

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(51000);
    if(inet_aton(argv[1], &servaddr.sin_addr) == 0) {
        printf("Invalid IP-address!\n");
        close(sockfd);
        exit(1);
    }

    if(connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0) {
        perror("connect");
        close(sockfd);
        exit(1);
    }

    for(i = 0; i < 3; i++) {
        printf("string=>");
        fgets(sendline, sizeof(sendline), stdin);

        if(write(sockfd, sendline, strlen(sendline)) < 0) {
            perror("write");
            close(sockfd);
            exit(1);
        }

        if((n = read(sockfd, recvline, sizeof(recvline)-1)) < 0) {
            perror("read");
            close(sockfd);
            exit(1);
        }

        recvline[n] = '\0';
        printf("%s", recvline);
    }

    close(sockfd);
    return 0;
}
```

**Особливості:**
- Сервер працює у нескінченному циклі, приймаючи підключення і надсилаючи echo.
- Клієнт підключається до порту 51000, надсилає повідомлення і отримує echo.
- Використовується буфер 1024 байти.
- Обробка помилок через `perror()`.

---

## Task 2 — TCP Client (Port 51000)
### client.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char **argv) {
    int sockfd, n;
    char sendline[1024], recvline[1024];
    struct sockaddr_in servaddr;

    if(argc != 2) {
        printf("Usage: %s <ServerIP>\n", argv[0]);
        exit(1);
    }

    sockfd = socket(PF_INET, SOCK_STREAM, 0);
    if(sockfd < 0) { perror("socket"); exit(1); }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(51000);

    if(inet_aton(argv[1], &servaddr.sin_addr) == 0) {
        printf("Invalid IP address\n"); close(sockfd); exit(1);
    }

    if(connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0) {
        perror("connect"); close(sockfd); exit(1);
    }

    printf("Enter message: ");
    fgets(sendline, sizeof(sendline), stdin);

    if(write(sockfd, sendline, strlen(sendline)) < 0) { perror("write"); close(sockfd); exit(1); }

    n = read(sockfd, recvline, sizeof(recvline)-1);
    if(n < 0) { perror("read"); close(sockfd); exit(1); }

    recvline[n] = '\0';
    printf("Echo from server: %s\n", recvline);

    close(sockfd);
    return 0;
}
```

### server.c — same as Task 1 (port 51000)

---

## Task 3 — TCP Client (Port 51000, modified for test)
**client.c** — модифікація Task 2 клієнта, порт 51000
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char **argv) {
    int sockfd, n;
    char sendline[1024], recvline[1024];
    struct sockaddr_in servaddr;

    if(argc != 2) { printf("Usage: %s <ServerIP>\n", argv[0]); exit(1); }

    sockfd = socket(PF_INET, SOCK_STREAM, 0);
    if(sockfd < 0) { perror("socket"); exit(1); }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(51000);

    if(inet_aton(argv[1], &servaddr.sin_addr) == 0) { printf("Invalid IP address\n"); close(sockfd); exit(1); }

    if(connect(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0) { perror("connect"); close(sockfd); exit(1); }

    printf("Enter message: ");
    fgets(sendline, sizeof(sendline), stdin);

    if(write(sockfd, sendline, strlen(sendline)) < 0) { perror("write"); close(sockfd); exit(1); }

    n = read(sockfd, recvline, sizeof(recvline)-1);
    if(n < 0) { perror("read"); close(sockfd); exit(1); }

    recvline[n] = '\0';
    printf("Echo from server: %s\n", recvline);

    close(sockfd);
    return 0;
}
```

---

## Task 4 — Parallel TCP Server
**server.c**
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <signal.h>
#include <arpa/inet.h>
#include <sys/wait.h>

#define PORT 51000
#define BUF_SIZE 1024

void handle_client(int sockfd) {
    char buf[BUF_SIZE];
    int n;
    while ((n = read(sockfd, buf, BUF_SIZE-1)) > 0) {
        buf[n] = '\0';
        printf("Received from client: %s", buf);
        if (write(sockfd, buf, n) < 0) { perror("write"); break; }
    }
    if (n < 0) perror("read");
    close(sockfd);
    exit(0);
}

void sigchld_handler(int sig) {
    while(waitpid(-1, NULL, WNOHANG) > 0);
}

int main() {
    int sockfd, newsockfd;
    struct sockaddr_in servaddr, cliaddr;
    socklen_t clilen;

    signal(SIGCHLD, sigchld_handler);

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) { perror("socket"); exit(1); }

    int opt = 1;
    setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(PORT);

    if (bind(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0) { perror("bind"); close(sockfd); exit(1); }

    if (listen(sockfd, 5) < 0) { perror("listen"); close(sockfd); exit(1); }

    printf("TCP Server started on port %d...\n", PORT);

    while (1) {
        clilen = sizeof(cliaddr);
        newsockfd = accept(sockfd, (struct sockaddr*)&cliaddr, &clilen);
        if (newsockfd < 0) { perror("accept"); continue; }

        if (fork() == 0) {
            close(sockfd);
            handle_client(newsockfd);
        } else {
            close(newsockfd);
        }
    }

    close(sockfd);
    return 0;
}
```

**Особливості паралельної обробки:**
- Сервер використовує fork() для кожного клієнта.
- Дочірні процеси обробляють клієнтів, батьківський процес продовжує слухати нові з'єднання.
- Використовується `signal(SIGCHLD, sigchld_handler)` та `waitpid(..., WNOHANG)` для видалення зомбі-процесів.
- Псевдопаралельна робота: одночасно можуть обслуговуватись декілька клієнтів.
- Сервер відправляє echo кожному клієнту, друкує повідомлення для контр