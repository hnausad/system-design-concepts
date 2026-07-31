# 50 System Design Concepts Explained 

# Section 1: Core Infrastructure Concepts (1-10)

## 1. Scalability
Scalability is the ability of a system to handle growth without breaking. Growth can mean more users, more data, more requests, or more geographic reach.

A scalable system expands its capacity to meet that growth while keeping performance acceptable.
The important thing to understand about scalability is that it is not a property of any single component. It is a property of the entire system, and the system is only as scalable as its least scalable part. You can scale your application servers to handle ten times the traffic and the bottleneck simply moves to the database.
True scalability requires every layer to be able to grow.

## 2. Vertical Scaling
Vertical scaling means making one machine more powerful. When the system needs more capacity, you upgrade the server with more CPU, more memory, or faster storage. The application does not change and the architecture does not change. You just get a bigger machine.
Vertical scaling is simple and works well until it does not.
The problem is that single machines have a maximum size, and a single machine is a single point of failure.
There is no redundancy. When it goes down, everything goes down with it.

