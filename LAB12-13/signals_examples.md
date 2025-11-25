**Task 1**

### sender.c
```c
#include <signal.h>
#include <unistd.h>

int main(void) {
    signal(SIGINT, SIG_IGN);
    while (1) {
        pause();
    }
    return 0;
}
```

---

**Task 2**

### handler.c
```c
#include <signal.h>
#include <stdio.h>

void my_handler(int nsig) {
    printf("Receive signal %d, CTRL-C pressed.\n", nsig);
}

int main(void) {
    (void) signal(SIGINT, my_handler);
    while (1);
    return 0;
}
```

---

**Task 3**

### ignore_signals.c
```c
#include <signal.h>
#include <unistd.h>

int main(void) {
    signal(SIGINT, SIG_IGN);
    signal(SIGQUIT, SIG_IGN);
    while (1) {
        pause();
    }
    return 0;
}
```

---

**Task 4**

### multiple_handler.c
```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void my_handler(int nsig) {
    if(nsig == SIGINT)
        printf("Received SIGINT (CTRL-C)\n");
    else if(nsig == SIGQUIT)
        printf("Received SIGQUIT (CTRL-\\)\n");
    fflush(stdout);
}

int main(void) {
    signal(SIGINT, my_handler);
    signal(SIGQUIT, my_handler);

    while(1) {
        pause();
    }

    return 0;
}
```

---

**Task 5**

### restore_handler.c
```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

int i = 0;
void (*p)(int);

void my_handler(int nsig) {
    printf("Receive signal %d, CTRL-C pressed\n", nsig);
    fflush(stdout);
    i++;
    if(i == 5)
        signal(SIGINT, p);
}

int main(void) {
    p = signal(SIGINT, my_handler);
    while(1)
        pause();
    return 0;
}
```

---

**Task 6**

### sender.c
```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

void send_number(pid_t pid, int number) {
    for(int i = 31; i >= 0; i--) {
        int bit = (number >> i) & 1;
        if(bit == 0)
            kill(pid, SIGUSR1);
        else
            kill(pid, SIGUSR2);
        usleep(10000); // маленька затримка між сигналами
    }
}

int main(int argc, char *argv[]) {
    if(argc != 3) {
        printf("Usage: %s <receiver_pid> <number>\n", argv[0]);
        return 1;
    }

    pid_t pid = atoi(argv[1]);
    int number = atoi(argv[2]);

    send_number(pid, number);
    return 0;
}
```

### receiver.c
```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

int received = 0;
int bit_count = 0;

void handler(int sig) {
    received <<= 1;
    if(sig == SIGUSR2) {
        received |= 1;
    }
    bit_count++;

    if(bit_count == 32) {
        printf("Received number: %d\n", received);
        fflush(stdout);
        received = 0;
        bit_count = 0;
    }
}

int main(void) {
    printf("Receiver PID: %d\n", getpid());
    fflush(stdout);

    signal(SIGUSR1, handler);
    signal(SIGUSR2, handler);

    while(1) pause();
    return 0;
}
```

---

**Task 7**

### sigchld_handler.c
```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

void my_handler(int nsig) {
    int status;
    pid_t pid;

    while ((pid = waitpid(-1, &status, WNOHANG)) > 0) {
        if ((status & 0xff) == 0) {
            printf("Process %d exited with status %d\n", pid, status >> 8);
        } else {
            printf("Process %d killed by signal %d %s\n", pid, status & 0x7f,
                   (status & 0x80) ? "with core file." : "without core-file.");
        }
    }
}

int main(void) {
    pid_t pid;

    signal(SIGCHLD, my_handler);

    if ((pid = fork()) < 0) {
        printf("Can't fork child1!\n");
        exit(1);
    } else if (pid == 0) {
        exit(200);
    }

    if ((pid = fork()) < 0) {
        printf("Can't fork child2!\n");
        exit(1);
    } else if (pid == 0) {
        while (1);
    }

    while (1) pause();

    return 0;
}
```

---

**Task 8**

У цій програмі батьківський процес створює 6 дочірніх процесів, кожен з яких одразу завершується викликом `exit(200+i)`. Батьківський процес встановлює обробник сигналу SIGCHLD, який має друкувати повідомлення про завершення процесу через `waitpid()`.  

Проблема: стандартна функція `signal()` для SIGCHLD не гарантує, що сигнал буде зберігатися, якщо декілька дітей завершуються майже одночасно. Якщо кілька дочірніх процесів завершуються дуже швидко, сигнал може бути “втрачений”, і обробник спрацює лише один раз.  

Висновок: очікуємо 6 повідомлень, реально отримаємо менше, часто лише одне. Для коректного оброблення всіх дітей потрібно використовувати `sigaction()` з `SA_RESTART` та цикл `waitpid()` всередині обробника.

---

**Task 9**

### wait_example.c
```c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    pid_t pid;
    int status;
    int i;

    for (i = 0; i < 5; i++) {
        pid = fork();
        if (pid < 0) {
            printf("Can't fork child %d!\n", i+1);
            exit(1);
        } else if (pid == 0) {
            exit(200 + i);
        }
    }

    for (i = 0; i < 5; i++) {
        pid = wait(&status);
        if ((status & 0xff) == 0) {
            printf("Process %d exited with status %d\n", pid, status >> 8);
        } else {
            printf("Process %d killed by signal %d %s\n", pid, status & 0x7f,
                   (status & 0x80) ? "with core file." : "without core-file.");
        }
    }

    return 0;
}
```

