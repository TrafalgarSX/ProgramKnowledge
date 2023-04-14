gdb: The GNU  Project Debugger

* run       run program
* break     break后可以接 函数名、行号
* next      run the next line of code
* list      print out the lines of code around where i currently am
* print     print out the value of a variable or expressions or    derefference a pointer
* quit      leave current instance of gdb
* up        peek up the call stack
* down      peek down the call stack
* display   tell gdb display the value of a variable every time
* undisplay stop display  **but use display id, not variable name**
* backtrace print the entire call stack
* step      unlike next, step go inside the line of code
* continue  keep running code util meet next Breakpoint
* finish    tell gdb to finish this current function call and then stop once we finish the call
* watch     watch a variable until it has been changed then stop
* info breakpoints  print out all information of breakpoint、watchpoint
* delete    delete breakpoint、watchpoint by using id and you can use `delete`  to delete all breakpoints or watchpoints 
* wahtis    tell you the type of the variable you give
* target record-full   tell gdb to record what happens in order to kind of replay in reverse
* reverse-next reverse-step reverse-continue
* set var  variable_name = value