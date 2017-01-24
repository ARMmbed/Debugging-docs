# Compile time errors

Compile time errors and warnings are caused by incorrect syntax, or misuse of variables or functions. An error will prevent the compile process from completing (and therefore no binary file will be created). A warning will not prevent the binary from being created, but you should still review the warning as it may mean that your code is not going to do what you had intended.

Common errors are:

* Missing declarations of variables and interfaces, leading to `Identifier undefined` errors.
* Missing semicolons (`;`). Semicolons are required at the end of each line.
* Missing quotes or brackets (`""`, `()`, `[]` or `{}`). These are used in pairs to contain various types of statement. The compiler will report an error if you have not used them in correct pairings.
* Always tackle the very first error that is reported, as later errors might be as a result of the first one, and will disappear when the first one is corrected.

If you are seeing a compile time error or warning that you do not understand, Google will usually find explanations of the error message, or post to the [mbed Forums](https://developer.mbed.org/questions/).
