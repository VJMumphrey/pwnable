# Solution
Simple chanllenge that was meant to test the users knowledge of file descriptors on linux.

Reading the source code you can see that the program "fd" is supposed to take in one argument.
This one argument is supposed to be a number that is subtracted by 0x1234.
Once this is done a string is read from a file descriptor fd and stored in a buffer that is compared to
"LETMEWIN".

The file descriptor that is used to read user input is stdin or 2, so I need to find a number that once
subtracted by 0x1234 will equal 2.

![finding_number](https://github.com/VJMumphrey/blob/main/pwnable/screenshots/fd/finding_number.png)

Once this is done I need to run "fd" with the this number. Then I need to type in "LETMEWIN" into the open stdin file descriptor. 

![printing_flag](https://github.com/VJMumphrey/blob/main/pwnable/screenshots/fd/printing_flag.png)

Thats it

Pretty simple start