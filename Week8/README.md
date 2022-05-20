# Exercise 8 : Resource Constraints 
## Memory layout
```
 /**
 * ############################################################################
 * #  .data  #  .bss  #       newlib heap       #          MSP stack          #
 * #         #        #                         # Reserved by _Min_Stack_Size #
 * ############################################################################
 * ^-- RAM start      ^-- _end                             _estack, RAM end --^
 */
 ```

Default memory layout starts with .data -> .bss -> heap (grows up) -> stack (grows down)
