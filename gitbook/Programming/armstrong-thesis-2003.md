# Making reliable distributed systems in the presence of software errors

https://erlang.org/download/armstrong_thesis_2003.pdf

**Definition of an architecture**

- A problem domain
  > What type of problem is the architecture designed to solve? Sodware architectures are not general purpose but are designed for solving speciﬁc problems.
- A philosophy
  > What is the reasoning behind the method of sodware construction? What are the central ideas in the architecture?
- A set of construction guidelines
  > How do we program a system? We need an explicit set of construction guidelines. Our systems will be written and maintained by teams of programmers—it is important that all the programmers, and system designers, understand the system architecture, and its underlying philosophy.
- A set of pre-defined components
  > Design by “choosing from a set of pre-deﬁned components” is far easier than “design from scratch.”
- A way of describing things
  > How can we describe the interfaces to a component? How can we describe a communication protocol between two components in our system? How can we describe the static and dynamic structure of our system? To answer these questions we will introduce a number of dicerent, and specialised notations. Some for describing program APIs, and other notations for describing protocols, and system structure.
- A way of configuring things
  > How can we start, stop, and conﬁgure, the system. How can we re-conﬁgure the system while it is in operation?

**Problem domain(for Erlang, for telecommunication)**

Requirements:

1. The system must be able to handle very large numbers of concurrent activities.
2. Actions must be performed at a certain point in time or within a certain time.
3. Systems may be distributed over several computers.
4. The system is used to control hardware.
5. The sodware systems are very large.
6. The system exhibits complex functionality such as, feature interaction.
7. The systems should be in continuous operation for many years.
8. Sodware maintenance (reconﬁguration, etc) should be performed without stopping the system.
9. There are stringent quality, and reliability requirements.
10. Fault tolerance both to hardware failures, and sodware errors, must be provided.

Motivate to:

- Concurrency
- Soft real-time
- Distributed
- Hardware interaction
- Large sodware systems
- Complex functionality
- Continuous operation
- Quality requirements
- Fault tolerance

**Philosophy**

To make a fault-tolerant software system which behaves reasonably in the presence of sodware errors we proceed as follows:

1. We organise the sodware into a hierarchy of tasks that the system has to perform. Each task corresponds to the achievement of a number of goals. The sodware for a given task has to try and achieve the goals associated with the task.

   Tasks are ordered by complexity. The top level task is the most complex, when all the goals in the top level task can be achieved then the system should function perfectly. Lower level tasks should still allow the system to function in an acceptable manner, though it may ocer a reduced level of service.

   The goals of a lower level task should be easier to achieve than the goals of a higher level task in the system.
2. We try to perform the top level task.
3. If an error is detected when trying to achieve a goal, we make an attempt to correct the error. If we cannot correct the error we immediately abort the current task and start performing a simpler task.

**Concurrency oriented programming**

The vast majority of programming languages are essentially sequential; any concurrency in the language is provided by the underlying operating system, and not by the programming language.

In this thesis I present a view of the world where concurrency is provided by the programming language, and not by the underlying operating system. Languages which have good support for concurrency I call Concurrency Oriented Languages.

**Programming by observing the real world**

We often want to write programs that model the world or interact with the world.

1. We identify all the truly concurrent activities in our real world activity.
2. We identify all message channels between the concurrent activities.
3. We write down all the messages which can ﬂow on the dicerent message channels.

The structure of the program should exactly follow the structure of the problem. Each real world concurrent activity should be mapped onto exactly one concurrent process in our programming language. If there is a 1:1 mapping of the problem onto the program we say that the program is isomorphic to the problem.

It is extremely important that the mapping is exactly 1:1. The reason for this is that it minimizes the conceptual gap between the problem and the solution. If this mapping is not 1:1 the program will quickly degenerate, and become diecult to understand. This degeneration is oden observed when non-CO languages are used to solve concurrent problems. Oden the only way to get the program to work is to force several independent activities to be controlled by the same language thread or process. This leads to a inevitable loss of clarity, and makes the programs subject to complex and irreproducible interference errors.

**Characteristics of a COPL**

1. COPLs must support processes. A process can be thought of as a self-contained virtual machine.
2. Several processes operating on the same machine must be strongly isolated. A fault in one processe should not adversely ecect another process, unless such interaction is explicitly programmed.
3. Each process must be identiﬁed by a unique unforgeable identiﬁer. We will call this the Pid of the process.
4. There should be no shared state between processes. Processes interact by sending messages. If you know the Pid of a process then you can send a message to the process.
5. Message passing is assumed to be unreliable with no guarantee of delivery.
6. It should be possible for one process to detect failure in another process. We should also know the reason for failure.

Note that COPLs must provide true concurrency, thus objects represented as processes are truly concurrent, and messages between processes are true asynchronous messages, unlike the disguised remote procedure calls found in many object-oriented languages.

Note also that the reason for failure may not always be correct. For example, in a distributed system, we might receive a message informing us that a process has died, when in fact a network error has occurred.

**Process isolation**

The notion of isolation is central to understanding COP, and to the construction of fault-tolerant sodware. Two processes operating on the same machine must be as independent as if they ran on physically separated machines.

Indeed the ideal architecture to run a CO program on would be a machine which assigned one new physical processor per sodware process. Until this ideal is reached we will have to live with the fact that multiple processes will run on the same machine. We should still think of them as if they ran on physically separated machine.

Isolation has several consequences:

1. Processes have “share nothing” semantics. This is obvious since they are imagined to run on physically separated machines.
2. Message passing is the only way to pass data between processes. Again since nothing is shared this is the only means possible to exchange data.
3. Isolation implies that message passing is asynchronous. If process communication is synchronous then a sodware error in the receiver of a message could indeﬁnitely block the sender of the message destroying the property of isolation.
4. Since nothing is shared, everything necessary to perform a distributed computation must be copied. Since nothing is shared, and the only way to communicate between processes is by message passing, then we will never know if our messages arrive (remember we said that message passing is inherently unreliable.) The only way to know if a message has been correctly sent is to send a conﬁrmation message back.

**Names of processes**

We require that the names of processes are unforgeable. We will assume that processes know their own names, and that processes which create other processes know the names of the processes which they have created. In other words, a parent process knows the names of its children.

If we know the name of a process, we can send a message to that process. If we do not know the name of a process we cannot interact with it in any way, thus the system is secure. Once the names of processes become widely know the system becomes less secure. We call the process of revealing names to other processes in a controlled manner the _name distribution problem_ — the key to security lies in the name distribution problem. When we reveal a Pid to another process we will say that we have published the name of the process. If a name is never published there are no security problems.

Thus knowing the name of a process is the key element of security. Since names are unforgeable the system is secure only if we can limit the knowledge of the names of the processes to trusted processes.

**Message passing**

rules:

1. Message passing is assumed to be atomic which means that a message is either delivered in its entirety or not at all.
2. Message passing between a pair of processes is assumed to be ordered meaning that if a sequence of messages is sent and received between any pair of processes then the messages will be received in the same order they were sent.
3. Messages should not contain pointers to data structures contained within processes—they should only contain constants and/or Pids.

Note that point two is a design decision, and does not reﬂect any underlying semantics in the network used to transmit messages. The underlying network might reorder the messages, but between any pair of processes these messages can be bucered, and re-assembled into the correct order before delivery. This assumption makes programming message passing applications much easier than if we had to always allow for out of order messages.

We say that such message passing has send and pray semantics. We send the message and pray that it arrives. Conﬁrmation that a message has arrived can be achieved by returning a conﬁrmation message.

Message passing is also used for synchronisation. Suppose we wish to synchronise two processes A, and B. If A sends a message to B then B can only receive this message at some point in time ader A has sent the message. This is known as casual ordering in distributed systems theory.

**Protocols**

Isolation of components, message passing between components, sufficient for protecting a system from the consequences of a software error, but not sufficient to specify the behaviour of a system, no in the event of some kind of failure to determinen which component has failed.










