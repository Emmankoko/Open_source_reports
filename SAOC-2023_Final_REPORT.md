# EMMANUEL DANSO NYARKO 
## SAOC 2023 FINAL MILESTONE REPORT
### IMPROVING C++ INTEROPERABILITY IN D

- STD::SET ON RUNTIME GCC ON UBUNTU
    - std::set is an associative container that prioritizes uniqueness
    - std::set is primarily implemented as an Red black tree.
    - it is very important that we match the exact orientation of the tree on D side so we do not encounter illegal access to memory when copying and swapping our containers. 
    - initially, I implemented a customized Red black tree to correctly fit a 48 byte size which is the size of the tree on linux.
    - so I was free to bind to available functions without need to worry about illegal memeory access.

    - Issues encountered
        - About 50 percent of the functions were not available for binding

    - solution to address aforementioned issue
        - I had to reimplement on the D side the C++ functions that were not available for binding

    - I worked on reimplementing the Red black tree on the D side for "CppRuntime_Gcc" so I could call some of the tree functions
    from the std::set functions. 
    - I have been able to get several functions from the std::set library working for D
    - see below for link to work


- STD::SET on MICROSOFT RUNTIME
    - microsoft implements std::set as a Red Black tree that fits into a size of 16 bytes as a function of a pointer to the tree node and a track of its size
    - as usual, it is very key that we match the size of the container on the D side to avoid illegal memory access during copying, swapping, and even overloading the "=" operator in opAssign functions.
    - here, it is not a considerable approach to customize a 16 byte size fields in our D container for binding.
    - Reason
        - only three functions from std::set are available to be picked by the linker after instantiations
    - approach
        - I had to reimplement on the D side the Red black tree very well so we can use them in our std::set container
        - some of the symbols of the Red Black tree were emmitted and could be picked by the linker so I was able to use them as well in assisting me implement most complex functions.
        - most of the functions are tested for their availability and emptiness without passing our container elements around yet. This is a continuous step as I am very close to making our Rbtree very efficient to be able to pass elements around
    - simple difference
        - the microsoft STL std::set header file does not provide definition for most of the functions including functions such as size, empty, clear, contains, count, insert etc. [can view here](https://github.com/microsoft/STL/blob/main/stl/inc/set)
    - simple approach
        - since all the functions are implemented in the Red Black Tree, I provided our own definition of the function in the SET struct 
        - these defintion acts as wrappers from which we call the Red Black Tree functions from and it works as far as tested.
        - see below link to work

- PS:
- PLANS FOR TREE FOR ASSOCIATIVE CONTAINERS
    - The associative containers are heavily dependent on the Red Black Tree
    - so currently, my primary focus is making the tree work very well when called from the main functions.
    - currently, I have them privately in the set.d module and still making progress with them.
    - I perform a slow testing strategy per stdcpp.set function for our tree functions
    - I will be moving it into its own stdcpp._xtree.d module to be imported in the set.d module and the other associative containers.
    - getting the Tree work very well for all our three runtimes ensures that our associative containers also work well.
    - On macOS, both 11 and 13 tested, std::set instatiations generates no symbols as all functions are inlined.
    - After I'm done with getting the tree work on windows, I will focus on getting the tree fully implemented for macOS runtime as well.
    - so I can call them the stdcpp.set functions on clang runtime.



- SAOC SUMMARY
    - I started SAOC researching into the current state of C++ interop in D exploring all the possibilities and impossibilities as at now and also learning a lot of approaches from Dconf talks and directly from my mentor.
    - after few weeks of coming to an "okay" ground, we moved core.stdcpp package from druntime as druntime tranfers a copy of our programs to us in an already compiled state during distribution time. when this happens, the already compiled program will be compiled for the compiler runtime as targatted by D and might affect any ABI/implementation differences between different compilers on a particular runtime.
    - we already had vector.d written for only microsoft runtime. I worked on making vector usable on Linux and macOS. all these will be revisited and tested for a larger runtime version range as we started with a smaller target(macOS-11, windows 2019 and ubuntu 20.04).
    - I continued to test for the new ABI implementation for std::string on GNU GCC to improve it. in the improvement phase, i realized we had our symbols emmitting to druntime so we couldn't "alias std_string = basic_string!char". since we have moved the project from druntime, we are now free to use that. 
    - I continued to work on getting std::list in D. I ensured that it worked on microsoft, linux, and macOS.
    - currently, I have been able to get std::set work in D on linux, and also testing for them on microsoft. Hope to complete and get them started on macOS CppRuntime_Clang.
    - workflow approach as discussed with my mentor
        - I have each container per compiler per runtime on seperates github branches.
        - so I can get focused attention on each compiler target on every runtime
        - I keep working on seperate branches and getting reviews from my mentor
        - then moved to the main branch for a pull request that will be broken down into smaller chunks

    - EXTRAS
        - After I am done with std::set, I will be working to make map, unordered map usable from D.
        - I will also run complete tests throughout the whole stdcpp package to make sure we release a quality project as i realized some few modules such as allocator.d needs to be updated. I will be looking into more of them.  
        - I have started making room to prepare a very communicative documentation for this projects
        - I will be working on writing a tool to generate our instantiations so we can use in our projects as well, as this is already discussed with my mentor Mathias Lang
        - I will be running address sanitizations on each struct to check for memory leaks to deallocate heap allocations to make our API safe to use
        - I will also improve the "interface to c++" documentation on the dlang reference page as there have been a lot of improvements to it.
        - weekly meetings with my mentor still continues.


- TAKEAWAYS
    - I joined SAOC as a very young programmer very hungry for growth and to take on challenges.
    - I have been particularly excited about the mentorship provided by the super talented programmers in the D community
    - I have learnt so much from my mentor in all aspects of programming and his experiences as well.
    - personally, my experiences have been tokens of growth and maturity to my life all around.
    - A Few takeaways to share
        - SAOC has significantly increased my hunger for knowledge as I spent most of time reading, constantly brainstorming, and experinmeting possible solutions.
        - SAOC has taught me how to appropiately use my time as I always had my time scheduled and finding to fix current issues in my milestones. 
    - I recommend SAOC to everyone who wants to contribute to something meaningful


- FUTURE GOALS
    - I want to stay with D and be an active contributor to D because I owe my growth as programmer to D and SAOC and this is how I want to give back.
    - I read on the dlang forum that we are still not able to catch exceptions in D on windows and I have decided to add that as a task to research through and work on it as well
    - I want to also play a role in the longevity of SAOC by contributing either financially,possible proposals and also provide mentorships to projects within the scope of my knowledge and experiences later in the future. 




- dlang forum updates
    - [week 14](https://forum.dlang.org/post/lxbkkerhaucjkfeeetiv@forum.dlang.org)
    - [week 15](https://forum.dlang.org/post/wqisijqapervzzmxzndz@forum.dlang.org)
    - [week 16](https://forum.dlang.org/post/woscjimjsgikbylskpbh@forum.dlang.org)
    - [week 17](https://forum.dlang.org/post/zwspowafbtgxxzqqoqry@forum.dlang.org)

- std::set on linux
    - [set1 branch for linux runtime](https://github.com/Emmankoko/stdcpp/tree/Set1)
    - [current test level for set on linux](https://github.com/Emmankoko/stdcpp/blob/Set1/source/stdcpp/test/set.d)
    - [beginning module for std::set on linux](https://github.com/Emmankoko/stdcpp/blob/Set1/source/stdcpp/set.d)

- std::set on microsoft
    - [set rb_tree starting point build up for microsoft runtime](https://github.com/Emmankoko/stdcpp/blob/Set2_W/source/stdcpp/set.d#L328)
    - [current tests state for set on microsoft runtime](https://github.com/Emmankoko/stdcpp/blob/Set2_W/source/stdcpp/test/set.d)
    - NB: Rb tree will be moved to its own stdcpp._Rbtree.d module