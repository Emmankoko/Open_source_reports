<h3 style="text-align: center;">GSOC-24 WORK SUBMISSION </h3>
<h3 style="text-align: center;"> NetBSD ALTQ REFACTORING</h3>


## Overivew

-   [The NetBSD](https://www.netbsd.org) operating system is a free, fast, secure, and highly portable Unix-like Open Source operating system. It is available for a wide range of platforms, from large-scale servers and powerful desktop systems to handheld and embedded devices. one of its goals is to improve [ALTQ](https://en.wikipedia.org/wiki/ALTQ), for improved quality of serivce, code quality and also to be integrated in the [NPF](https://en.wikipedia.org/wiki/NPF_(firewall)) packet filter which offers a lot more as a packet filter. [problem statement](https://wiki.netbsd.org/projects/project/altq/). This is going to improve the quality of services that are running on NetBSD due to its sensitivity to packets losses and overfloats.

[Githug repository](https://github.com/Emmankoko/altq_refactoring_gsoc) <br>
[refactoring branch](https://github.com/Emmankoko/altq_refactoring_gsoc/tree/gsoc_altq_refactoring) <br>
[codel branch](https://github.com/Emmankoko/altq_refactoring_gsoc/tree/altq_codel)

## Work done

<h3 style="text-align: center;"> PHASE 1</h3>

I spent my initial phase intoducing good design patters, fixing compile time errors, removing dead portions, improving execution performance, and introducing security patches in the kernel ALTQ codebase to make it more maintainable. <br>
All work of this phase were integrated into [gsoc_altq_refactoring branch](https://github.com/Emmankoko/src/tree/gsoc_altq_refactoring/sys/altq)
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


<h3 style="text-align: center;"> PHASE 2</h3>

Spent the second phase integrating [Codel](https://en.wikipedia.org/wiki/CoDel) active queue management in NetBSD. Codel is an active queue management used in network routing primarily designed to overcome bufferbloat in networking hardwares. it was to manage queues under control of the minimum delays experiencedby the packets. A long lasting problem faced by the internet world of computer science
is the bufferbloat problems that has pushed the need for implementing new Active Queue Management. CoDel was proposed to be a solution not to completely swallow the issues with our buffer when arriving packets fill it up but at least improve the quality of our internet service. <br>

all work of this phase is integrated in the [altq_codel](https://github.com/Emmankoko/altq_refactoring_gsoc/tree/altq_codel/sys/altq) branch.

#### KERNEL WORK

- imported the direct CoDel implementation which is open sourced into [sys/altq/altq_codel.c](https://github.com/Emmankoko/altq_refactoring_gsoc/blob/altq_codel/sys/altq/altq_codel.c) file. added the corresponding header file to use both external and internal declarations.

- refactored the code to use NetBSD components, tools, functions, types etc that semantically agree with the previously implemented scheme.

- in the [codel header](https://github.com/Emmankoko/altq_refactoring_gsoc/blob/altq_codel/sys/altq/altq_codel.h) file, I included IOCTL calls for use from userland and also removed userland useful structs from KERNEL declaration blocks.

- implemented device interface(open, close, and ioctl functions) in the implementation API as the driect kernel-userland communication to configure parameters, fetch statistics, attach and detach to interface, etc.

- introduced CoDel flags to be used on other schedulars and also in the userland.

#### USERLAND WORK

- implemented a [codel parser](https://github.com/Emmankoko/altq_refactoring_gsoc/commit/b2ca2b892cdc729c9b6290cfa6b314a3aca1d5b3) to provide semantic anaysis on codel commands when configuring CoDel.

- implemented an interface parser, and its related interface routines for queue operations in [altq_qop](https://github.com/Emmankoko/altq_refactoring_gsoc/blob/altq_codel/usr.sbin/altq/libaltq/qop_codel.c) file. this does a direct interaction with the kernel, transfers data between the kernel using ioctl calls,  and acts as a server for an inter-process communication for client processes to get network statistics as interacted from the kernel.

- implemented a [codel_stat_loop](https://github.com/Emmankoko/altq_refactoring_gsoc/blob/altq_codel/usr.sbin/altq/altqstat/qdisc_codel.c#L53) function which is a non-terminating loop which displays the current queuing statistics for network performance analysis. this acts a client process which prints information from the quip server described above.

- ALTQ client-server interprocess communication scheme uses the the Unix doman socket where the server creates a file path, binds it address to the file path, and the client connects to the created socket.

- Added [codel qdisc structure ](https://github.com/Emmankoko/altq_refactoring_gsoc/blob/altq_codel/usr.sbin/altq/altqstat/qdisc_conf.c#L60) to the active list of queue disciplines in the general queue discipline configurations.

- NetBSD kernel was then compiled with ALTQ_CODEL, and built a distribution including the new userland codel codes.

#### INITIAL TESTING

- /etc/altq.conf file was set up with codel configured on the network interface.
- the interface command sets up a discipline on an interface

- codel default parameters was also set and the altq statistics was checked.
- the codel keyword sets the default parameters which is the target, interval, ecn.

### SIMPLE DEMO

In /etc/altq.conf file <br>
Interface wm0 bandwidth 5M codel <br>
codel 5 15 0 <br>
codel 5 15 0 means set a target of 5 milli seconds, 15 milliseconds and does not enable ecn notification on packets.


altqstat output

---------------
altqstat: codel on interface wm0 <br>
target: 5 [ms] , interval: 15 [ms] <br>
qlen: 0, qlimit: 60 , maxpackets: 256 pkts <br>
xmit: 0 pkts, drops: 0 pkts <br>
throughput : 53 kbps

--------------
this loops until the process is terminated by user. this statistics may change in respond to services using your network and how the user admin configures it.

NB: when no parameters are set, codel uses a default target of 5 milliseconds and interval of 100 milliseconds.

#### ANALYSIS

[cablelabs](https://www.cablelabs.com) did a preliminary study of CoDel AQM in a DOCSIS network assessing the Quality of service related to a Simulated Cable Modem. Codel indeeds proves to mitigate an appreciable portion of the bufferbloat issue.  Below is a simple analysis to demonstrate CoDel improvements to the field of network quality of service.

<img src="./Screenshot 2024-08-23 at 5.28.42 PM.png">

Queuing Delay of Simulated Cable Modem with Bufferbloat

This shows the downside of overbuffering. A single TCP session will aim to keep the buffer as full as it can, causing unacceptable latency for competing traffic. Note here that the buffer is set to a fixed number of packets, which results in a maximum buffering latency of 250 ms during the ~5.5 seconds of the initial 20 Mbps burst, then jumps to 1 second from that point forward.

<img src="./Screenshot 2024-08-23 at 5.28.13 PM.png">

Queuing Delay of Simulated Cable Modem with CoDel AQM

We see that CoDel allows the single TCP and the five TCPs to quickly ramp up to 20 Mbps (due to the large buffer), but that CoDel reacts to the large buffer and forces the TCPs to back off to the point where they have minimal queue sitting in the buffer, but still keep the channel occupied.

When the buffer is configured for 130 or 170 packets, this issue goes away. Also of note is that, while CoDel generally keeps the queuing latency below the 5 ms target, at the TCP start and at the rate transition from Peak Traffic Rate to Max Sustained Rate, CoDel allows a spike in queuing latency that takes some time (~2.5 seconds) to subside.

#### TAKE-AWAYS
CoDel has been a major improvements in Active Queue Managements helping to reduce packet loss and other related downfalls in the quality of service domain.

In taking this work up, I hope to propse future improvements to these studies that eventually swallows bufferbloats issues relating to underbuffering oroverbuffering.

#### TODO AFTER GSoC:

- properly set up ATF test which is the network testing kit in netbsd on larger systems, more services, on different networks.

- integrtate ALTQ in the NPF packet filter and dissociate the PF packet filter from ALTQ.

- NPF is a packet filter with stateful inspection, Network Address Translation, IP sets, etc. [find out here](http://rmind.github.io/npf/). It was designed purposely for the NETBSD operating system and offers more as compared to the traditional packet filters available.

Special thanks to my mentors who availed themselves to help me overcome all roadblocks that we encountered.
