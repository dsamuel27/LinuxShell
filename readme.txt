---------------------------------------------------------------------------
User Manuel For My Shell
---------------------------------------------------------------------------
To launch the actual shell you type: ./myshell

If you wish to input commands into the shell using a batch file then you type: ./myshell name_of_file
---------------------------------------------------------------------------
Built In Commands

The shell has nine built in commands. The words in quotations indicate the word you type in to execute the built in command. 

1. “exit” - exit allows the user to terminate the running shell. 

2. “clear” - clear allows the user to remove all the output displayed on the shell.

3. “help” - help will display the user manual of the shell.

4. “dir” - dir will list the contents of the directory 

5. “pause” - pause will halt the execution of the shell until the user presses ‘Enter’

6. “shellPath” - shellPath will display the directory path to the shell’s executable file. 

7. “environ” - environ will display the environment variables.

8. “echo argument1 argument2 … argumentN” - echo takes arguments from the user, and then  displays them in the shell. 

9. “cd path” - cd takes an argument which represents a directory the user would like to move to. Cd will move to the path if its adjacent to the current working directory, and if it exists. Thus changing the user’s current working directory. *Note to move backwards the user should enter “..” as path.*

---------------------------------------------------------------------------
External Commands

The user can enter other commands provided in the /bin/ directory.

Running commands in the background

The user can run commands in the background of the shell by adding an amperstan (‘&’) to the end of their input.  

Input and Output Redirection

The user is able to read input from a file, or redirect output to a file. Using the following symbols:

1. ‘ <’ - this symbol will allow the user to take input from a file

Usage of ‘ < ‘ : command < input_file

2. ‘>’ - this symbol will allow the user to direct output into a file. This user directed output will overwrite any other information in the file provided. If a file does not exist, a file is created with the name provided by the user, and the output from the command is stored in the newly created file. 

Usage ‘>’ : command > input_file

3. ‘>>’ - this symbol will allow the user to direct output into a file. This user directed output will be appended to the file. If a file does not exist, a file is created with the name provided by the user, and the output from the command is stored in the newly created file. 

Usage ‘>’> : command >> input_file

*Note that I/O characters can be used together*

Combined Usage:      command < input_file > output_file
				 command < input_file >> output_file

These are the only valid ways to combine the I/O characters!
---------------------------------------------------------------------------
Piping

The user will be able to use the output of one command, as the input for another command using the symbol “|”. 

Usage: command_One | command_Two
---------------------------------------------------------------------------
Other Info

Note that if the user enters an invalid command, the shell will output “INVALID COMMAND”

---------------------------------------------------------------------------