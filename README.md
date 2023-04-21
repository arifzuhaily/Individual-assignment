#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <signal.h>

void sigint_handler(int signo){
    printf("\nProgram interrupted by user.\n");
    exit(0);
}

int main(){
    signal(SIGINT, sigint_handler);

    int num_child = 3; // change this to create more child processes

    int fd[num_child][2];
    for(int i=0; i<num_child; i++){
        if(pipe(fd[i]) == -1){
            perror("Pipe creation failed");
            exit(1);
        }
    }

    pid_t pid[num_child];
    for(int i=0; i<num_child; i++){
        pid[i] = fork();
        if(pid[i] == -1){
            perror("Forking child process failed");
            exit(1);
        }
        else if(pid[i] == 0){
            close(fd[i][1]); // close write end
            char buffer[1024];
            int nbytes = read(fd[i][0], buffer, sizeof(buffer));
            printf("Child process %d (PID %d) received message: %s", i+1, getpid(), buffer);
            close(fd[i][0]); // close read end
            exit(0);
        }
    }
 for(int i=0; i<num_child; i++){
        close(fd[i][0]); // close read end
    }

    char message[1024];
    printf("Enter a message to send to child processes: ");
    fgets(message, sizeof(message), stdin);

    for(int i=0; i<num_child; i++){
        write(fd[i][1], message, sizeof(message));
        close(fd[i][1]); // close write end
    }

    for(int i=0; i<num_child; i++){
        waitpid(pid[i], NULL, 0);
    }

    return 0;
}



