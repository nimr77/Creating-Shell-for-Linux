# Creating a custom  shell for Linux

## How it works ?
The first thing to keep in mind is, this is a  programe, so it will loop for the infinite.
``` while(1) ```, 
then it will ask the user for input, the input will be prased to get its contain, like pips or spaces,
then we will excute, the first command then the pip, and so on.

### pips 
They will take the output of the first command and send it to the next one, by using the dup2 system call.
####
and in the end we will clean the memory and keep the programe working
## The Code 
We will import the following libs:
```c
#include<stdio.h> 
#include<string.h> 
#include<stdlib.h> 
#include<unistd.h> 
#include<sys/types.h> 
#include<sys/wait.h> 
#include<readline/readline.h> 
#include<readline/history.h> 
```
Please note that you can launch the Script to install the missing libs.

#### The UI
This to clean the screen
```c
/* Clearing the shell using escape sequences, The screen getting cleared with cursor 
 positioned at the beginning of the screen. */
#define clear() printf("\033[H\033[J") 
```
and this for welcoming the user 
```c
// Greeting shell during startup, It's vary cool to be added  
void init_shell() 
{ 
	clear(); 
	printf("\n\n\n\n******************"
		"************************"); 
	printf("\n\n\n\t**** Nimr's ****"); 
	printf("\n\n\t-USE FOR DEVELOPMENT-"); 
	printf("\n\n\n\n*******************"
		"***********************"); 
	char* username = getenv("USER"); 
	printf("\n\n\nUSER is: @%s", username); 
	printf("\n"); 
	sleep(1); 
	clear(); 
} 
```
after init the shell and greeting the user, we will print some info
#### The current dir
```c
		// Function to print Current Directory. 
void printDir() 
{ 
	char cwd[1024]; 
	// The get CWD is for getting the current dirctory 
	getcwd(cwd, sizeof(cwd)); 
	printf("\nDir: %s", cwd); 
} 
```


#### The input
Here we will take the input of the user and add them to the history in the linux system
The ``` readline ``` will  /* Readline functions. */
 Read a line of input.  Prompt with PROMPT.  A NULL PROMPT means none. 
```c
// Function to take input,return's 0 if all ok, 1 if error 
int takeInput(char* str) 
{ 
	char* buf; 
	// Here we will read the input of the user and by that we will see if its all ok, then we will go for 
	//the next proccesing 
	buf = readline("\n>>> "); 
	if (strlen(buf) != 0) { 
		// this lib will help us to add the history into the unix history
		add_history(buf); 
		strcpy(str, buf); 
		return 0; 
	} else { 
		return 1; 
	} 
}
```

#### Excuting the command 
Now after the input we will handle the command excuting, by calling 
```c
int processString(char* str, char** parsed, char** parsedpipe) 
{ 

	char* strpiped[2]; 
	int piped = 0; 

	piped = parsePipe(str, strpiped); 

	if (piped) { 
		parseSpace(strpiped[0], parsed); 
		parseSpace(strpiped[1], parsedpipe); 

	} else { 

		parseSpace(str, parsed); 
	} 

	if (ownCmdHandler(parsed)) 
		return 0; 
	else
		return 1 + piped; 
} 
```
This will take the input and the prased argus, the piped argues, 
and will prase them into parsePipe:
##### parsePipe and parseSpace
this will take the input and gives us the argus as commands and space with pips
the '|'
```c
// function for finding pipe 
int parsePipe(char* str, char** strpiped) 
{ 
	int i; 
	for (i = 0; i < 2; i++) { 
		strpiped[i] = strsep(&str, "|"); 
		if (strpiped[i] == NULL) 
			break; 
	} 

	if (strpiped[1] == NULL) 
		return 0; // returns zero if no pipe is found. 
	else { 
		return 1; 
	} 
} 
```
the ' '
```c
// function for parsing command words, that are with space 
void parseSpace(char* str, char** parsed) 
{ 
	int i; 

	for (i = 0; i < MAXPIPES; i++) { 
		parsed[i] = strsep(&str, " "); 

		if (parsed[i] == NULL) 
			break; 
		if (strlen(parsed[i]) == 0) 
			i--; 
	} 
} 
```

after proccesing now we will excute the line, by chacking the return of  ```processString ```
	```	// execflag returns zero if there is no command 
		// or it is a builtin command, 
	  // 1 if it is a simple command 
	// 2 if it is including a pipe. ```

	Now we got two Function to handle the excuting one piped the other not.
##### execArgs
this will handle the not piped one
```c
// Function where the system command is executed 
void execArgs(char** parsed) 
{ 
	// Forking a child 
	pid_t pid = fork(); 

	if (pid == -1) { 
		printf("\nFailed forking child.."); 
		return; 
	} else if (pid == 0) { 
		// Here we will excute the command and check if the command is a command !
		if (execvp(parsed[0], parsed) < 0) { 
			printf("\nCould not execute command.."); 
		} 
		exit(0); 
	} else { 
		// waiting for child to terminate 
		wait(NULL); 
		return; 
	} 
} 
```
##### execArgsPiped
This is the piped one
```c
// Function where the piped system commands is executed 
void execArgsPiped(char** parsed, char** parsedpipe) 
{ 
	// 0 is read end, 1 is write end 
	int pipefd[2]; 
	pid_t p1, p2; 

	if (pipe(pipefd) < 0) { 
		printf("\nPipe could not be initialized"); 
		return; 
	} 
	p1 = fork(); 
	if (p1 < 0) { 
		printf("\nCould not fork"); 
		return; 
	} 

	if (p1 == 0) { 
		// Child 1 executing.. 
		// It only needs to write at the write end 
		close(pipefd[0]); 
		dup2(pipefd[1], STDOUT_FILENO); 
		close(pipefd[1]); 

		if (execvp(parsed[0], parsed) < 0) { 
			printf("\nCould not execute command 1.."); 
			exit(0); 
		} 
	} else { 
		// Parent executing 
		p2 = fork(); 

		if (p2 < 0) { 
			printf("\nCould not fork"); 
			return; 
		} 

		// Child 2 executing.. 
		// It only needs to read at the read end 
		if (p2 == 0) { 
			close(pipefd[1]); 
			dup2(pipefd[0], STDIN_FILENO); 
			close(pipefd[0]); 
			if (execvp(parsedpipe[0], parsedpipe) < 0) { 
				printf("\nCould not execute command 2.."); 
				exit(0); 
			} 
		} else { 
			// parent executing, waiting for two children 
			wait(NULL); 
			wait(NULL); 
		} 
	} 
}
```

Now what about the custom commands ?
#### The costum command

This function contains the list of costum command, you can edit it
and add your commands.
```c
// Function to execute builtin commands=> {}
int ownCmdHandler(char** parsed) 
{ 
	int NoOfOwnCmds = 5, i, switchOwnArg = 0; 
	char* ListOfOwnCmds[NoOfOwnCmds]; 
	char* username; 
	///List of the custom commands 
	ListOfOwnCmds[0] = "exit"; 
	ListOfOwnCmds[1] = "cd"; 
	ListOfOwnCmds[2] = "help"; 
	ListOfOwnCmds[3] = "hello"; 
	ListOfOwnCmds[4] = "whoDevMe"; 

	for (i = 0; i < NoOfOwnCmds; i++) { 
		if (strcmp(parsed[0], ListOfOwnCmds[i]) == 0) { 
			switchOwnArg = i + 1; 
			break; 
		} 
	} 

	switch (switchOwnArg) { 
	case 1: 
		printf("\nGoodbye\n"); 
		exit(0); 
	case 2: 
		chdir(parsed[1]); 
		return 1; 
	case 3: 
		openHelp(); 
		return 1; 
	case 4: 
		username = getenv("USER"); 
		printf("\nHello %s.\nMind that this is "
			"not a place to play around."
			"\nUse help to know more.. or contact me at my email nimr.h.sawafta@gmail.com \n", 
			username); 
		return 1; 
	case 5:
		printf("This shell been developed by Nimr Sawafta");
		return 1;
	default: 
		break; 
	} 

	return 0; 
} 
```

