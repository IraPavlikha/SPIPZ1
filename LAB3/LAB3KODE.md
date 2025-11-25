**Task 1**

### execle_example.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[], char *envp[]) {
    (void) execle("/bin/cat", "cat", "03-2.c", NULL, envp);
    printf("Error on program start\n");
    exit(-1);
}
```

---

**Task 2**

### execv_example.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    char *args[] = {"cat", "03-2.c", NULL};
    execv("/bin/cat", args);
    printf("Error on program start\n");
    exit(-1);
}
```

---

**Task 3**

### fork_env_example.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[], char *envp[]) {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child process:\n");
        for (int i = 0; i < argc; i++) printf("argv[%d] = %s\n", i, argv[i]);
        for (int i = 0; envp[i] != NULL; i++) printf("envp[%d] = %s\n", i, envp[i]);
    } else if (pid > 0) {
        wait(NULL);
        printf("Parent process finished\n");
    } else {
        printf("Fork failed\n");
        exit(-1);
    }
    return 0;
}
```

---

**Task 4**

### execve_example.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[], char *envp[]) {
    char *args[] = {"cat", "03-2.c", NULL};
    execve("/bin/cat", args, envp);
    printf("Error on program start\n");
    exit(-1);
}
```

---

**Task 5**

### fork_execle_example.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[], char *envp[]) {
    pid_t pid = fork();
    if (pid == 0) {
        execle("/bin/ls", "ls", "-l", NULL, envp);
        printf("Error starting child process\n");
        exit(-1);
    } else if (pid > 0) {
        printf("Parent process running\n");
        wait(NULL);
        printf("Parent process finished\n");
    } else {
        printf("Fork failed\n");
        exit(-1);
    }
    return 0;
}
```

---

**Notes**

Різниця між execle, execv та execve:

1️⃣ **execle(path, arg0, arg1, …, NULL, envp)**
- Аргументи передаються списком.
- Останній аргумент — envp: масив рядків з середовищем процесу.
- Можна змінювати середовище для нового процесу.

2️⃣ **execv(path, argv[])**
- Аргументи передаються масивом рядків (argv[]).
- Середовище процесу не передається, використовується поточне.

4️⃣ **execve(path, argv[], envp[])**
- Аргументи передаються масивом рядків.
- Середовище процесу передається через envp[], можна його змінити.
- Поєднує зручність argv[] з можливістю задавати власне середовище.

