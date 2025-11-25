**Task 1**

### get_uid.c
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    uid_t i;
    i = getuid();
    printf("Отриманий ідентифікатор користувача (UID): %d\n", (int) i);
    return 0;
}
```

---

**Task 2**

### get_gid.c
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    gid_t g;
    g = getgid();
    printf("Отриманий ідентифікатор групи користувача (GID): %d\n", (int) g);
    return 0;
}
```

---

**Task 3**

### get_process_info.c
```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    uid_t uid = getuid();     
    gid_t gid = getgid();     
    pid_t pid = getpid();      
    pid_t ppid = getppid();   

    printf("UID користувача: %d\n", (int) uid);
    printf("GID групи: %d\n", (int) gid);
    printf("PID процесу: %d\n", (int) pid);
    printf("PPID (батьківський PID): %d\n", (int) ppid);

    return 0;
}
```

---

**Пояснення для питань самоконтролю**

- `sys/types.h` – визначає системні типи даних (`uid_t`, `gid_t`, `pid_t` і т.д.).
- `unistd.h` – містить оголошення системних викликів UNIX (`getuid`, `getgid`, `getpid`, `fork` тощо).
- `getuid()` – повертає ідентифікатор користувача, від імені якого виконується процес.
- `getgid()` – повертає GID (ідентифікатор групи користувача).
- `getpid()` – повертає PID поточного процесу.
- `getppid()` – повертає PID батьківського процесу.
- `uid_t` і `gid_t` – зазвичай синоніми `unsigned int` (але залежить від системи).
- `pid_t` і `ppid_t` – синоніми типу `int`.
- Операція приведення типів – це явна вказівка компілятору, як інтерпретувати дані іншого типу. Приклад:
```c
uid_t uid = getuid();
printf("%d", (int) uid);  // приведення до int
```
- ESC-символи в `printf` – це спеціальні керуючі послідовності:
  - `\n` – новий рядок
  - `\t` – табуляція
  - `\\` – вивести зворотний слеш