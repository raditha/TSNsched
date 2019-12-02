# TSNSched

> TSNSched uses the [Z3 theorem solver](https://github.com/Z3Prover/z3) to generate traffic schedules for [Time Sensitive Networking (TSN)](https://en.wikipedia.org/wiki/Time-Sensitive_Networking).

This repository is a result of research conducted at [fortiss](https://www.fortiss.org/en/) to develop a Time-Aware Shaper for TSN systems. 

## Table of Contents

- [Quickstart Guide](#quickstart-guide)
- [Requirements](#requirements)
- [Command Line Usage](#command-line-usage)
  * [The Input](#the-input)
  * [Executing the program](#executing-the-program)
  * [The output](#the-output)
- [Using as a Library](#using-as-a-library)
  * [Setting up the network](#setting-up-the-network)
  * [Executing the program](#executing-the-program-1)
  * [The output](#the-output-1)
- [Generating topologies](#generating-topologies)
  * [Repository files:](#repository-files-)
  * [Execution instructions for the Z3 code](#execution-instructions-for-the-z3-code)
- [Overview of Classes](#overview-of-classes)
  * [Device](#device)
  * [Flow](#flow)
  * [FlowFragment](#flowfragment)
  * [Switch](#switch)
  * [TSNSwitch](#tsnswitch)
  * [Network](#network)
  * [PathNode](#pathnode)
  * [PathTree](#pathtree)
  * [Cycle](#cycle)
  * [Port](#port)
  * [ScheduleGenerator](#schedulegenerator)

## Quickstart Guide

TSNSched can be started by a prepared script:

```
cd /home/cav/tsnsched_artifact/Script/
./generateSchedule.sh example.java
```

The input java file (example.java) describes a network topology (10 switches, 50 devices) with one small flow 4 subscribers and 3 switches in the path tree as illustrated below:

    P -> SW1 -> SW2 -> Sub1
         |       |
         |       V
         |       Sub2
         V        
         SW3 -> Sub3
         |
         V
        Sub4

The generated TSN schedule can be found in the output directory in the file output/example.java.log and output/example.java.out.
The total execution time, average latency and average jitter of the topology the scheduler solved can be found at the end of the output/example.java.out file.

You can find the java files with the network topologies described in the submitted paper in the following folder:

/home/cav/tsnsched_artifact/TestCasesPaper

You can also generate other topologies using our flow generator. How to use this generator is shown in section "Generating Topologies".

Have fun testing :-) 

## Requirements

It is recommended that the user runs the code using Java Version 1.8.0_181 and the Z3 package version 4.8.0.0. Both TSNSched and Z3 libraries must be part of the Java project build path in order to use the classes without any errors, if used as a library. If used in the command line, these same files must be part of the classpath. 

In order to be able to use the Z3 library, you need to add a few dependencies to your system by installing it from source like follows:

```
git clone https://github.com/Z3Prover/z3.git
python scripts/mk_make.py --java
cd build
make
sudo make install
```

The tests of both eclipse project and command line versions were run on a machine with Ubuntu 18.04.1 LTS as its operational system. The GNU bash version used was the 4.4.19(1)-release (x86_64-pc-linux-gnu). The Eclipse IDE version used was the Photon Release (4.8.0).

## Command Line Usage

The files for the command line usage of this project are all stored in the folder "Script" in this repository. They can be downloaded and used separately.

### The Input

For the input of the command line version of this project, the user must provide a java file with a method called "runTestCase()". This function must implement the network and call the generateSchedule(Network net) method from a TSNSched object. The user can then call methods of the flows and cycles to evaluate the latency, jitter, cycle duration, cycle start, etc.

If the user is not interested in building his own network, we also make available a topology generator, discussed later in this file. The output of this generator is already in the format accepted by the execution script. Samples generated by this tool can be found in the folder "TestCase" and are indentified by the .java extension.

This file must be placed inside the folder "Script". The name of the file does not matter, as it will be an input on the command line.

### Executing the program

For the command line usage, a script was developed in order to compile the Java file containing the network topology, execute it and handle the input and output files.

Once the input file is placed in the same folder of the script, the user must execute the script giving the java file containing the topology as an argument. Given that the name of the file is "example.java", the command will look like the following:

```
./generateSchedule.sh example.java
```

For practical reasons, the given file will be duplicated, renamed, parsed by the script in order to adapt the code. Then, the files will be compiled and executed with references to the Z3 and TSNSched libraries, placed on the subfolder "libs". The 2 output files will be generated and placed on the folder "output" under the the same name of the argument given in the execution of the switch, but now with the extra extensions .out and .log.   

### The output

The output of this process can be found in the "output" subfolder. If the network topology was specified on a file called "topology.java", then the user should be able to find two new files in the output folder called "topology.java.out" and "topology.java.log", if there was no problem executing the script.

The files with the extension .out contain the printed model generated by Z3 with the extra output created by the user (optional).

The files with the extension .log contain the information about the topology, as well as the Z3 values generated for the properties of the network. These files will be divided in a list of switches and a list of flows. 

The list of switches contains individual information about each switch (such as transmission time and time to travel) and its ports (virtual index, first cycle start and duration for debugging purposes. Mostly redundant). 

The list of flows contains individual information about each flow. Here, the user can check the flow fragments to retrieve the values of the priority of the flow on a certain switch, the slot start and duration of that flow, and the arrival, departure and scheduled times of each of the packets that goes through the switch covered by this fragment of the flow. 

Currently, the scheduler is building the schedule for 5 packets sent by each flow, which can be configured for different settings. Due to this, the user might indentify a pattern on the log files. A index of the packet between parenthisis can be seen followed by the departure, arrival and scheduled times. After this, a dashed line will be printed, separating it from the information about the next packet of the same flow. 

## Using as a Library

This tool also can be imported as a library allowing the user to aggregate TSNSched functionalities to other projects. With this, not only the topology can be handled as the developer wishes, the values generated by the scheduler will be stored in the cycle and flow objects in the program, allowing users to manipulate data without waiting for an output of a program external to their projects.


### Setting up the network

After adding the TSNSched and Z3 packages to the Java build path, one only needs to import the classes in order to be able to make use of it:

```
import schedule_generator.*;
```

With this, components of the network can be created:

```
// Creating a device
Device dev = new Device(float packetPeriodicity,  //  Periodicity of the packet
                        float hardConstraint);    //  Maximum latency tolerated by this device (Hard constraint)

// Creating a switch
TSNSwitch switch = new TSNSwitch(float timeToTravel,      // Time taken to travel on the medium connected to this switch
                                 float transmissionTime);  // Time taken to transmit a packet inside this switch
                                 
// Creating a cycle
Cycle cycle = new Cycle(float upperBoundCycleTime,    // Maximum duration of the cycle
                        float lowerBoundCycleTime,    // Minimum duration of the cycle
                        float maximumSlotDuration);   // Maximum duration of a time window of the cycle

// With the cycle, create ports. 
// First parameter is the device that is being connected to the switch, second is the cycle of the port.
switch.createPort(Device deviceA, Cycle cycle1); 
switch.createPort(Switch switchB, Cycle cycle2); 

// Creating a unicast flow:
Flow flow = new Flow(Flow.UNICAST);

// Setting start device, path and end device of a unicast flow
flow.setStartDevice(Device devA);
flow.addToPath(Switch switchA);
flow.addToPath(Switch switchB);
flow.setEndDevice(Device devB);
	
// Creating a publish subscribe flow:
Flow flow = new Flow(Flow.PUBLISH_SUBSCRIBE);

// Creating a path for a publish subscribe flow (Similar to creating a simple tree, node by node):
PathTree pathTree = new PathTree();
PathNode pathNode;
pathNode = pathTree.addRoot(Device devA); // Setting root, returns the reference to the root node
pathNode = pathNode.addChild(Switch switchA); // Adding switchA as the next hop for the flow, returns the reference to the new node added
pathNode.addChild(Device devB); // Adding a device as a subscriber 
pathNode.addChild(Device devC); // Adding a device as a subscriber 
flow.setPathTree(pathTree); // Giving the path to the flow

// Creating and populating a network (Giving switches and flows to it):
Network net = new Network(float jitterUpperBoundRange); // Creating a network giving the maximum average jitter allowed per flow
net.addSwitch(Switch switchA); 
net.addSwitch(Switch switchB); 
net.addSwitch(Switch switchC); 
net.addFlow(Flow flowA); 
net.addFlow(Flow flowB); 

```


### Executing the program

The user must add both Z3 and TSNSched packages to the classpath of the project. These two files can be found in the "libs" folder of this repository.

Most of the sofisticated IDEs can do this just by adding external libraries as JAR files on the configuration of the projects.

If compiling the project in the command line, do not forget to add the libraries manually or set them in the Java PATH environment variable.

After setting up the network, the user must now call the method for generating a schedule:

```
// Generating a schedule:
ScheduleGenerator scheduleGenerator = new ScheduleGenerator();
scheduleGenerator.generateSchedule(Network net); // The network is the input for the schedule generation
```
### The output

A "log.txt" file must be generated within the project folder. This file contains the information about the topology, as well as the Z3 values generated for the properties of the network (such as cycle start and duration, priorities and packet times).

After setting up the network and generating the schedule, the cycles and flows are now able to return numeric data to the user:

```
// Retreiving the departure time from a packet in a publish subscribe flow 
flow.getDepartureTime(Device targetDevice,     // Destination of the packet. One of the subscribers 
                      int switchNum,           // Index of the switch in the path
                      int packetNum);          // Index of the packet in the sequence

// Retreiving the arrival time from a packet in a publish subscribe flow 
flow.getArrivalTime(Device targetDevice,       // Destination of the packet. One of the subscribers 
                    int switchNum,             // Index of the switch in the path
                    int packetNum);            // Index of the packet in the sequence

// Retreiving the scheduled time from a packet in a publish subscribe flow 
flow.getScheduledTime(Device targetDevice,       // Destination of the packet. One of the subscribers 
                      int switchNum,             // Number of the switch in the path
                      int packetNum);            // Index of the packet in the sequence

// Retrieving the average latency and average jitter of a flow, respectively:
flow.getAverageLatency();
flow.getAverageJitter();

// Retrieving the cycle duration and cycle start of a flow:
cycle.getCycleDuration();
cycle.getCycleStart();

// Retrieving the index of priorities used:
cycle.getSlotsUsed();

// Retrieving the priority slot duration and priority slot start:
cycle.getSlotStart(int prt);        // Index of a priority
cycle.getSlotDuration(int prt);     // Index of a priority

```

## Generating topologies

To aid in the generation of topologies for the testing of TSNSched, a generator of Java files containing the specification of a network was created. It can be found in the folder "ScenarioGenerator" of this repository.

Basically, given certain properties of a network as variables, they can be set in order to generate a topology according to the user's needs. The number of devices, switches, flows and constructor parameters are set, and then the file is written.

Currently, the number of nodes and subscribers is used in the following pattern. To create a small flow (3 switches in the path tree and 5 subscribers), the value of the configuration variable is 1. To create a medium flow (5 switches in the path tree and 10 subscribers), the value of the configuration variable is 2. To create a large flow (7 switches in the path tree and 10 subscribers), the value of the configuration variable is 3.

Firstly, a publisher device is picked from the pool of devices and it is made the root node, then the switch that it connects to is added to the path tree. From now own, every node in the path tree can have randomly up to 2 children nodes that will be switches picked from the mesh network. While the number of switches in the tree is smaller the *numberOfNodes* variable (which represents the number of switch nodes in the tree), branches will be created by level. This way, there can be at most a difference of one between the size of the biggest branch and the smallest branch. At this point, a number of devices that can go up to the specified number of subscribers will be equally divided by the switches in the end of the branches.

Even though the variation of flows in a generated file isn't too great (as to avoid creating completely different scenarios with similar configuration), the topologies created by this tool can be really complex to be solved.

To run the tool, the user must set the value of the variables according to the desired topology and then run the following commands on the ScenarioGenerator folder of this repository:

```
javac *.java
java ScenarioGenerator
```

The output file (GeneratedCode.java which contains the topology) will be generated within the same folder.

<!--
### Repository files:

|  File  |  Description  |
| ------ | ------ |
|Z3Code/scheduler-scenario1.z3|Z3 modeling of a simple scheduler (1 sender, 1 switch, 1 receiver) scenario with dynamic priorities and time slots|
|Z3Code/scheduler-FTS|Z3 modeling of a simple scheduler scenario with fixed time slots|
|Z3Code/scheduler-simple|Z3 modeling of a simple scheduler scenario with one time slot|
|Documents/Scenarios|PDF document containing the formal specification of two TSN scenarios|
|src/*|Files used by the Java project|

### Execution instructions for the Z3 code
* Open the file with the desired modeled scenario  
* Copy the content of the file
* Load the [Z3 website][z3]
* Paste the copied code on the website editor
* Press the "run" button
* The output will be printed bellow the editor
-->


## Overview of Classes

A brief presentation of the project classes and its quirks.

### Device

The Device class represents a start or end device in a TSN flow. All the properties of device nodes in the network are specified here. These properties are rather trivial to understand but they are build the core of the constraints of a flow. Here, the user can specify the packet periodicity of the flow and the hard constraint (maximum allowed latency). 

### Flow

This class specifies a flow (or a stream, in other words) of packets from one source to one or multiple destinations. It contains references for all the data related to this flow, including path, timing, packet properties, so on and so forth. The flows can be unicast type or publish subscribe type flows.

In a more in deph perspective, each flow will later be broken into "smaller flows", called flow fragments. This class will also store the reference to them in a simple ArrayList (for unicast flows) or inside a PathTree object (for publish subscribe flows).

### FlowFragment

This class is used to represent a fragment of a flow. Simply put, a flow fragment represents the flow it belongs to regarding a specific switch in the path. With this approach, a flow, regardless of its type, can be broken into flow fragments and distributed to the switches in the network. It holds the time values of the departure time (leaving the previous node), arrival time (arriving in the current node) and scheduled time (leaving the current node) of packets from this flow on the switch it belongs to. Since these times are specified as Z3 objects, there is no need to store copies of them, just the reference.

This approach allows the user to have a more encapsulated code, since it doesn't matter the type of flow being used here, the user can simply break the flow of packets into nuclear streams (a stream that only covers one hop), and visualize the fragment of a flow as a link in a chain.

FlowFragment objects store information about the current and next nodes in its path and also the departure time, arrival time, scheduled time of the packets that go through it. It is important to have in mind that the departure, arrival and scheduled times stored by FlowFragment objects are float values, not Z3 variables. The Z3 variable for these values can be retrieved through the port or the switch that this fragment goes through.


### Switch
Contains most of the properties of a normal switch that are used to build the schedule. Since this  scheduler doesn't take in consideration scenarios where normal switches and TSN switches interact, no Z3 properties had to be specified in this class. 
 
It is currently used as parent class for TSNSwitch. Can be used to further extend the usability of this project in the future.

### TSNSwitch

This class contains the information needed to specify a switch capable of complying with the TSN patterns to the schedule. Aside from part of the Z3 data used to generate the schedule, objects created from this class are able to organize a sequence of ports that connect the switch to other nodes in the network.

TSNSwitch objects also can reference the Z3 variables for the departure, arrival and scheduled time of FlowFragments.  

### Network

Using this class, the user can specify the network topology using switches and flows. The network will be given to the scheduler generator so it can iterate over the network's flows and switches setting up the scheduling rules.

In this current implementation, the upper bound jitter variation is specified in the Network, making it uniform for all flows added to the topology.

### PathNode

In a publish subscribe flow, contains the data needed in each node of a pathTree. Since a publish subscribe flow path can be seen as a tree, a single node on that tree is a PathNode.

Can reference a father, possesses an device or switch, a list of children and a flow fragment for each of the children in case of being a switch.

### PathTree

Used to specify the path on publish subscribe flows. Has a reference to the PathNode root, which contains the starting device, and also references to the leaves, which contain the destinations of the publisher messages.

It is basically a tree of PathNodes with a few simple and classic tree methods. 

### Cycle

The Cycle class represents the cycle of a TSN switch. A cycle is a time interval with a specific duration where time windows can be distributed according to constraints to prioritize critical traffic. During these time windows, the gate of the respective priority queue will be open. Each cycle has a a start, a duration and a sequence of time windows (priority slots).

In this project implementation, since a set of priority queues is given for every port in a switch, every port has a cycle, but the cycle start and duration is the same for every port.

After the specification of its properties through user input, the toZ3 method can be used to convert the values to Z3 variables and query the unknown values. 
 
There is no direct reference from a cycle to its time slots. The user must use a priority from a flow to reference the time window of a cycle. This happens because of the generation of Z3 variables. 
 
For example, if I want to know the duration of the time slot reserved for the priority 3, it most likely will return a value different from the actual time slot that a flow is making use. This happens due to the way that Z3 variables are generated. A flow fragment can have a priority 3 on this cycle, but its variable name will be "flowNfragmentMpriority". Even if Z3 says that this variable's value is 3, the reference to the cycle duration will be called "cycleXSlotflowNfragmentMpriorityDuration", which is clearly different from "cycleXSlot3Duration".
 
To make this work, every flow that has the same time window has the same priority value. And this value is limited to a maximum value *numOfSlots*. So, to access the slot start and duration of a certain priority, a flow fragment from that priority must be retrieved. This also deals with the problem of having unused priorities, which can end up causing problems due to constraints of guard band and such.

### Port

This class is used to implement the logical role of a port of a switch for the scheduler. The core of the scheduling process happens here. Simplifying the whole process, the other classes in this project are used to create, manage and break flows into smaller pieces. These pieces are given to the switches, and from the switches they will be given to their respective ports according to the path of the flow.
 
After this is done, each port now has an array of fragments of flows that are going through them. This way, it is easier to schedule the packets since all you have to focus are the flow fragments that might conflict in this specific port. The type of flow, its path or anything else does not matter at this point.

### ScheduleGenerator

Used to generate a schedule based on the properties of a given network through the method generateSchedule. Will create a log file and store the timing properties on the cycles and flows.
