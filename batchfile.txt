#include <stdio.h>
#include<string.h>
#include <unistd.h>
#include<stdlib.h>
#include <dirent.h>
 #include <sys/stat.h>
 #include <fcntl.h>

/* Questions to ask. 

Do our tokens and input in the command line have to by dynammically allocated?

Do other white spaces count as delimeters?

Ask about functionality of piping. 

Confirm that builtins don't have to be forked. 


FEB 26th 2017

Can I assume the client gives a valid name for the batch file?

After entering invalid input, exit commands works after I type exit the amount of times + 1 i entered 
the invalid command. RESOLVED

NOTES

If amperstand then we dont wait for child process to complete. Does this apply to builtins? (No) Does amperstand have to
be at the end? (Yea)
Can fork builtins that will use IO. 
Still have to complete help builtin, and make file for help.

For ampersand for pipes and redir, can remove it after checking LUL. 


*/ 



/*
Info. 

Piping should be in this format arg | (or > or < or >> or <<) arg
Need to make a batch file that inputs commands into the shell


*/

/*

Objective: Design a C shell that takes command input from user input or a batch
file. The shell will incorporate 9 built in commands { "exit", "clear", "help",
"dir", "pause", "shellPath", "environ", "echo", "cd"} via function calls. The user
can also pass commands found in the "/bin/" directory. The shell should also be 
able to perform input and output redirection with files, and commands using the symbols: "<",">",">>". Moreover,
the shell should be able to perform piping functions using the symbol "|". Lastly,
if the user wants to run a command in the background they must enter an amperstan ("&")
at the end of their input. 



*/


// This value determines the max amount of characters the user can input into the
// command line
#define COMMAND_LINE_BUFFER 256
// this value determines the max amount of characters for a token ( the token is 
// created when the user input is parsed).
#define TOKEN_BUFFER_CMDLINE 64 


// This is a set of strings that represents the built in commands for the shell.
// Later on we will determine if the user wants to invoke a built in command, by 
// comparing these strings to the parsed user input.
char builtins[9][10] = { "exit", "clear", "help", "dir", "pause", "shellPath", "environ", "echo", "cd"};

// This set of strings represents if the user would like to invoke I/O redirection.
char IOstrings[3][3] = { "<",">",">>"};




// these are global variables that will represent of I/O and piping characters. 
// They are intialized to -1, to represent the lack of I/O and piping characters.
int indexOfIO = -1; int indexOfIO_Two = -1; int indexOfPipe = -1;
int sizeOfTotalArgs;

// FUNCTION DECLARATIONS

// The following are the function delcarations for the built in commands of the 
// shell. More information on these functions is at the bottom, where their actual
// code for functionality resides.
void my_exit();
void my_clear();
void my_dir();
void my_pause();
void my_echo(char **stringToBeEchod);
void my_environ();
void my_ShellPath();
void my_cd(char *path);
void my_help();

// The following function delcarations are for functions I will use to run my 
// shell.

void reDirChannels(char *file, int type_of_ReDir_Request);
void reDirection(char **arguments, char *shellPath,int IOreturnValue); 
char **parseInput(char *input);
int  check_Builtins(char **arguments);
int check_IO(char **arguments);
int runInBackgroundRequest(char **arguments);
int checkPipe(char **arguments);
char **splitStringArr(char **arguments,int beginning, int end);
void runPipe(char **arguments, char *pathOfShell);
void executeCommands(char **arguments, char *pathOfShell, int ampReturnValue);


// The shell can run via user input, or input from a batch file.

int main(int argc, char *argv[])  {
	
	
	
	// When we start the shell we want a "handle"  which tells you where you 
	// started in directory path. Also you want the directory path of the 
	// executable file of your shell. 

	// Using getcwd function to store path to working directory
	char pathToCurrentDirectory[100000];
	getcwd(pathToCurrentDirectory, sizeof(pathToCurrentDirectory));
	
	// This will be used as an argument for the shellPath function.
	char pathOfShell[100000];
	getcwd(pathOfShell, sizeof(pathOfShell));
	
	// First we want to check if the client wants to use input from a batchfile
	if (argc == 2) {
		
	// We check if the user has provided a valid name for the batch file	
	if ((strcmp(argv[1],"batchfile.txt") == 0) && argv[1] != NULL) {
		
		// Check for error in opening the batch file
		FILE* batch = fopen("batchfile.txt", "r"); //Open file for reading
		if(batch == NULL){
			printf("Error. Could not open file.\n");
		}
		
		// commandLineString will hold input from the batch file, so we allocate
		// memory for commandLineString to hold the input. 
		// fgets will read from the batch file line by line. 
		// When the batch file has nothing to read this loop will stop, or when
		// the my_exit() function is invoked.
		char *commandLineString = malloc(sizeof(char) * COMMAND_LINE_BUFFER);
		while(fgets(commandLineString,COMMAND_LINE_BUFFER+1,batch)) {
	
			
	getcwd(pathToCurrentDirectory, sizeof(pathToCurrentDirectory));
	
	//This output shows our current working directory
	printf("%s >>> ",pathToCurrentDirectory);
	
	
	
	// Parse user input, and check if i contains a "&", I/O characters, or pipe.
	
	char **arguments = malloc(sizeof(commandLineString)*20);;
	 int returnValue;
	 int IOreturnValue;
	 int ampReturnValue;

	
	
	
	arguments = parseInput(commandLineString);	
	//returnValue = check_Builtins(arguments);
	IOreturnValue = check_IO(arguments);
	ampReturnValue = runInBackgroundRequest(arguments);
	int pipeFlag = checkPipe(arguments);
	
	
	
	// If the pipe flag or IO flag return false values, then we don't have to further
	// split the arguments.
	
	// If either flags return true values, then we split. 
	
	
	// In this case the user doesn't want to perform redirection and piping.
	// So we work with all the arguments as a whole.
	if (IOreturnValue == 0 && pipeFlag == 0) {
		
		executeCommands(arguments,pathOfShell,ampReturnValue);
		
	} else { //In this case the user would like to perform operations with IO redirection. 
		
		// We must split the arguments into pieces
		
		
		
		if(IOreturnValue == 0 && pipeFlag == 1 ) {
	
			runPipe(arguments,pathOfShell);
			
		} else if (IOreturnValue > 0 && pipeFlag == 0) {
			
			reDirection(arguments,pathOfShell,IOreturnValue);
			
		}
		
		
	} 

}


	
	
	

	// the function calls above change the values of the global variables, so
	// we must reset their values to -1. 
	indexOfIO = -1; indexOfIO_Two = -1; indexOfPipe = -1;
	
	// This will free the previously allocated memory/
	free(commandLineString);
	
	// allocate memory for next command.
	commandLineString = malloc(sizeof(char) * COMMAND_LINE_BUFFER);
				
		} else {printf("Invalid argument passed\n");} // This will output if an invalid batch file name was entered.
	}
		
		
	 else { //USER INPUT CASE
	
	
// We want to run an "infinite" loop. So the shell can keep running, to take user input. 
// This loop will be exited via an exit function. 
while(1) {
	
	getcwd(pathToCurrentDirectory, sizeof(pathToCurrentDirectory));
	
	
	printf("%s >>> ",pathToCurrentDirectory);
	
	// This is a flag to check for empty input 
	int flagForEmptyInput = 0;
	
	// This character array will hold the input by the user.
	char *commandLineString = malloc(sizeof(char) * COMMAND_LINE_BUFFER);
	
	// These two lines will check if the user entered something, using the return
	// value of gets(). If gets() returns NULL, then the user has entered nothing.
	 char *returnValueOf_gets = gets(commandLineString);
	
	 if (*returnValueOf_gets == '\0' ) {flagForEmptyInput = 1; }
	
	// Parse user input
	
	char **arguments = malloc(sizeof(commandLineString)*20);;
	 int returnValue;
	 int IOreturnValue;
	 int ampReturnValue;
	;
	
	
	 // if the flag is set to zero, that means that there is input
	 // and therefore we will proceed to parse input. 
	if (flagForEmptyInput == 0) {
	arguments = parseInput(commandLineString);	
	IOreturnValue = check_IO(arguments);
	ampReturnValue = runInBackgroundRequest(arguments);
	int pipeFlag = checkPipe(arguments);
	
	
	
	// If the pipe flag or IO flag return false values, then we don't have to further
	// split the arguments.
	
	// If either flags return true values, then we split. 
	
	
	// In this case the user doesn't want to perform redirection and piping.
	// So we work with all the arguments as a whole.
	if (IOreturnValue == 0 && pipeFlag == 0) {
		
		executeCommands(arguments,pathOfShell,ampReturnValue);
		
	} else { //In this case the user would like to perform operations with IO redirection. 
		
		
		
		
		// This is the case where the user wants to use a pipe
		if(IOreturnValue == 0 && pipeFlag == 1 ) {
	
			runPipe(arguments,pathOfShell);
			
		} else if (IOreturnValue > 0 && pipeFlag == 0) { //This is the case where user wants I/O redirection.
			
			reDirection(arguments,pathOfShell,IOreturnValue);
			
		}
		
		
	}
	
	} else { printf("NOTHING ENTERED\n");} // this will print if the user has entered nothing into the shell.




	
	
	
	// the function calls above change the values of the global variables, so
	// we must reset their values to -1.

	indexOfIO = -1; indexOfIO_Two = -1; indexOfPipe = -1;
	
	// Free previously allocated memory.
	free(commandLineString);
			
	
}} }
	


// calls the exit function in C, to exit from the program.
void my_exit() {
 exit(1);	
}

// clears output from the shell.
void my_clear() {
	system("clear");
}

// Echo will output the parameter stringToBeEchoed to the shell. 
void my_echo(char **stringToBeEchod){
	
	int index = 1;
	
	while(stringToBeEchod[index] != NULL) {
	printf("%s ",stringToBeEchod[index]); index++;	
	}
	printf("\n");
}

// This function will pause the shell, and will wait for the user to press enter
// for the shell to be unpaused.
void my_pause(){
	printf("Shell is paused, press enter to continue\n");
	char c;
	while ( (c = getchar()) != '\n'){}
}
// This function will print out the environment variables.
void my_environ(){
	extern char **environ;
	int i = 0;
	
	while(environ[i]) {
		printf("%s\n", environ[i++]); 
	}
}

// my_dir() will output all the directories in the current working directory.
void my_dir(){
		 DIR           *d;
  struct dirent *dir;
  d = opendir(".");
  if (d)
  {
    while ((dir = readdir(d)) != NULL)
    {
      printf("%s\n", dir->d_name);
    }

    closedir(d);
  }
}

// This function changes the current working directory using the chdir function.
// The directory we would like to change to is passed to my_cd via the path 
// variable.
void my_cd(char *path){
 chdir(path);	
}

// My help will output the user manual to the shell.
void my_help(){
	char c;
		FILE* fp;
			// The user manual's name is readme. So we open that file, only in 
			// reading mode. 
	        if (( fp = fopen("readme.txt", "r") ) == NULL) {
            perror("opening file, my_help");
        }
        while(( c = fgetc(fp) ) != EOF) {
            fputc(c, stdout);
        }
 
    fclose(fp);
}

// This function will split the user input by seperating the input based on white
// space. The split up words will be stored in a array of strings.
char **parseInput(char *input) {
	// arrayOfTokens will store the arguments in an array. 
	char **arrayOfTokens = malloc(sizeof(char*) * TOKEN_BUFFER_CMDLINE);
	char *token;
	
	// We will use strtok to create tokens out of the user input.
	// The delimeter is whitespace. 
	token = strtok(input,"\t\n\v\f\r ");	
	int index = 0;
	
	while(token != NULL) {
		// save our token in this array
		arrayOfTokens[index] = token;
		
		// strtok has its own internal state, from the last time it was invoked.
		// use NULL to imply that you want the function to begin from the last
		// saved point. 
		token = strtok(NULL, "\t\n\v\f\r ");
		index++;
	}
	
	// make the last index without a string equal NULL.
	arrayOfTokens[index] = NULL; 
	
	sizeOfTotalArgs = index;
	
/*	// testing
	int j = 0;
	
	while (arrayOfTokens[j] != NULL) {
		printf("%s\n",arrayOfTokens[j]);	
		j++;
	} */
	
	return arrayOfTokens; 
}

// This function will check if the parsed input contains a string matching the
// built in strings.
int check_Builtins(char **arguments) {
	
	int index = 0;
	
	// Using strcmp to check if the parsed input matches any of the strings in the builtin
	while(strcmp(arguments[0],builtins[index]) != 0 && index < 9) { index++;}
			
		if (index == 9) return -1; //return -1 if there is no match.
		else return index; // the return value will depend on the index in which we had a match. 
		// { "exit", "clear", "help", "dir", "pause", "shellPath", "environ", "echo", "cd"} So
		// the return value will be 0-9 if there is a match corresponding to the index of built in 
		// array. 
	}
	
int check_IO(char **arguments) {
	
		
		/*
		
		Will return negative one if inaccurate pipes were sent through.
		
		Will return 0 if no redir characters found
		
		return 1 if only <
		
		return 2 if only >
		
		return 3 if only >> 
		
		return 4 if < and >
		
		return 5 if < and >>
		
		
		
		*/
		
		int index = 0;
		int indexForIOstrings;
		
		// These flags will determine if <, >, >> are in the arguments parameter.
		int flagForInpReDir = 0; // corresponds to <
		int flagForOutReDir = 0; // corresponds to >
		int flagForOutReDir_Append = 0; //corresponds to >>
		//int pipeFlag = checkPipe(arguments);
		
		//char IOstrings[3][3] = { "<",,">",">>"};
		
		// this loop iterate through the arguments.
		while(arguments[index] != '\0') {
			indexForIOstrings = 0;
			
			// this loop iterates through the IOstrings array initialized at the beginning.
			while(indexForIOstrings < 3) {
			if(strcmp(IOstrings[indexForIOstrings],arguments[index]) == 0 && indexForIOstrings == 0){
				
				// This checks to see the correct usage of redirect characters. In this case we're
				// testing to see "< >, << > or > >".
				if (flagForOutReDir == 1 || flagForOutReDir_Append == 1 || flagForInpReDir == 1) return -1;
				
				// We will set the indexOfIO to the value of index if there is a match. 
				indexOfIO = index; flagForInpReDir = 1;
				
			}
			if(strcmp(IOstrings[indexForIOstrings],arguments[index]) == 0 && indexForIOstrings == 1) {
				
				// This checks to see the correct usage of redirect characters. In this case we're
				// testing to see "> > or >> >. 
				if(flagForOutReDir == 1 || flagForOutReDir_Append == 1) return -1;
				
				// set flag to 1 if match
				flagForOutReDir = 1;	
				
				// set indexOfIO_two to value of index if there was a < before. (This is for the < > case)
				// else you'll set indexOfIO to value of index. 
				if(flagForInpReDir == 1) { indexOfIO_Two = index; } else {indexOfIO = index;}
				
				
				
				
			}
			if(strcmp(IOstrings[indexForIOstrings],arguments[index]) == 0 && indexForIOstrings == 2) {
				
				// This checks to see the correct usage of redirect characters. In this case we're
				// testing to see "> >> or >> >>. 
				if(flagForOutReDir_Append == 1 || flagForOutReDir == 1) return -1;
				
				// set flag to 1 if match
				flagForOutReDir_Append = 1;	
				
				// set indexOfIO_two to value of index if there was a < before. (This is for the < > case)
				// else you'll set indexOfIO to value of index. 
				if(flagForInpReDir == 1) { indexOfIO_Two = index; } else {indexOfIO = index;}
			}
			indexForIOstrings++;
			}
			index++;	
		}
		
		// return values based on set flags. 
		
		 if(flagForInpReDir == 1 && flagForOutReDir == 0 && flagForOutReDir_Append == 0) {
			return 1;
		} else if(flagForInpReDir == 0 && flagForOutReDir == 1 && flagForOutReDir_Append == 0) {
			return 2;
		} else if(flagForInpReDir == 0 && flagForOutReDir == 0 && flagForOutReDir_Append == 1) {	
			return 3;
		} else if(flagForInpReDir == 1 && flagForOutReDir == 1 && flagForOutReDir_Append == 0) {
			return 4;
		} else if(flagForInpReDir == 1 && flagForOutReDir == 0 && flagForOutReDir_Append == 1) {
			return 5;
		}
		
		return 0;
		
	
} 






// check is user has provided an amperstand at the end of their input.
int runInBackgroundRequest(char **arguments) {
	
	int index = 0;
	char *amp = "&";
	int flag = 0;
	while (!flag && arguments[index] != '\0') {
		if ( strcmp(arguments[index],amp) == 0 ) flag = 1; //set flag to true if found.
		index++;
	}
	
	if(flag == 0) return 0; //returns 0 if no ampersan found
		else return 1; // return 1 if there is an ampersand
}

// checks to see if pipe value is in parsed user input. 
int checkPipe(char **arguments) {
	
	int index = 0;
	char *pipe = "|";
	int flag = 0;
	while (!flag && arguments[index] != '\0') {
		if ( strcmp(arguments[index],pipe) == 0 ){ flag = 1; indexOfPipe = index;}
		index++;
	}
	
	if(flag == 0) return 0;
		else return 1;
}
	

// This function will split an array of strings, and return a subarray.
// int beginning is the start of where you want to create subarray, and end is the
// end of where you want to create a subrray.
char **splitStringArr(char **arguments,int beginning, int end){
	
	char **newArr = (char**) malloc( (sizeof(char*) * COMMAND_LINE_BUFFER));
	
	int index = 0;
	int indexForArgs = beginning;
	//printf("%d\n",indexForArgs + 1);
	while(indexForArgs < end && arguments[indexForArgs] != '\0') {
		//newArr[index] = strdup(arguments[indexForArgs]); 
		//memcpy
		newArr[index] = (char*)malloc(strlen(arguments[indexForArgs]+1));
		strcpy(newArr[index],arguments[indexForArgs]);
		index++; indexForArgs++;
	}
	
	newArr[index] = NULL;
	
	return newArr;
}

// This function changes the standard input, and output of the shell.
void reDirChannels(char *file, int type_of_ReDir_Request) {
	
	// This will change the standard input.
	if(type_of_ReDir_Request == 1) {
		int in = open(file, O_RDONLY);
		dup2(in, 0);
		close(in);
	} else if (type_of_ReDir_Request == 2) { // This will change the standard output to that of a file.
		
		// O_ lets u write and trunc. O_S sets permission of file
		int out = open(file, O_WRONLY | O_TRUNC | O_CREAT, S_IRUSR | S_IWUSR  | S_IWGRP | S_IRGRP);
		dup2(out, 1);
		close(out);
	} else if (type_of_ReDir_Request == 3) {
		
		// you can append for this change of standard output.
		
		int out = open(file, O_APPEND |  O_WRONLY | O_CREAT,S_IRUSR | S_IWUSR  | S_IWGRP | S_IRGRP);
		dup2(out, 1);
		close(out);
			
	}
	
}

// this function will run the builtin functions.
// The returnValue argument should be the value from the check_Builtins function. 
void runBuiltIns(int returnValue, char **arguments, char *pathOfShell) {
	
	
		if (returnValue == 0) {
		my_exit();
		}
	else if (returnValue == 1) {
		my_clear();
	}
	else if (returnValue == 2) {
		my_help();
	}
	else if (returnValue == 3) {
		my_dir();
	}
	else if (returnValue == 4) {
		my_pause();
	}
	else if (returnValue == 5) {
		printf("%s\n",pathOfShell);
	}
	else if (returnValue == 6) {
		my_environ();
	}
	else if (returnValue == 7) {
		my_echo(arguments);
	}
	else if (returnValue == 8) {
		my_cd(arguments[1]);
	}
	
	
}

// This function will invoke user arguments if there is no pipe or I/O redirect chars.
// ampReturnValue is the return value from runInBackgroundRequest.
void executeCommands(char **arguments, char *pathOfShell, int ampReturnValue) {
	
	
// check to see if the arguments contain builtins. If the return value is 0 or greater
// Then we will use the run_Builtins to invoke command.	
int returnValue = check_Builtins(arguments);

		if(returnValue >= 0 && returnValue <= 8) {
			
			runBuiltIns(returnValue,arguments,pathOfShell);
				
		} else { // If the arguments don't match any of the builtins then we invoke from bin.
			pid_t pid;
			
			pid = fork();
			
			if ( pid == 0) {
				
				// execvp will invoke the external command from the bin.
				if(execvp(arguments[0],arguments) == -1) {printf("INVLALID COMMAND\n");my_exit();}
				
			} else {
				
				// This will determine if we have to run the command in the background.
				if(ampReturnValue == 0) {
					wait(NULL);	
				}
				
			}
			
		}
			
	
}

// This function will invoke piping functionality, by creating two children processes.
// We will use the output of one child as the input for the other child using the pipe(),and
// dup2() functions. 
void runPipe(char **arguments, char *pathOfShell) {
	
	// We need two arrays of strings. One array for the commands before the pipe symbol
	// The other array for commands after the pipe symbol.
	char **preOprArgs;
	char **afterOprArgs;
	
		// Use the splitStringArr() function to create the subarrays. 
		preOprArgs = splitStringArr(arguments,0,indexOfPipe);
		afterOprArgs = splitStringArr(arguments,indexOfPipe+1,sizeOfTotalArgs);
		
		
		// We check if the commands before the pipe symbol contain built in commands.
		int builtInCheck = check_Builtins(preOprArgs);
		
		//{ "exit", "clear", "help", "dir", "pause", "shellPath", "environ", "echo", "cd"};
		//  check if the built in matches help, dir,shellPath,environ or echo. As these are the 
		// only valid builtins that will work with piping. We also check if there is no built ins.
		if (builtInCheck == 2 || builtInCheck == 3 || builtInCheck == 5 || builtInCheck == 6 || builtInCheck == 7 || builtInCheck == -1) {
			pid_t pid1, pid2;
		   int pipefd[2];
		   pipe(pipefd);
				 pid1 = fork();
		   if (pid1 == 0) {
			  // Change standard output to write into other command
			  dup2(pipefd[1], STDOUT_FILENO);
			  close(pipefd[0]);	
			  int returnValue = check_Builtins(preOprArgs);
		   if ( returnValue >= 0 && returnValue <= 8) {
		   	   runBuiltIns(returnValue,preOprArgs,pathOfShell);
		   	   
		   } else { // this case there is no builtin, so we invoke an external command from the bin. 
		   	   //execvp(preOprArgs[0], preOprArgs);
		   	   if (execvp(preOprArgs[0], preOprArgs) == -1){printf("INVLALID COMMAND\n");my_exit();}
		   }
			  
			 
			 
			  my_exit(); //exit in case we ran builtin. 
		   }
		   // Create our second process.
		   pid2 = fork();
		   if (pid2 == 0) {
			  // Change standard input to accept input from other command. 
			  dup2(pipefd[0], STDIN_FILENO);
			  close(pipefd[1]);
			  
			    int returnValue = check_Builtins(afterOprArgs);
		   if ( returnValue >= 0 && returnValue <= 8) {
		   	   runBuiltIns(returnValue,afterOprArgs,pathOfShell);
		   } else {
		   	   //execvp(afterOprArgs[0], afterOprArgs);
		   	    if (execvp(afterOprArgs[0], afterOprArgs) == -1){printf("INVLALID COMMAND\n");my_exit();}
		   }
			  
			
			  
		   }
		   // Close both ends of the pipe.  
		   close(pipefd[0]);
		   close(pipefd[1]);
		  
		   
		   wait(NULL); //wait for both children to finsih. 
		   wait(NULL);	 
	
			
		} else { printf("Cannot perform piping operation\n");	
	}
}


// This function will carry out the STDIN or STDOUT redirection from/to files.  
void reDirection(char **arguments, char *shellPath,int IOreturnValue) {
	
	// sub array declarations. Two needed if only one I/O character, or three if there
	// are two I/O characters. We will know the amount of I/O chars depending on the check_IO function.
	char **preOprArgs,**afterOprArgs,**thirdSetArgs;
	int i = 0;
	if ( IOreturnValue > 0) { 
		
		// only use two subrrays if return value is between 1 and 3.
		if (IOreturnValue <= 3 && IOreturnValue >= 1) {
		
		preOprArgs = splitStringArr(arguments,i,indexOfIO);
		afterOprArgs = splitStringArr(arguments,indexOfIO+1,sizeOfTotalArgs);

		} else if (IOreturnValue > 3 ) {
		
		// three subarrays if return value is above 3.
		preOprArgs = splitStringArr(arguments,0,indexOfIO);
		afterOprArgs = splitStringArr(arguments,indexOfIO+1,indexOfIO_Two);
		thirdSetArgs = splitStringArr(arguments,indexOfIO_Two+1,sizeOfTotalArgs);
	
		}
		
		
		pid_t pid;
		
		pid = fork();
		
		if (pid == 0) {
					 		
			 	
			 	if (IOreturnValue == 1 ) { // This case is for "<"
			 		reDirChannels(afterOprArgs[0],1); // change standard input, to the file 
			 		// described by afterOprArgs
			 		
			 		
			 		 if(execvp(preOprArgs[0],preOprArgs) == -1) {
			 		 printf("INVLALID COMMAND\n");my_exit(); }
			 		
			 		 
			 		
			 	} else if (IOreturnValue == 2) { // This case is for " >"
			 		
			 		// change standard output by file described by afterOprArgs.
			 		// File will be truncated
			 		reDirChannels(afterOprArgs[0],2); 
			 		
			 	
			 		// Check if preOprArgs is a builtin command. Run builtins if it is. 
			 		int i = check_Builtins(preOprArgs);
			 		if ( i >= 0) {
			 				runBuiltIns(i,preOprArgs,shellPath); my_exit();
			 		} else {
			 		 if(execvp(preOprArgs[0],preOprArgs) == -1) {
			 		 printf("INVLALID COMMAND\n");my_exit(); 
			 		 }
			 		}
			 		 
			 		
			 	}else if (IOreturnValue == 3) { // This case is for ">>"
			 		
			 		
			 		// change standard output by file described by afterOprArgs.
			 		// File will be appended.
			 		
			 		reDirChannels(afterOprArgs[0],3);
			 		
			 		// Check if preOprArgs is a builtin command. Run builtins if it is. 
			 		int i = check_Builtins(preOprArgs);
			 		if ( i >= 0) {
			 				runBuiltIns(i,preOprArgs,shellPath); my_exit(); 
			 		} else {
			 		 if(execvp(preOprArgs[0],preOprArgs) == -1) {
			 		 printf("INVLALID COMMAND\n");my_exit();
			 		 }
			 		 } 
			 		
			 	} else if (IOreturnValue == 4) { // This case is for "< >"
			 		// change standard input, to the file 
			 		// described by afterOprArgs
			 		reDirChannels(afterOprArgs[0],1);
			 		
			 		// change standard output by file described by thirdSetArgs.
			 		// File will be truncated
			 		reDirChannels(thirdSetArgs[0],2);
			 		executeCommands(preOprArgs,shellPath,0);
			 		
			 		 }
			 	else if (IOreturnValue == 5) { //This case is for " < >>"
			 		
			 		// change standard input, to the file 
			 		// described by afterOprArgs
			 			reDirChannels(afterOprArgs[0],1);
			 			
			 			
			 			// change standard output by file described by thirdSetArgs.
			 		// File will be appended
			 			reDirChannels(thirdSetArgs[0],3);
			 			executeCommands(preOprArgs,shellPath,0);
			 			
			 	} else if (IOreturnValue == -1) { //Invalid I/O redirect chars.
			 		printf("INACCURATE USAGE OF I/O CHARS\n");
			 		my_exit();
			 				
			 	}
			 		
		} else {
			
			wait(NULL);
		}
		
	}
}

	
	
	



	




	
