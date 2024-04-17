### EMMNAUEL DANSO  NYARKO
#### SAOC 2023 C++ STL BINDINGS

#### LIST FOR CLANG ON MACOS
#### LIST FOR MSVC ON WINDOWS
#### SEED LEVEL FOR SET ON GCC ON UBUNTU

 - BINDING STD::LIST FOR CLANG ON MACOS
    * clang implements list as a 16 byte template structure that has nodes connected as a linked list.
    * It is a duty as usual to match this size and layouts by owning our fields so we do not get segmentation faults
    by trying to access a memory region not owned by our D extern(C++) struct.
    * then I was free to bind global text symbols picked up by the linker to their definitions


    * The three sections in the memory segments being .data, .bss, .text sections emits symbols
    in the symbol table that define the scope of access and availability of them.
    * on the API level, clang on MacOs emits some of the symbols as local text symbols that have an access scope level only during the lifetime of the program and have no life externally(opposite to global text symbols).
    * these symbols are not picked up by the linker during linking and so they do not become available to bind.
    * these functions were re-implemented on macOS and have been tested and will still be tested to ensure
    strenght and efficiency of them.
    * see below for link to work done

- BINDING STD::LIST FOR MSVC ON WINDOWS
    * getting std::list work effectivelyon windows required an extra step of research. 
    * the visual C++ compiler has a very tricky name mangling algorithm that requires a high level attention to detail. 
    * Fortunately, we did not have some symbols local so we could get the linker pick them up.
    * I took my time through [visual c++ name mangling](https://en.wikiversity.org/wiki/Visual_C%2B%2B_name_mangling) that really helped me in my debugging
    * I also faced a little problem linking to our object file in a directory that does not contain our object file
    * I managed to solve that issue with a minor fix as [this](https://github.com/dlang/stdcpp/commit/9c19cfa12f3c87e31e2dbb5d8cbd13fe81a1e43a)
    * I missed some level of attention to detials but these experiences are appreciated much as they played a part in getting our bindings work properly.


- SEED LEVEL BINDINGS FOR STD::SET WITH GCC TARGET ON LINUX
    * std::set is an associative container that contains a sorted set of unique objects of type Key which(sorting) is done using the
    key comparison function Compare.
    * Sets are usually implemented as Red–black trees
    * sets is a strong advocate of uniqueness
    * currently at the seed level, I have taken measures to
        * match the fields and size by getting a well implememted red black tree working that produces a size of 48 bytes
        with gcc as initial implementations were emitting segmentation faults but has been worked on well and we are now safe.
        * fit our red-black tree into our fields.
        * implememt utility functions that will play a role in mangling and linking. 
        * set up a simple unittest of simple constructor and few functions to be checked and function

    * I am currently getting myself very much fused with the whole orientation of the template library to easily accomodate 
        for the need for re-implementations(by mangling substitutions issues that will be encountered, or local text symbols that will be needed in our struct).
    * see below for link to seed level work done.



- [week 9 forum update](https://forum.dlang.org/post/oicfjuonppswndlrnnwv@forum.dlang.org)
- [week 10 forum update](https://forum.dlang.org/post/iztulkvdikiynipjlmlx@forum.dlang.org)
- [week 11 forum update](https://forum.dlang.org/post/mkrgdcjgjacyzmwqkrmt@forum.dlang.org)
- [week 12 forum update](https://forum.dlang.org/post/mcvdvaenusqonrofokcq@forum.dlang.org)
- [week 13 forum update](https://forum.dlang.org/post/aeihhjffusmnevwurmah@forum.dlang.org)

- list on clang(macos)
    - [t1 branch for debugging and testing list for clang on macOS](https://github.com/Emmankoko/stdcpp/tree/t1)
    - [list implementation clang target on macOS on](https://github.com/Emmankoko/stdcpp/blob/t1/source/stdcpp/list.d)
    - [list basic testings for clang on macOS](https://github.com/Emmankoko/stdcpp/blob/t1/source/stdcpp/test/list.d)

- list on windows(msvc)
    - [ms-extern branch for list on microsoft built on gcc and clang bindings to form a complete module](https://github.com/Emmankoko/stdcpp/tree/ms-extern/source/stdcpp)
    - [list.d module to be tested on msvc](https://github.com/Emmankoko/stdcpp/blob/ms-extern/source/stdcpp/list.d)
    - [list.d test module to provide tests for list.d on windows](https://github.com/Emmankoko/stdcpp/blob/ms-extern/source/stdcpp/test/list.d)

- set
    - [set basic testings for gcc on ubuntu](https://github.com/Emmankoko/stdcpp/blob/Set1/source/stdcpp/test/set.d)
    - [basic level implementation for std::set with red-black tree implementation](https://github.com/Emmankoko/stdcpp/blob/Set1/source/stdcpp/set.d)
    - [Set1 branch for testing std::set on gcc on linux](https://github.com/Emmankoko/stdcpp/tree/Set1)

PS:
- I am working on every new module in a new branch with only one test module targetting a particular runtime. This i think will give the
modules the attention they need so we can get the best results possible on each runtime.
- I will be setting up a CI to test our complete modules on our target runtimes in a matrix 
- As learnt from a 2017 dconf talk by , I will be adding smaller implementations to reduce the overhead cost in C++ STL interop.
- I am putting things in place to make our API direct, easy to use and difficult to misuse
- also putting things in place to sanitize our address to check for memory leaks to make our API safe to use.(this will be done in retesting of all stdcpp modules as well and also when implementing the deque sequence container.)
- I am putting things in place to get the C++ container adapters(stack, queue, priority_queue) brought to D as well
- already existing modules must be revisisted for possible improvements