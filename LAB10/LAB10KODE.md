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

