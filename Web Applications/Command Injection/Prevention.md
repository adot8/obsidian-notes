We should always avoid using functions that execute system commands, especially if we are using user input with them.

Instead of using system command execution functions, we should use built-in functions that perform the needed functionality, as back-end languages usually have secure implementations of these types of functionalities.

If we needed to execute a system command, and no built-in function can be found to perform the same functionality, we should never directly use the user input with these functions but should always validate and sanitize the user input on the back-end.

Implement Input validation and sanitization