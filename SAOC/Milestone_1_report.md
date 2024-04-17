### EMMANUEL DANSO NYARKO
#### C++ STL BINDINGS

#### **RESEARCHING INTO THE CURRENT STATE OF C++ INTEROP IN D**

* WEEK 1 - (Thorough overview of the compiler and linker) 
    * I spent time walking through [this compiler book(pdf)](https://www.bing.com/ck/a?!&&p=2631ff323ce365a1JmltdHM9MTY5NzMyODAwMCZpZ3VpZD0zODQ1MDE2My00N2ZjLTYxZjMtMGVhYi0xMWE3NDY2ZjYwZjYmaW5zaWQ9NTI0NQ&ptn=3&hsh=3&fclid=38450163-47fc-61f3-0eab-11a7466f60f6&psq=introduction+to+compilers+and+language+design&u=a1aHR0cHM6Ly93d3czLm5kLmVkdS9-ZHRoYWluL2NvbXBpbGVyYm9vay9jb21waWxlcmJvb2sucGRm&ntb=1) to understand the compiler as a tool and how well I can work with it. more time was spent on the semantic analysis, memory organizations, and also Assembly generations phase of the compiler as they are key components that enhances the interoperability of various systems. These understandings have helped so far
    
        * Semantic analysis which primarly involved type checking and name(symbol) resolution whose compatibility factor is a major component in interoperation. 
        * Memory organizations which expands on OS architecturs and heap and stack management. more focus was geared towards the stack management which was revealed to me how the order and details of elements in a stack frame differ by OS and CPU architecturs and also how the agreement between how these elements are passed to registers during assembly generations(calling conventions) between different systems ensure interoperability of code.  
        * the symbol table is a helper to ensuring correctness of external linking particularly for different systems as such. the "nm" tool was primarly used for linkage checks. [forum update](https://forum.dlang.org/post/pgkuiyogjvzgieqmseac@forum.dlang.org)
    

      

* Week 2 - A(Continuation with linker and Binary Interfaces).
    *   shortly revised and tested the symbol relocation into final executables in D, after symbol resolution during linking,  that are loaded into memory for execution.
    * reviewed through the binary interface as an object code interface that mostly interface between user codes and libraries. Binary interfaces are not standard as they differ by compilers, OS architecture, and also programming interface dependent. assimilation of the full ABI aids in efficient mixing of object files 

* Week 2 - B - Parallel learning into the C++ Itanium ABi using [C++ Itanium reference](https://www.bing.com/ck/a?!&&p=856a41f595c0d1ddJmltdHM9MTY5NzMyODAwMCZpZ3VpZD0zODQ1MDE2My00N2ZjLTYxZjMtMGVhYi0xMWE3NDY2ZjYwZjYmaW5zaWQ9NTE4Nw&ptn=3&hsh=3&fclid=38450163-47fc-61f3-0eab-11a7466f60f6&psq=itanium+c%2b%2b+abi+pdf&u=a1aHR0cHM6Ly9pdGFuaXVtLWN4eC1hYmkuZ2l0aHViLmlvL2N4eC1hYmkvYWJpLmh0bWw&ntb=1) and the current state of C++ interop in D using the dlang reference page.
    * I walk through the C++ Itanium ABI reference occasinally but hasn't been fully consumed as it serves as a reference for special cases where I have to check for binary compatibility issues. 

    * walking through the C++ interop, I seperately perform specialized test for interoperations that cover very important areas that play key roles in the C++ STL interop. areas such as ABI testing, calling conventions, sizes, type attributes, virtual tables, namespaces, classes, structs, templates, exceptions, references, pointers, conditional compilations, mangling, and attributes. [forum update](https://forum.dlang.org/post/hhoqrvuwplpjoljmjsnw@forum.dlang.org)

* Week 3 - (continuation of general C++ interop discoveries  and Runtime explorations) 
    * Watched two Dconf talks on C++ interop by [Mathias Lang](https://www.youtube.com/watch?v=mI6-PmZy-u0&t=72s&pp=ygUSbWF0aGlhcyBsYW5nIGRjb25m) on the state of C++ interop and [Razvan](https://www.youtube.com/watch?v=j-Ws4VjyUWg&t=52s&pp=ygUTemVybyBvdmVyaGVhZCBkY29uZg%3D%3D) who lightened on zero overhead between D and C++ standard Library. These are talks that have really helped me by giving a well defined approach to attacking this work. Mathias provides additions to several unexplained concepts like Operator overloads which is believed to be not interoperable between D and C++. Razvan also gives his talks on several issues currently encountered like mangling issues due to some parameter passed to functions, exceptions, and also a call overhead which should be taken care of by inlining. 
    *  Alongside, I was researching into druntime to cooperate with My mentor as he had suggested we move C++ STL interop from druntime. with my findings, I realized a project such as C++ interop shouldn't be in a runtime environment as they become very restricted to runtime libraries characteristics being tied to a certain runtime as supported by the D compiler. since we want to bind to different versions of C++ interop from different compilers, the template library was agreed to be moved into its own project.  
        * currently, the project has started and is under development. [dlang/stdcpp](https://github.com/dlang/stdcpp)

        [forum update](https://forum.dlang.org/post/kvfdispkgwpieicesfsk@forum.dlang.org)

* Week 4 - Run tests through std::vector and started working to address existing problems.
    * Currently, we have [vector](https://github.com/dlang/dmd/blob/master/druntime/src/core/stdcpp/vector.d) in druntime. bindings have been written for only Microsoft Runtime. 
    * After being moved into a new seperate workspace, work has began towards making std vector function on Gcc runtime. currently, member functions size(), capacity(), reserve(), resize() have been implemented. vector constructors are currently being implemented and hope to be completed soon which can ace the completion for Gcc runtime. [checkout this repository](https://github.com/Emmankoko/stdcpp) to see work currently going on.
    * [forum update](https://forum.dlang.org/thread/wtuyusdifxvzwndlzueg@forum.dlang.org)
    
* PS: Extra checkouts
    * the C++ STL is a "header only" template libray which contains all definitions of the class template as required by the C++ compilers. this header also reveals the internal organization such as size and layout.
    * An explicit template instantiation file with instances of all supported types will be linked against so references to symbols can be bound.
    * the D interface being implemented should also reveal the internal organization such as size and layout because the instantiated object(container) in D is going to depend on D's interface being implemented and so it should match the semantics of C++ using D rules(very important).



