## What can you borrow from Kubernetes design principles 


### Kubernetes Key Design Principles
1. Declarative Configuration over Imperative
    * User declares a desired state of the system to produce a result. e.g., `I would like there to be FIVE replicas of my web server running at all times`.
    * One of the key benefits of declarative configuration is to enable `self-correcting and self-healing behaviors` elegantly.
    * **[Very Debatable and Subjective]** Solutions and Tooling like Puppet/Chef/Ansible is heavily (or at a high percentage) based on `Imperative Configuration`, which is also known as `Procedural Configuration`. e.g., `Create FIVE replicas of my web server!`.
      
      Below is an example of Chef Recipe:
        ```
        packages = ['vim', 'git', 'curl']
        
        packages.each do |package|
         apt_package package do
           action :install
         end
        end
        ```
2. Reconciliation over Global State Controller
    * The state control is decentralized to a large number of controllers, each performing its own independent reconciliation loop.
    * **NOTE**: *The easiest example to help you understand the operation of a reconciliation control loop is the thermostat in your home. It has a desired state (the temperature that you entered on the thermostat), it makes observations of the world (the current temperature of your house), it finds the difference between these values, and it then takes actions (either heating or cooling) to make the real world match the desired state of the world.*
    * Monolithic state controller usually have one centralized controller which has a global view of the world. One of the examples would be the `global view` controller in Software Defined Network's control plane.
    
3. Implicit/Dynamic Grouping over Static
    * Whether it is grouping together a set of replicas or identifying the backends for a load balancer, there are numerous times in the implementation of Kubernetes when it is necessary to identify a set of things.
    * In Kubernetes, this implicit grouping is achieved via labels and label queries or label selectors. Every API object in Kubernetes can have an arbitrary number of key/value pairs called “labels” that are associated with the object.

### What does it means for your product(s)/platform?
