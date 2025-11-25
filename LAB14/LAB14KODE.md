**Task 1**

### udp_echo_server.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
    int sockfd, n;
    char buffer[1000];
    struct sockaddr_in servaddr, cliaddr;
    socklen_t len;

    if((sockfd = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        perror("socket");
        exit(1);
    }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(7); // порт echo

    if(bind(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)) < 0)
    {
        perror("bind");
        close(sockfd);
        exit(1);
    }

    printf("UDP Echo server started on port 7...\n");

    while(1)
    {
        len = sizeof(cliaddr);
        n = recvfrom(sockfd, buffer, sizeof(buffer), 0, (struct sockaddr*)&cliaddr, &len);
        if(n < 0) 
        {
            perror("recvfrom");
            continue;
        }
        sendto(sockfd, buffer, n, 0, (struct sockaddr*)&cliaddr, len);
    }

    close(sockfd);
    return 0;
}
```

UDP-сервер працює на порті 7, як вимагає стандарт сервісу echo. Клієнт може підключатися до будь-якої IP-адреси сервера. Сервер отримує датаграми від клієнта і повертає їх без змін.

---

**Task 2-3**

### udp_server.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
    int sockfd, n;
    socklen_t clilen;
    char line[1000];
    struct sockaddr_in servaddr, cliaddr;

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(51000);
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

    if((sockfd = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        perror("socket");
        exit(1);
    }

    if(bind(sockfd, (struct sockaddr*) &servaddr, sizeof(servaddr)) < 0)
    {
        perror("bind");
        close(sockfd);
        exit(1);
    }

    while(1)
    {
        clilen = sizeof(cliaddr);
        n = recvfrom(sockfd, line, 999, 0, (struct sockaddr*) &cliaddr, &clilen);
        if(n < 0)
        {
            perror("recvfrom");
            close(sockfd);
            exit(1);
        }

        line[n] = '\0';
        printf("%s\n", line);

        if(sendto(sockfd, line, n, 0, (struct sockaddr*) &cliaddr, clilen) < 0)
        {
            perror("sendto");
            close(sockfd);
            exit(1);
        }
    }

    close(sockfd);
    return 0;
}
```

### udp_client.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv)
{
    int sockfd, n;
    char sendline[1000], recvline[1000];
    struct sockaddr_in servaddr;

    if(argc != 2)
    {
        printf("Usage: %s <ServerIP>\n", argv[0]);
        exit(1);
    }

    if((sockfd = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        perror("socket");
        exit(1);
    }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(51000);
    
    if(inet_aton(argv[1], &servaddr.sin_addr) == 0)
    {
        printf("Invalid IP address\n");
        close(sockfd);
        exit(1);
    }

    printf("Enter message: ");
    fgets(sendline, 1000, stdin);

    if(sendto(sockfd, sendline, strlen(sendline), 0, 
              (struct sockaddr*) &servaddr, sizeof(servaddr)) < 0)
    {
        perror("sendto");
        close(sockfd);
        exit(1);
    }

    n = recvfrom(sockfd, recvline, 1000, 0, NULL, NULL);
    if(n < 0)
    {
        perror("recvfrom");
        close(sockfd);
        exit(1);
    }

    recvline[n] = '\0';
    printf("Echo from server: %s\n", recvline);

    close(sockfd);
    return 0;
}
```

---

**Task 4**

### udp_client_1024.c
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char **argv)
{
    int sockfd, n;
    char sendline[1024], recvline[1024]; 
    struct sockaddr_in servaddr;

    if(argc != 2)
    {
        printf("Usage: %s <ServerIP>\n", argv[0]);
        exit(1);
    }

    if((sockfd = socket(PF_INET, SOCK_DGRAM, 0)) < 0)
    {
        perror("socket");
        exit(1);
    }

    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(51000);

    if(inet_aton(argv[1], &servaddr.sin_addr) == 0)
    {
        printf("Invalid IP address\n");
        close(sockfd);
        exit(1);
    }

    printf("Enter message (up to 1024 chars): ");
    fgets(sendline, 1024, stdin); 

    if(sendto(sockfd, sendline, strlen(sendline), 0, 
              (struct sockaddr*) &servaddr, sizeof(servaddr)) < 0)
    {
        perror("sendto");
        close(sockfd);
        exit(1);
    }

    n = recvfrom(sockfd, recvline, 1024, 0, NULL, NULL);
    if(n < 0)
    {
        perror("recvfrom");
        close(sockfd);
        exit(1);
    }

    recvline[n] = '\0';
    printf("Echo from server: %s\n", recvline);

    close(sockfd);
    return 0;
}
```