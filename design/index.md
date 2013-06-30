---
layout: page
title: Design Overview
class: design
---

Shadow does the following:

 * natively executes **real applications** like Tor
 * provides efficient, accurate, and **controlled** experiments
 + models network topology, latency, and bandwidth
 + models the Tor network using [Tor metrics][tormetrics]{: target="_blank"} 
 + runs without root on a single Linux box, or [in the cloud][wiki-ec2]
 + simulates multiple virtual nodes in virtual time
 + simulates network, crypto, CPU processing delays
 + much, much more!

Real Applications as Shadow Plug-ins
====================================

Shadow is a discrete-event simulator that runs **real applications** like [Tor][tor]{: target="_blank"} . Shadow links to real application software and **natively executes the application code** during simulation, providing faithful experiments and accurate results. Shadow models and runs distributed networks using these applications on a single Linux machine, easing experiment management while keeping the focus on the results.

What are Plug-ins?
------------------

Plug-ins are library shims that are linked to real applications. Shadow dynamically loads these libraries to natively execute the application code. Shadow intercepts and re-routes a selective set of system calls to enable seamless integration of an application to the simulated environment. In this way, the application may be unaware that it is running in the simulator and will function as if it was running in a standard UNIX environment.

What is Scallion?
-----------------

Scallion is a Shadow plug-in for simulating the [Tor][tor]{: target="_blank"}  anonymity network. Scallion integrates Tor into Shadow by wrapping the [Tor source code][torsource]{: target="_blank"}  with the necessary hooks that allow it to communicate with the Shadow simulator, thereby leveraging Shadow's unique functionality to allow rapid prototyping and experimentation of Tor. Scallion also contains scripts that assist in analyzing results, generating Tor topologies, and running experiments using the generated topologies.

Visit [the Scallion wiki page][wiki-scallion] for more information on the Tor plug-in and its memory requirements.


Simulation Blueprint
====================

The first step to using Shadow is to create a blueprint of an experiment. The format of the blueprint is standard XML. The XML file tells Shadow when it should create each virtual node, what software each virtual node should run. It also specifies the structure of the network topology, and network properties such as link latency, jitter, and packet loss rates. Shadow contains example XML files to help get started!

<a href="/assets/design1.png"><img title="design1" src="/assets/design1.png" alt="" width="520" /></a>

_Shadow takes a simulation blueprint as input. This XML file specifies the structure of the topology and the general flow of the experiment. Shadow's event engine initializes nodes using the blueprint, and runs events on their behalf until the simulation is complete._

Discrete Time Events
====================

Shadow creates several bootstrapping events after extracting the information from the supplied XML blueprint file. Each of these events are executed at a discrete time instant during the experiment. Each of these bootstrapping events will cause the virtual nodes to start executing the specified software, which in turn will spawn additional events for Shadow to process. Shadow tracks the time each virtual node spends processing inside the application, and delays events according to the node's configured virtual CPU speed. Events are continuously executed until the simulation end time.

Node Management
===============

Shadow runs real software, which is linked as a Shadow plug-in only once. But to run multiple virtual nodes, Shadow must keep track of that state for many nodes at once. To support multiple virtual nodes, the Shadow engine stores a copy of all the variable state that exists in each plug-in. Whenever Shadow executes an event for a virtual node's application, it first does a "context switch" by copying that application's state information from the storage area inside of Shadow to the physical memory regions where the application expects the state to exist during normal execution. With the virtual node's state in place, execution is then passed to the application. After the application finishes, Shadow updates its copy of that node's application state by updating the data that changed during application processing. The context then switches back to Shadow.

<a href="/assets/design2.png"><img title="design2" src="/assets/design2.png" alt="" width="520" /></a>

_Shadow links to and runs real C applications. All variable application state is copied for each node and stored in Shadow's memory space. This unique approach allows Shadow to handle multiple versions of the same application at the same time._

Function Interposition
======================

Shadow runs real applications that run on regular UNIX-type systems. These applications expect a wide range of kernel libraries to be available for use. For example, sending and receiving data over the network, system time, and device polling are all generally handled by the kernel at some level. These system functions (and others) are intercepted and redirected through Shadow-specific versions. In this way, Shadow provides the ability for nodes to seamlessly communicate with each other over the virtual network without requiring any changes to the application code.

Shadow Design Webcast
=====================

See the following webcast for more information about Shadow's design, and for an explanation of some experiments that utilize this unique architecture. Checkout [the about page](/about) for our Shadow research publications.

{% youtube Tb7m8OdpD8A %}

[tor]: https://www.torproject.org/
[tormetrics]: https://metrics.torproject.org/
[torsource]: https://gitweb.torproject.org/tor.git
[wiki-ec2]: https://github.com/shadow/shadow/wiki/Running-Shadow-on-EC2
[wiki-scallion]: https://github.com/shadow/shadow/wiki/Using-the-scallion-plug-in
[youtube-shadow-design]: http://youtu.be/Tb7m8OdpD8A
