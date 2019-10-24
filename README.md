# MIPS

* You get a reference sheet so don't bother memorising specific registers, instructions, calling conventions
* General architecture
    * 32 general purpose registers of 32 bits
        * on reference sheet
    * Special registers
        * Instruction Register (IR)
        * Program Counter (PC)
        * Memory address/read/write (MAR/MRR/MWR)
    * Arithmetic Logic Unit
    * Memory
        * Byte addressible (4 byte words means you often multiply by 4)
        * Segments assign implicit meaning to words
        * OS segment
        * Text segment (executable code)
        * Data segment
            * Static data (known at compile time)
            * Heap segment (growing down, runtime allocatable)
            * Stack segment (growing up, used for function locals)
        * Then another OS segment
* Address dereferencing
    * In this unit, all of our arrays have the first element as the length, then the actual elemnts are 1 indexed
        * so allocate an extra word (4 bytes)
    * NOTE: always increment by a multiple of 4 (a word)
    * sw $t0, 4($sp) # mem[$sp + 4] = $t0
    * if you allocate array dynamically, you store the pointer, so use lw not la to retrieve it correctly
    * Classic array acess pattern
```
lw  $t0, the_list  # get pointer to start of list (actually the length "element")
lw  $t1, i         # load i
sll $t1, 2         # multiply i by 4 to get byte offset
add $t0, $t0, $t1  # $t0 is now &the_list[i-4]
lw  $t2, 4($t0)    # $t2 = the_list[i]
sw  $t3, 8($t0)    # the_list[i+1] = $t3
```

* Functions
    * In this unit, always use stack for arguments and locals (unless optimising)
    * Follow conventions in the reference sheet
    * Decrement $sp to allocate stack space - "push"
    * Increment $sp to deallocate - "pop"
    * Recursion - don't worry about it, stack deals with it
    * Classic memory diagram of a stack frame

| Register which points here | meaning for humans  | example value | how to access                     |
|----------------------------|---------------------|--------------:|-----------------------------------|
| $sp ->                     | (local) result      |           125 | -8($fp)                           |
|                            | (local) other_local |            42 | -4($fp)                           |
| $fp ->                     | (saved $fp)         |    0x7FFF03A0 | after deallocating locals, 0($sp) |
|                            | (saved $ra)         |    0x0040005C | after deallocating locals, 4($sp) |
|                            | (arg) base          |             5 | 8($fp)                            |
|                            | (arg) exp           |             3 | 12($fp)                           |
* Gotchas
    * .ascii "Hello" does not null terminate (when string ends is not specified), so please use .asciiz
    * You cannot use ALU operations (arithmetic, logic) on memory, you must load them into registers first, then store the results
    * When doing comparisons, you usually want the exact opposite e.g. say you want if (x > 2) {...} you want to jump after the block on condition !(2 < x) (slt then bne)
    * do not forget unconditional jumps at end of if for an if-else, or at end of loop
* Compiler optimisations
    * Constant folding - evaluate constant expressions at compile time e.g. 24 * 365 => 8760
    * Multiplication by powers of two - use sll e.g. instead of x*8 do x << 3
    * Division by powers of two - use srl (logical) or sra (arithmetic)
    * Reordering expressions to use fewer registers
    * Reusing reisters instead of reading/writing from/to memory all the time
    * Changing loop conditions (if safe) or deferring them until after first iteration
    * Breaking functional calling conventions to reduce instructions, including passing arguments by registers, not using $fp, not using $ra (if safe)

# Algorithms

* Sorting
    * n is number of element in the list
    * You might need to multiply complexity by comparison cost e.g. if strings

| Algorithm      | Overview                                       | Best                               | Worst  | Stable                  | Incremental                    |
| Bubble sort    | Repeatedly swap adjacent out-of-order elements | O(n^2), or O(n) if detecting swaps | O(n^2) | Yes (strict inequality) | Yes (if element added to front, which sucks because it is O(n)) |
| Selection sort | Repeatedly swap minimum element to front       | O(n^2)                             | O(n^2) | No                      | No                             |
| Insertion      | Build a sorted sublist at the front            | O(n)                               | O(n^2) | Yes (strict inequality) | Yes (if element is added to back, which is usually O(1)) |

# Misc

* Python variable representation
    * Your variable names do not hold the actual values
    * They hold pointer to actual values
    * This is why lists are mutable - your variable name still points to the same thing
    * Be careful
* Assertions
    * Check preconditions which must not be violated in between your own code
    * Do not use this to validate user input - this is an "expected" violation of requirements but not precondition
    * AssertionError raised if False
* Exceptions
    * Use in "exceptional" cases e.g. invalid user input
* Unit testing
    * Test valid and invalid cases of code
    * Ensure correct results produced
    * Ensure correct exceptions raised