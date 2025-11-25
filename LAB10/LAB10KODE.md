**Task 1**

### 08-1a.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

#define LAST_MESSAGE 255

int main() {
    int msgid;
    char pathname[] = "08-1a.c";
    key_t key;
    int i, len;

    struct mymsgbuf {
        long mtype;
        char mtext[81];
    } mybuf;

    if ((key = ftok(pathname, 0)) < 0) {
        printf("Can't generate key!\n");
        exit(-1);
    }

    if ((msgid = msgget(key, 0666 | IPC_CREAT)) < 0) {
        printf("Can't get msgid!\n");
        exit(-1);
    }

    for (i = 1; i <= 5; i++) {
        mybuf.mtype = 1;
        strcpy(mybuf.mtext, "This is text message.");
        len = strlen(mybuf.mtext) + 1;
        if (msgsnd(msgid, (struct msgbuf*)&mybuf, len, 0) < 0) {
            printf("Can't send message to queue!\n");
            msgctl(msgid, IPC_RMID, NULL);
            exit(-1);
        }
    }

    mybuf.mtype = LAST_MESSAGE;
    len = 0;
    if (msgsnd(msgid, (struct msgbuf*)&mybuf, len, 0) < 0) {
        printf("Can't send message to queue!\n");
        msgctl(msgid, IPC_RMID, NULL);
        exit(-1);
    }

    return 0;
}
```

### 08-1b.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <stdio.h>
#include <stdlib.h>

#define LAST_MESSAGE 255

int main() {
    int msgid;
    char pathname[] = "08-1a.c";
    key_t key;
    int len, maxlen = 81;

    struct mymsgbuf {
        long mtype;
        char mtext[81];
    } mybuf;

    if ((key = ftok(pathname, 0)) < 0) {
        printf("Can't generate key!\n");
        exit(-1);
    }

    if ((msgid = msgget(key, 0666 | IPC_CREAT)) < 0) {
        printf("Can't get msgid!\n");
        exit(-1);
    }

    while (1) {
        if ((len = msgrcv(msgid, (struct msgbuf*)&mybuf, maxlen, 0, 0)) < 0) {
            printf("Can't receive message from queue!\n");
            exit(-1);
        }

        if (mybuf.mtype == LAST_MESSAGE) {
            msgctl(msgid, IPC_RMID, NULL);
            exit(0);
        }

        printf("message type = %ld, info = %s\n", mybuf.mtype, mybuf.mtext);
    }

    return 0;
}
```

---

**Task 2**

### sender.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <stdio.h>
#include <stdlib.h>

#define LAST_MESSAGE 255

struct mymsgbuf {
    long mtype;
    struct {
        short sinfo;
        float finfo;
    } info;
};

int main() {
    int msgid;
    key_t key;
    struct mymsgbuf mybuf;

    key = ftok(".", 'A');
    if (key < 0) {
        perror("ftok");
        exit(1);
    }

    msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid < 0) {
        perror("msgget");
        exit(1);
    }

    for (short i = 1; i <= 5; i++) {
        mybuf.mtype = 1;
        mybuf.info.sinfo = i;
        mybuf.info.finfo = i * 1.5f;
        if (msgsnd(msgid, &mybuf, sizeof(mybuf.info), 0) < 0) {
            perror("msgsnd");
            exit(1);
        }
    }

    mybuf.mtype = LAST_MESSAGE;
    if (msgsnd(msgid, &mybuf, 0, 0) < 0) {
        perror("msgsnd last");
        exit(1);
    }

    return 0;
}
```

### receiver.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <stdio.h>
#include <stdlib.h>

#define LAST_MESSAGE 255

struct mymsgbuf {
    long mtype;
    struct {
        short sinfo;
        float finfo;
    } info;
};

int main() {
    int msgid;
    key_t key;
    struct mymsgbuf mybuf;
    int len;

    key = ftok(".", 'A');
    if (key < 0) {
        perror("ftok");
        exit(1);
    }

    msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid < 0) {
        perror("msgget");
        exit(1);
    }

    while (1) {
        len = msgrcv(msgid, &mybuf, sizeof(mybuf.info), 0, 0);
        if (len < 0) {
            perror("msgrcv");
            exit(1);
        }

        if (mybuf.mtype == LAST_MESSAGE) {
            msgctl(msgid, IPC_RMID, NULL);
            break;
        }

        printf("Received: sinfo=%d, finfo=%.2f\n", mybuf.info.sinfo, mybuf.info.finfo);
    }

    return 0;
}
```

---

**Task 3**

### process1.c
```c
#include "common.h"
#include <unistd.h>

int main() {
    key_t key = ftok(".", QUEUE_KEY_ID);
    if (key < 0) { perror("ftok"); exit(1); }

    int msgid = msgget(key, IPC_CREAT | QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct mymsgbuf msg;
    for (int i = 1; i <= 5; i++) {
        msg.mtype = 1;
        msg.info.value = i * 10;
        if (msgsnd(msgid, &msg, sizeof(msg.info), 0) < 0) {
            perror("msgsnd"); exit(1);
        }
        printf("Process1 sent: %d\n", msg.info.value);

        if (msgrcv(msgid, &msg, sizeof(msg.info), 2, 0) < 0) {
            perror("msgrcv"); exit(1);
        }
        printf("Process1 received: %d\n", msg.info.value);
    }

    return 0;
}
```

### process2.c
```c
#include "common.h"

int main() {
    key_t key = ftok(".", QUEUE_KEY_ID);
    if (key < 0) { perror("ftok"); exit(1); }

    int msgid = msgget(key, IPC_CREAT | QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct mymsgbuf msg;
    for (int i = 1; i <= 5; i++) {
        if (msgrcv(msgid, &msg, sizeof(msg.info), 1, 0) < 0) {
            perror("msgrcv"); exit(1);
        }
        printf("Process2 received: %d\n", msg.info.value);

        msg.mtype = 2;
        msg.info.value = msg.info.value + 1;
        if (msgsnd(msgid, &msg, sizeof(msg.info), 0) < 0) {
            perror("msgsnd"); exit(1);
        }
        printf("Process2 sent: %d\n", msg.info.value);
    }

    return 0;
}
```

### common.h
```c
#ifndef COMMON_H
#define COMMON_H

#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <stdio.h>
#include <stdlib.h>

struct mymsgbuf {
    long mtype;
    struct {
        int value;
    } info;
};

#define QUEUE_PERMISSIONS 0666
#define QUEUE_KEY_ID 'Q'

#endif
```

---

**Task 4**

### client.c
```c
#include "common.h"

int main(int argc, char *argv[]) {
    if (argc < 3) { printf("Usage: %s <client_id> <value>\n", argv[0]); return 1; }

    int client_id = atoi(argv[1]);
    int value = atoi(argv[2]);

    int msgid = msgget(SERVER_QUEUE_KEY, QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct request_msg req;
    req.mtype = 1;
    req.client_id = client_id;
    req.value = value;

    if (msgsnd(msgid, &req, sizeof(req) - sizeof(long), 0) < 0) {
        perror("msgsnd"); exit(1);
    }

    struct response_msg resp;
    if (msgrcv(msgid, &resp, sizeof(resp) - sizeof(long), client_id, 0) < 0) {
        perror("msgrcv"); exit(1);
    }

    printf("Client %d received: %d\n", client_id, resp.result);
    return 0;
}
```

### server.c
```c
#include "common.h"

int main() {
    int msgid = msgget(SERVER_QUEUE_KEY, IPC_CREAT | QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct request_msg req;
    struct response_msg resp;

    while (1) {
        if (msgrcv(msgid, &req, sizeof(req) - sizeof(long), 1, 0) < 0) {
            perror("msgrcv"); exit(1);
        }

        printf("Server received from client %d: %d\n", req.client_id, req.value);

        resp.mtype = req.client_id;
        resp.result = req.value * 2;

        if (msgsnd(msgid, &resp, sizeof(resp) - sizeof(long), 0) < 0) {
            perror("msgsnd"); exit(1);
        }

        printf("Server sent to client %d: %d\n", req.client_id, resp.result);
    }

    return 0;
}
```

### common.h
```c
#ifndef COMMON_H
#define COMMON_H

#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define SERVER_QUEUE_KEY 1234
#define QUEUE_PERMISSIONS 0666

struct request_msg {
    long mtype;
    int client_id;
    int value;
};

struct response_msg {
    long mtype;
    int result;
};

#endif
```

---

**Task 5**

### common.h
```c
#ifndef COMMON_H
#define COMMON_H

#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define SEM_QUEUE_KEY 5678
#define QUEUE_PERMISSIONS 0666

struct sem_msg {
    long mtype;
};

#endif
```

### semaphore_init.c
```c
#include "common.h"

int main() {
    int msgid = msgget(SEM_QUEUE_KEY, IPC_CREAT | QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct sem_msg msg;
    msg.mtype = 1;

    int initial_value = 3;

    for (int i = 0; i < initial_value; i++) {
        if (msgsnd(msgid, &msg, 0, 0) < 0) {
            perror("msgsnd"); exit(1);
        }
    }

    printf("Semaphore initialized with value %d\n", initial_value);
    return 0;
}
```

### semaphore_wait.c
```c
#include "common.h"

int main() {
    int msgid = msgget(SEM_QUEUE_KEY, QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct sem_msg msg;

    if (msgrcv(msgid, &msg, 0, 0, 0) < 0) {
        perror("msgrcv"); exit(1);
    }

    printf("Semaphore decremented (P)\n");
    return 0;
}
```

### semaphore_signal.c
```c
#include "common.h"

int main() {
    int msgid = msgget(SEM_QUEUE_KEY, QUEUE_PERMISSIONS);
    if (msgid < 0) { perror("msgget"); exit(1); }

    struct sem_msg msg;
    msg.mtype = 1;

    if (msgsnd(msgid, &msg, 0, 0) < 0) {
        perror("msgsnd"); exit(1);
    }

    printf("Semaphore incremented (V)\n");
    return 0;
}
```

