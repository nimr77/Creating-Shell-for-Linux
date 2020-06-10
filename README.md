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
