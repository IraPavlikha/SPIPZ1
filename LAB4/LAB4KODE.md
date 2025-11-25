**Task 1**

### write_file.c
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    int fd;
    char string[] = "Hello, world!\n";
    ssize_t size;

    umask(0);

    fd = open("myfile", O_WRONLY | O_CREAT, 0666);
    if (fd < 0) {
        printf("Can't open file!\n");
        exit(-1);
    }

    size = write(fd, string, 14);
    if (size != 14) {
        printf("Can't write all string!\n");
        exit(-1);
    }

    if (close(fd) < 0) {
        printf("Can't close file!\n");
    }

    return 0;
}
```

---

**Task 2**

### read_file.c
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    int fd;
    char buffer[100];
    ssize_t size;

    fd = open("myfile", O_RDONLY);
    if (fd < 0) {
        printf("Can't open file!\n");
        exit(-1);
    }

    size = read(fd, buffer, sizeof(buffer) - 1);
    if (size < 0) {
        printf("Can't read file!\n");
        exit(-1);
    }

    buffer[size] = '\0';
    printf("%s", buffer);

    if (close(fd) < 0) {
        printf("Can't close file!\n");
    }

    return 0;
}
```

---

**Task 3**

### copy_file.c
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    int fd1, fd2;
    char buffer[100];
    ssize_t size_read, size_write;

    fd1 = open("myfile", O_RDONLY);
    if (fd1 < 0) {
        printf("Can't open myfile!\n");
        exit(-1);
    }

    fd2 = open("myfile1", O_WRONLY | O_CREAT, 0666);
    if (fd2 < 0) {
        printf("Can't open myfile1!\n");
        close(fd1);
        exit(-1);
    }

    size_read = read(fd1, buffer, sizeof(buffer));
    if (size_read < 0) {
        printf("Can't read myfile!\n");
        close(fd1);
        close(fd2);
        exit(-1);
    }

    buffer[size_read] = '\0';
    printf("Read from myfile:\n%s", buffer);

    size_write = write(fd2, buffer, size_read);
    if (size_write != size_read) {
        printf("Can't write all data!\n");
    }

    close(fd1);
    close(fd2);

    return 0;
}
```