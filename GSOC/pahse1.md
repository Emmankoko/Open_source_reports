### ALTQ REFACTORING

- All work were integrated into [gsoc_altq_refactoring branch](https://github.com/Emmankoko/src/tree/gsoc_altq_refactoring/sys/altq)
#### Addressing security defects and dead condtitions

* src/sys/sltq/altq_classq.h file in Netbsd contains class queue definitions extraced from src/sys/altq/altq_rmclass.h. 

* [_getq_random](https://github.com/Emmankoko/src/blob/trunk/sys/altq/altq_classq.h#L145) which randomly select a packet in the queue uses the [random()](https://github.com/Emmankoko/src/blob/trunk/sys/altq/altq_classq.h#L158) function which is easy to break because of its weak congruential algorithm. this function was replaced by Netbsd's cprng random generator with improved security and speed in the number generation which can be found in [this commit](https://github.com/Emmankoko/src/commit/11c7cdce6c8bc2aadcf93565ceef52d9748dc53a)

#### Handling duplications 

* in ALTQ, routines that are duplicated for use in several other routines violating the DRY principle have been handled. this avoids having several points of changes if there be need for it in the future. these are commits that reveal such works
    - in sys/altq/altq_afmap.c, several function perform address family lookup to get the current address family head pointer.
    the refactor was integrated in three buildups. the address lookup was put into a afm_lookup function that returns the head after the current head found and used throughout the code.
        - [general duplicate handling](https://github.com/Emmankoko/src/commit/9861e98df652ea7964e8df95be9b6d699469f477)
        - [pass pointers by double pointers/reference](https://github.com/Emmankoko/src/commit/2ce386288d74028fcd59f21764d01b73f3b61a60)
        - [avoid unecessay use of double pointers](https://github.com/Emmankoko/src/commit/d86e560d59b9ade5365555d4eacd7b064b3c6bde)
    - end product of sys/altq/altq_afmap.c = [afmap](https://github.com/Emmankoko/src/blob/gsoc_altq_refactoring/sys/altq/altq_afmap.c)

        - in sys/altq/altq_wfq.c, wfq hashing code duplications were present for sorce and destination address types. 
        this was simply handled with [this commit](https://github.com/Emmankoko/src/commit/1c282f4dae07b652d1de4fefbf60d5ff3a965d30)

        - in sys/altq/altq_subr.c,
            - " #ifdef INET6 .... #endif" and "#ifdef INET .. #endif " blocks for ipv6 and ipv4 family respectively were duplicated for the internal function prototypes. duplications were handled with [this commit](https://github.com/Emmankoko/src/commit/6e2d68da4b5ba54c6104ba11c735e3cad30b74a9)

            - packet assertions code in writing and reading dsfield was duplicated. this was handled with [this commit](https://github.com/Emmankoko/src/commit/90c88d03fdce4d860c25091fd74e9019a99008e0)

        - in sys/altq/altq_jobs.c, tslist drop and dequeue contains a duplicated revomval routine that removed either fom the front or the rear. this duplication was handled with [this commit](https://github.com/Emmankoko/src/commit/2a6785693000e318ce4861db4d28b393bc1af610)


#### single responsibility / fundtion cleanup

* several functions in ALTQ contained very large blcoks and did not obey the single responsibility principle. they were doing more than one thing in the function. 
Therefore, nested routines that were implemented in the functions were also put into their own function blocks and called from their respective functions of need. 


#### fix compile time errors

* in sys/altq/altq_jobs.c, previous patches left it with errors that were never run and caught. when detaching from an altq interface, the wrong altq jobs interface was used. this was corrected with [this commit](https://github.com/Emmankoko/src/commit/5dbb67c33f4bbe4ad46a078b0ea3b6270eaaa3b4)

#### goto statements removed

* goto statements in ALTQ wouldn't be the right choise for condtional execution in the ALTQ system. it messes up the execution flow. goto labels were transformed into their function calls and correct use of while and do while loops which guves the layer a better execution flow as compare to using goto. 
look at [this commit](https://github.com/Emmankoko/src/commit/e5628c36ecb25bcbbf373532b80adcd6a54596f1) and upwards for the next 5 commits.

#### remove parenthesis from return statements

* according to the netbsd style of coding, there should be no parenthesis on return statements. all sys/altq/*.c files contained return statements that were enclodsed with return statements. all these were removed from the altq codebase with exceptions in statements that evaluated not straightforward expressions, or pointer dereferencing.

#### FREEBSD blcoks were removed

* freebsd blocks that do not compile were removed from ALTQ.
see [this commit](https://github.com/Emmankoko/src/commit/570a31a8b74881725750f18efd0f0396d36ea863) and [this commit](https://github.com/Emmankoko/src/commit/b329f56ea72bfa29204f9538d132eb9d8e26154a)


* PS :
 NetBSD was compiled and run by checking for the altqstat for every commit made. propper testing will be carried on when the whole altq codebase refactoring is completed for every queueing schemme.