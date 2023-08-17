## Cloud Computing
#### Olli Alm & Ilkka Kylmäniemi

---

![Meme](img/cloud.jpg)

---

# Evolution of cloud computing

**Servers**

* Individual machines
* Fixed hardware, with fixed software
* One-to-one correspondence between hardware and operating system

**Virtualization**

* Virtual machine (VM): machine as logical construct, not "following" the hardware constraints
* Software / operating system (OS) is separated from hardware
* For one (hardware) machine, there can be multiple OSs
* For multiple machines, there can be one OS

**Cloud computing**

* Next generation of virtualization
* Big amount of computers somewhere
* Geographical location unclear
* No more talk about computers, but *units* or *nodes*

---

# NIST definition of cloud computing

* NIST: National Institute of Standards and Technology (US): http://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-145.pdf

1. On-demand self-service
   * Consumer controls the resources, immediately in use
   * Was before<sup>1</sup>: no self-service, long installation time
1. Broad network access
   * Use with any device with stadard protocols (HTTP)
   * Was: controlled access with special software
1. Resource pooling
   * Set of shared resources for all the users (multi-tenancy)
   * Was: resources were static: machine reserved for the specific users, resources wasted
1. Rapid elasticity
   * Scaling up and down of the resources dynamically
   * Was: no scaling, only by buying new hardware
1. Measured service
   * Billing is based on the resources used
   * Was: billing based on the static resources bought

(<sup>1</sup> *"was before"* --> how the things *were before* the cloud computing)

---

# 3 service models of cloud computing

*Service model:* what is provided for the customer

**SAAS**: Software as a service

* Customer gets *an application*, e.g. email, map application
* *SAAS* is for end users

**PAAS**: Platform as a service

* Customer gets *a server*, a platform to host software
* *PAAS* is for developers, no need to own server hardware

**IAAS**: Infrastructure as a service

* Customer gets *a stack of services* to support a company or organization
* Company IT-administrators could use *IAAS* and provide it for employees
* *IAAS* is "a service for providing services".


#### Good article: [Iaas vs. PaaS vs. SaaS](https://www.redhat.com/en/topics/cloud-computing/iaas-vs-paas-vs-saas?sc_cid=7013a000002pgRYAAY&gclid=CjwKCAjw5dqgBhBNEiwA7PryaEuwb1tCOZev80A45Zps0EkEHepja0o-BvkpB6qN4Rtrq8j3RaOSNRoCvL0QAvD_BwE&gclsrc=aw.ds)
