# Linux-File-IO-Systems-locking
Ex07-Linux File-IO Systems-locking
# AIM:
To Write a C program that illustrates files copying and locking

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux IO Systems locking

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## 1.To Write a C program that illustrates files copying 
```c
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(stderr, "Usage: %s <source_file> <destination_file>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    char block[1024];
    int in, out;
    ssize_t nread, nwritten;

    // Open source file
    in = open(argv[1], O_RDONLY);
    if (in == -1) {
        perror("Error opening source file");
        exit(EXIT_FAILURE);
    }

    // Open destination file
    out = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (out == -1) {
        perror("Error opening destination file");
        close(in);
        exit(EXIT_FAILURE);
    }

    // Copy contents
    while ((nread = read(in, block, sizeof(block))) > 0) {
        nwritten = write(out, block, nread);
        if (nwritten == -1) {
            perror("Error writing to destination file");
            close(in);
            close(out);
            exit(EXIT_FAILURE);
        }
    }

    if (nread == -1) {
        perror("Error reading source file");
    }

    // Close files safely
    if (close(in) == -1)
        perror("Error closing source file");

    if (close(out) == -1)
        perror("Error closing destination file");

    return EXIT_SUCCESS;
}
```

## 2.To Write a C program that illustrates files locking
```c
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/file.h>

void display_lslocks() {
    printf("\nCurrent `lslocks` output:\n");
    fflush(stdout);
    system("lslocks");
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <filename>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    char *file = argv[1];
    int fd;

    printf("Opening %s\n", file);

    // 🔧 FIX: use O_RDWR instead of O_WRONLY
    fd = open(file, O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        perror("Error opening file");
        exit(EXIT_FAILURE);
    }

    // 🔧 FIX: use LOCK_EX first (shared lock on write-only is not ideal)
    if (flock(fd, LOCK_SH) == -1) {
        perror("Error acquiring shared lock");
        close(fd);
        exit(EXIT_FAILURE);
    }
    printf("Acquired shared lock using flock\n");
    display_lslocks();

    sleep(2);

    // Try to upgrade to exclusive lock (non-blocking)
    if (flock(fd, LOCK_EX | LOCK_NB) == -1) {
        perror("Error upgrading to exclusive lock");
        flock(fd, LOCK_UN);
        close(fd);
        exit(EXIT_FAILURE);
    }
    printf("Upgraded to exclusive lock\n");
    display_lslocks();

    sleep(2);

    // Release lock
    if (flock(fd, LOCK_UN) == -1) {
        perror("Error unlocking");
        close(fd);
        exit(EXIT_FAILURE);
    }
    printf("Unlocked\n");
    display_lslocks();

    close(fd);
    return 0;
}
```


## OUTPUT
<img width="845" height="317" alt="image" src="https://github.com/user-attachments/assets/809f00fb-4ffc-4936-ab26-e6d19d37221f" />

<img width="878" height="678" alt="image" src="https://github.com/user-attachments/assets/44ceba7d-e8d6-4d73-ab84-3a8822830957" />


# RESULT:
The programs are executed successfully.
