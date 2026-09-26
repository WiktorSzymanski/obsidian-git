# `Event storming`

This lesson discusses the design process for event-driven systems. When designing such systems, a great starting point is where events happen. There are a couple of design processes and design techniques such as `Event storming` and `Event modeling` that use the `Events` as a starting point.

The term `Event storming` refers to a rapid group modeling approach to domain-driven design. It is a workshop-style technique that brings project stakeholders together to explore complex business `Domains`. It is a valuable technique because it allows you to start with the big picture and use `Events` to discover the different aspects of a system that are important to you.

Some advantages of `Event storming` are:

- It reduces the time required to produce a business `Domain model`.
- It divides the process into simple terms so all stakeholders can understand it.
- It provides a hands-on approach to designing a `Domain model`.
- It Results in a complete behavioral `Model` that can be implemented and validated rapidly.

The steps of `Event storming` are:

1. Invite a mix of all stakeholders involved to design the `Domain model`.
2. Provide unlimited modeling space.
3. Explore the business domain:
    
    A. Identify domain `Events`.
    
    B. Connect domain `Events` to `Commands`.
    
    C. Connect `Events` to reactions (results of an event).
    
1. Combine with domain-driven design (group modules with `Bounded context`)
# `Event modeling`

Unlike `Event storming`, `Event modeling` focuses on details. Its aim is to deliver a working solution rather than a generic approach to designing any system.

`Event modeling` refers to a method of describing how information has changed over time in a system.

Some advantages of `Event modeling` are:

- It flattens the cost curve of the average feature cost.
- Implementing any other workflow step will not require revisiting other completed steps.
- Measuring the effort for an organization to implement can be done for many features over time.
- It is easy to introduce changes in the management of the model.
- It is easy to identify security issues as the model displays where sensitive data crosses boundaries.

The process of going from the domain to a running solution can be split into the following **steps**:

1. Identify the `Events`.
2. Plot all of these `Events` in a timeline.
3. Construct a storyboard:
    
    A. Wireframe.
    
    B. Identify the `Commands`.
    
    C. Identify the `Queries`.
    
    D. Identify swim lanes (`Aggregates`/contexts).