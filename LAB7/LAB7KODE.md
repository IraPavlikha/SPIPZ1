**Task 1**

### 06-1a.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    int *array;
    int shmid;
    int new = 1;
    char pathname[] = "06-1a.c";
    key_t key;

    if ((key = ftok(pathname, 0)) < 0) {
        printf("Can't generate key!\n");
        exit(-1);
    }

    if ((shmid = shmget(key, 3 * sizeof(int), 0666 | IPC_CREAT | IPC_EXCL)) < 0) {
        if (errno != EEXIST) {
            printf("Can't create shared memory!\n");
            exit(-1);
        } else {
            if ((shmid = shmget(key, 3 * sizeof(int), 0)) < 0) {
                printf("Can't find shared memory!\n");
                exit(-1);
            }
            new = 0;
        }
    }

    if ((array = (int*) shmat(shmid, NULL, 0)) == (int*)(-1)) {
        printf("Can't attach shared memory!\n");
        exit(-1);
    }

    if (new) {
        array[0] = 1;
        array[1] = 0;
        array[2] = 1;
    } else {
        array[0] += 1;
        array[2] += 1;
    }

    printf("Program 1 was spawn %d times, program 2 - %d times, total - %d times\n", array[0], array[1], array[2]);

    if (shmdt(array) < 0) {
        printf("Can't detach shared memory!\n");
        exit(-1);
    }
    
    return 0;
}
```

### 06-1b.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    int *array;
    int shmid;
    int new = 1;
    char pathname[] = "06-1a.c";
    key_t key;

    if ((key = ftok(pathname, 0)) < 0) {
        printf("Can't generate key!\n");
        exit(-1);
    }

    if ((shmid = shmget(key, 3 * sizeof(int), 0666 | IPC_CREAT | IPC_EXCL)) < 0) {
        if (errno != EEXIST) {
            printf("Can't create shared memory!\n");
            exit(-1);
        } else {
            if ((shmid = shmget(key, 3 * sizeof(int), 0)) < 0) {
                printf("Can't find shared memory!\n");
                exit(-1);
            }
            new = 0;
        }
    }

    if ((array = (int*) shmat(shmid, NULL, 0)) == (int*)(-1)) {
        printf("Can't attach shared memory!\n");
        exit(-1);
    }

    if (new) {
        array[0] = 0;
        array[1] = 1;
        array[2] = 1;
    } else {
        array[1] += 1;
        array[2] += 1;
    }

    printf("Program 1 was spawn %d times, program 2 - %d times, total - %d times\n", array[0], array[1], array[2]);

    if (shmdt(array) < 0) {
        printf("Can't detach shared memory!\n");
        exit(-1);
    }
    
    return 0;
}
```

---

**Task 2**

### writer.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

int main() {
    int shmid;
    key_t key;
    char *shared_mem;
    FILE *f;
    long filesize;

    char pathname[] = "writer.c";

    if ((key = ftok(pathname, 0)) < 0) {
        printf("Can't generate key!\n");
        exit(-1);
    }

    f = fopen("writer.c", "r");
    if (!f) {
        printf("Can't open file!\n");
        exit(-1);
    }
    fseek(f, 0, SEEK_END);
    filesize = ftell(f);
    fseek(f, 0, SEEK_SET);

    if ((shmid = shmget(key, filesize + 1, 0666 | IPC_CREAT)) < 0) {
        printf("Can't create shared memory!\n");
        exit(-1);
    }

    if ((shared_mem = (char *)shmat(shmid, NULL, 0)) == (char *)(-1)) {
        printf("Can't attach shared memory!\n");
        exit(-1);
    }

    fread(shared_mem, 1, filesize, f);
    shared_mem[filesize] = '\0';

    fclose(f);
    printf("Text copied to shared memory.\n");

    if (shmdt(shared_mem) < 0) {
        printf("Can't detach shared memory!\n");
        exit(-1);
    }

    return 0;
}
```

### reader.c
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    int shmid;
    key_t key;
    char *shared_mem;
    char pathname[] = "writer.c";

    if ((key = ftok(pathname, 0)) < 0) {
        printf("Can't generate key!\n");
        exit(-1);
    }

    if ((shmid = shmget(key, 0, 0666)) < 0) {
        printf("Can't find shared memory!\n");
        exit(-1);
    }

    if ((shared_mem = (char *)shmat(shmid, NULL, 0)) == (char *)(-1)) {
        printf("Can't attach shared memory!\n");
        exit(-1);
    }

    printf("Text from shared memory:\n%s\n", shared_mem);

    if (shmdt(shared_mem) < 0) {
        printf("Can't detach shared memory!\n");
        exit(-1);
    }

    if (shmctl(shmid, IPC_RMID, NULL) < 0) {
        printf("Can't remove shared memory!\n");
        exit(-1);
    }

    return 0;
}
```