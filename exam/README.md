**PRÜFUNGSFRAGEN (UV Grundlagen Compilersysteme SS2025)** <br>
Daniel Krall <br>
daniel.krall@stud.plus.ac.at

---

`01` <br>
Q: `4` <br>
How does a scanner process a stream of characters into a stream of symbols for the parser? Which types of symbols are there and how are whitespaces and comments handled?

A: `4` <br>
The frontend of a compiler is divided into two parts, the scanner and the parser. While we could implement both in one single step, separating them simplifies changes, for instance when using different encoding formats for characters. The scanner takes characters from an input stream and processes them, turning them into a sequence of symbols to be handled by the parser in the next step. In selfie we use the ’get_symbol()’ procedure to get the next valid symbol from our input. In general, a character is read and then the scanner has to identify whether it is just a single-character symbol (like ‘&’ or ‘x’) or if it is a part of a multi-character symbol, where all characters are processed and then grouped together (like ‘!=’ or a variable called ‘calc_result’). It must also be checked whether a processed symbol is even a valid part of the given programming language, according to a given specification (often using regular expressions), and if that is the case, the symbol is accepted, otherwise it is rejected (with a corresponding error message). <br>
Depending on the implementation of the scanner, a whole sequence of symbols (in a fitting data structure) or only the last read symbol could be stored. <br>
In compilers for most programming languages, single-line comments (e.g. ‘//’ or ‘#’) and block comments (e.g. ‘/* … */’) are handled by just being ignored, as they do not add anything meaningful to the structure of the program, they just improve readability of the code. For whitespaces it depends on their usage, for example spaces separating two symbols are important while new lines (‘\n’) and spaces or tabs for indentation can also be ignored.

---

`02` <br>
Q: `4` <br>
How does the compiler's parser work? What are single-pass and multi-pass parsers and what is the difference between top-down and bottom-up parsing?

A: `3` <br>
The second part of a compiler’s frontend (after the scanner) is formed by the parser, which uses a sequence of symbols (from the scanner) and checks its syntax, validating that the given input symbols are part of the given programming language. A valid input is precisely specified by the language’s formal grammar (in case of selfie, C* uses context-free grammar), which also means that, for example, a missing semicolon or parenthesis can be detected and reported using an error message. In the parsing step, a so-called “internal representation” (IR) of the program is created, which represents the internal structure of the code, like function calls and loops. The most common types used for IR are syntax trees, like parse trees and the Abstract Syntax Tree (AST). <br>
In a single-pass parser the internal representation is created by reading the input sequence only once, which is a relatively simple and quick approach at parsing. This results in some downsides, as error detection is limited to some degree and optimization is locally limited, in both cases due to the fact that later code (that has not been parsed yet) can not be utilized. This is improved upon by the multi-pass parser, where the internal representation is built by passing over the input several times, improving the result in each pass. Although it is a slower and more complex approach, error detection and optimization are improved as the entire code can be used. <br>
The difference between top-down and bottom-up parsing is from where we approach the input, so from high-level to details (top-down) and the other way around (bottom-up). In the top-down approach we start building a tree at the top, so from the root (start symbol), down to the leaves, by applying our grammar rules and matching the input symbols. Once again, a simple approach, that can be done by hand, with its drawbacks, as it cant’ handle certain grammars (like left-recursion) and if may be inefficient in certain cases. Bottom-up parsing, as mentioned, is the opposite, here we start from the input symbols and combine them according to our grammar rules until we reach the start symbol, so we are building the tree from the leaves to the root. Although a more complex method, it can deal with more complicated grammars (like left-recursion).


---

`03` <br>
Q: `3` <br>
How are symbol tables implemented and what is their purpose? Which data structures are used in selfie and how do they work?

A: `3` <br>
Important information about procedures, variables and identifiers (name, type, value, scope, memory address and additional data) is stored in symbol tables which forms an important part in the different stages of compilation (parsing, analysis, code generation). We store these values in a symbol table by mapping a key (identifier) to a value (information) which therefore enables us to manage and retrieve them easily. We can also operate in different scopes, like a global and a local symbol table, depending on the scope a given identifier is used in. This information helps in error detection as well, for example in type mismatches (when the stored information is different) or undeclared variables (no entry). <br>
Symbol tables can be implemented using various different data structures, like Hash Tables, Lists or Trees. Selfie uses hash tables (similar to arrays) where an entry’s key is hashed using a hash function and the resulting hash value is used as its index, this allows insertion and searching (when there are no collisions) in constant time. In case of more than one entry having the same has value (a collision), singly-linked lists are used to store multiple different entries at the given index. Such a list consists of two parts, the entry and a pointer to the next node (entry). These allow insertion in constant time but searching in linear time (in  the number of entries), which can be slow in case of many collisions and therefore long lists. <br>
In selfie we use the ‘create_symbol_table_entry’ procedure to create a new entry, and we retrieve it with the ‘get_scoped_symbol_table_entry’ procedure. 


---

`04` <br>
Q: `2` <br>
What is the Register Allocation Problem in compiler systems? What are some of the approaches used to deal with it?

A: `2` <br>
One difficulty in compiler systems in the problem of register allocation, describing the fact that the code that was generated at compile time includes a large number of variables and intermediate results which must all, at some point, be assigned to a very limited number of physical registers in the CPU at runtime, for instance, in selfie we have only seven registers. Two variables that need to be “live” at the same time should both be in a register but cannot be in the same. Accessing data in these registers is much faster than accessing it in memory and we therefore want to minimize accessing memory as much as possible, but whenever there are not enough registers available, we have to “spill” information to memory. <br>
There are several approaches to dealing with this problem, one example of a simple and easily predictable approach would be a stack allocator (used in selfie), where memory is handled like a stack in the last-in, first-out (LIFO) principle. Another approach is the so-called “bit vector”, where an array of bits is used, where every entry represents one variable with ‘0’ meaning “not in use” and ‘1’ meaning “in use”. With that information a compiler can decide which variables are still in use and need to stay in registers and which can be spilled and replaced by different ones. There are also more complex approaches based on which variables are in use, and which are not, which are similar to the graph colouring problem, where the best solution is approximated. A graph is built where the variables are represented by the nodes and the edges connect the nodes that need to be live at the same time and therefore are required to be in different registers.

---

`05` <br>
Q: `3` <br>
What are the two main types of memory allocation? When and where are the memory locations assigned?

A: `3` <br>
Memory allocation describes the assignment of variables in the program to physical memory in a computer. The two main types of memory allocation depend on when this allocation is managed, this being either at compile-time (static memory allocation) or runtime (dynamic memory allocation).
Static memory allocation at compile-time is used when the size of variables, and therefore their memory requirements, are known at compile-time. This involves global variables, static variables and constants (e.g. ‘const’ variables or string literals). Their size is known and does not change, with the memory locations being fully determined at compile-time and are stored in the data segment of the memory space, where they remain reserved during runtime of the program. The memory is then automatically deallocated after the program terminates.
The other type is dynamic memory allocation, which is executed dynamically during the runtime of a program, when the required memory is not known beforehand, like in dynamic data structures (such as variably-sized arrays) and local variables. Usually, memory is allocated on the heap by using functions (in C) like ‘malloc()’ and remain reserved there until being manually deallocated through functions like ‘free()’, although stack allocation may also be used. 


---

`06` <br>
Q: `2` <br>
What is a self-compiling compiler and how is bootstrapping used to achieve this characteristic? How can we confirm that self-compilation works correctly?

A: `2` <br>
Self-compiling (also called self-hosting) means that a compiler, written in a specific programming language (like C* for selfie) can compile its own source code (in C*) into machine code, and therefore produce an executable version of itself. <br>
Selfie, a C*-compiler written in C* can compile itself using ‘$ ./selfie -c selfie.c’. <br>
Examples, other than selfie, are ‘gcc’ (a C-compiler written in C) and ‘javac’ (a Java-compiler written in Java), with many others in different programming languages in existance.<br>
To achieve this characteristic, bootstrapping is frequently utilized, which is a way of creating a new compiler from scratch. This often happens in several steps, where an initial compiler, the so-called “bootstrapping compiler” may be written (often in Assembly language). An already existing compiler may also be used, like using ‘gcc’ to compile a new C-compiler. This initial compiler is then used to compile a version of the new, written from scratch, compiler. There may be several iterations of this, improving the “new” compiler in every step. 
To confirm whether a self-compiling compiler is actually working correctly, we once again utilize an already existing, trusted compiler, like ‘gcc’. In the first step we compile the source code for our new self-compiling compiler with the trusted compiler (written in another language). We then use this resulting compiler to once again compile the source code. After these two steps we compare the binaries of the two resulting compilers (one compiled with a trusted compiler and the other self-compiled). If the two are identical, we have confirmed that our self-compiling compiler is working properly. <br>

---

`07` <br>
Q: `3` <br>
What role do data types play in compilers? Which data types are there and which are supported in selfie?

A: `4` <br>
Data types form an important part of compilers as they are used to define the type of data a variable can hold, which in turn specifies its size, and rules for which operations may be performed on the variable. The compiler uses this information to allocate memory for the variable, depending on its size, and check whether the operation that is intended to be performed is valid for the given type. It also makes sure that variables of a certain type are handled consistently, which includes type mismatches (e.g. an ‘integer’ value being assigned to a ‘double’). The data type also ensures that the compiler chooses the correct machine instructions (e.g. how should ‘int value = 15‘ be handled compared to ‘double value = 15.5‘) and makes sure that type casting is possible. <br>
Data types are classified in different categories, which may differ depending on the programming language and literature but in general we have primitive (or basic) data types, which include ‘int’, ‘double’ and ‘char’ among others. Secondly, there are derived data types which, as the name suggests, are derived from primitive types, like ‘array’, ‘pointer’ or ‘function’. And lastly, user-defined types are used, which can be created by programmers according to their needs, which includes ‘class’ and ‘struct’.<br>
The only primitive data types supported by selfie are ‘uint64_t’ (unsigned integer with a fixed size of 64 bits) and ‘uint64_t*’ (a pointer to uint64_t). 

---

`08` <br>
Q: `2` <br>
What is the purpose of the prologue and epilogue in a compiled procedure and which operations are performed?

A: `3` <br>
In most cases, compiled procedures include a prologue and an epilogue, which help manage the execution of a procedure. For this, they are inserted by the compiler before (prologue) and after (epilogue) the procedure and are used to prepare the stack and registers for its execution. In selfie both are compiled in ‘compile_procedure()’. <br>
The prologue is executed before the procedure to prepare the stack and registers for the execution. It consists of several steps, including saving the frame pointer and return address of the caller (otherwise it would be overwritten), setting a new frame pointer for the current procedure and adjusting the stack pointer to allocate memory for local variables. Also, an important part is saving registers that must be kept for other executions, e.g. a register that contains the address of the next instruction may be overwritten by a procedure-call inside the current procedure. This process prepares a space on the stack for the actual procedure to be safely executed. <br>
The epilogue follows the procedure, it is executed before the procedure’s return and prepares the stack and registers for it. In this step the frame pointer and return address of the caller are restored, the stack pointer is moved back to restore the previous stack frame and memory for local variables and parameters is deallocated. This process makes sure that the program can resume in the same state it was in before the procedure was executed.
The prologue and epilogue are therefore essential when it comes to the correct execution of programs, especially when recursion or calls to other functions are involved.

---

`09` <br>
Q: `3` <br>
What does it mean for an Address to be "live" or "dead"? How are "lag" and "drag" associated with memory allocation? Is there a way of computing the "liveness" of Addresses?

A: `3` <br>
The term “liveness” plays an important role in the optimization of memory and register allocation, as it is used to check whether an address might still be needed in the future or not. An address is seen as being “live” when its value is needed later in the program and will therefore be read in the future and as a result, this address cannot be overwritten. The opposite would be a “dead” address, which will not be read again, and the address or register can be safely overwritten by the compiler. <br>
Two important terms associated with inefficient memory usage are ‘lag’ and ‘drag’. Lag means that there is a long time between the allocation of a register or memory and the first ‘write’ access, while ‘drag’ describes the other side, a long time between the last ‘read’ access and the dealloation of the memory or register. <br>
Unfortunately, we cannot compute whether an address is live or dead but what we can do is calculate reachability, to approximate liveness. When calculating this approximated set, we must be certain that every address outside our set is guaranteed to be dead. The resulting set may include dead addresses that we have calculated to be live but here we need to be on the safe side. These dead addresses may occupy memory but this is preferred over live addresses being overwritten. This approximation is, for instance, used by garbage collectors.

---

`10` <br>
Q: `2` <br>
What is "memory safety" and why is it so important in compilers? What role can a garbage collector play and how does it compare to manual memory management?

A: `3` <br>
Memory safety is an important concept for programs, with the aim of preventing faulty access on memory, like reading or (de)allocating memory it shouldn't. One potential violation would be "out-of-bounds" access, where an attempt is made to access an index of the array but is outside of said array. Another would be "use-after-free" where memory that was already deallocated is accessed. There is also a potential of "dangling pointers", where a pointer in a program is pointing to an invalid address, or "memory leaks" where "dead" addresses are never freed, as they are being calculated as "reachable" and therefore cannot be verified to be dead, and therefore take up valuable memory space. <br>
These violations could lead to crashes or security risks and their prevention is therefore very critical, for instance, by implementing out-of-bounds checks. <br>
There are some languages, like Java, which are memory-safe, meaning that access to deleted objects or out-of-bounds access is not possible. <br>
Instead of having to manually manage memory (like in C/C++), a so called "garbage collector" can be implemented which manages memory automatically at runtime. Here the "reachability" of addresses in computed and a set of unreachable addresses is created, which can be considered as "dead" and therefore safely be freed. While this concept improves memory safety, it might impact performance, while also not preventing certain issues like the before mentioned memory leaks (dead addresses not being freed). <br>
In languages like C/C++, manual management is possible, where memory is allocated via ‘malloc()’ and later deallocated via ‘free()’. This gives the programer more control and freedom over the memory, but e.g. deciding when to free memory is difficult. This is therefore prone to errors, which might result in lacking memory safety.

---

