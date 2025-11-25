**Task 1**

### fork_example1.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main()
{
    pid_t pid, ppid;
    int a = 0;
    fork();
    a = a + 1;
    pid = getpid();
    ppid = getppid();
    printf("My pid = %d, my ppid = %d, result = %d\n", (int)pid, (int)ppid, a);
    return 0;
}
```

---

**Task 2**

### fork_example2.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/wait.h>
#include <stdlib.h>

int main()
{
    pid_t pid;
    pid = fork();
    if(pid < 0) return 1;
    if(pid == 0)
    {
        int sum = 0;
        for(int i = 1; i <= 10; ++i) sum += i;
        printf("Child: sum 1..10 = %d\n", sum);
        _exit(0);
    }
    else
    {
        long prod = 1;
        for(int i = 1; i <= 10; ++i) prod *= i;
        int status;
        waitpid(pid, &status, 0);
        printf("Parent: product 1..10 = %ld\n", prod);
    }
    return 0;
}
```

---

**Task 3**

### vfork_example.c
```c
#define _POSIX_C_SOURCE 200112L
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/wait.h>
#include <stdlib.h>

int main()
{
    pid_t pid;
    int status;
    pid = vfork();
    if(pid < 0) return 1;
    if(pid == 0)
    {
        execlp("echo", "echo", "vfork child: hello from exec", NULL);
        _exit(127);
    }
    else
    {
        waitpid(pid, &status, 0);
        printf("parent: child with vfork finished with status %d\n", WEXITSTATUS(status));
    }
    return 0;
}
```

---

**Task 4**

### fork_struct_example.c
```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#include <sys/wait.h>
#include <stdlib.h>

struct Data { int id; double value; char note[64]; };

int main()
{
    pid_t pid;
    int number = 7;
    char text[64];
    struct Data d;

    strcpy(text, "orig");
    d.id = 1;
    d.value = 3.14;
    strcpy(d.note, "initial");

    pid = fork();
    if(pid < 0) return 1;
    if(pid == 0)
    {
        number += 5;
        strcat(text, "_child");
        d.id = 2;
        d.value *= 2;
        strcpy(d.note, "child_changed");
        printf("child: number=%d text=%s data={%d,%.2f,%s}\n", number, text, d.id, d.value, d.note);
        _exit(0);
    }
    else
    {
        number -= 3;
        strcat(text, "_parent");
        d.id = 99;
        d.value += 1.86;
        strcpy(d.note, "parent_changed");
        int status;
        waitpid(pid, &status, 0);
        printf("parent: number=%d text=%s data={%d,%.2f,%s}\n", number, text, d.id, d.value, d.note);
    }
    return 0;
}
```