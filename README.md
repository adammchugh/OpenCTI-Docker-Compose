# OpenCTI Docker Compose

This OpenCTI Docker Swarm Compose file is intended to be run in an environment where multiple hosts have been configured with specific hardware capabilities.

In my own test lab I have opted to use VMs managed through VMware vSphere.

## Breakdown of virtualisation hosts

Note: This is my current home-lab deployment - however should you have the ability to store your OpenCTI deployment on more performant hardware, I highly recommend you migrate Disk Stores to SSD or NVMe storage.

* Host #1
  * Single CPU (4C/4T) 64 GB RAM
  * 512 GB SSD
  * 4TB HDD
  * **Docker VMs**
    * **Docker-Core** (4vCPU/24GB RAM)
      * 32GB SSD Disk Store - Ubuntu 20.04 LTS
      * 512GB HDD Disk Store - mapped to /var/lib/docker/
* Host #2
  * Single CPU (4C/8T) 32 GB RAM
  * 256 GB SSD
  * 2TB HDD
  * **Docker VMs**
    * **Docker-Processor** [Template] (2vCPU/4GB RAM)
      * 16GB SSD Disk Store - Ubuntu 20.04 LTS
      * 64GB HDD Disk Store - mapped to /var/lib/docker/
    * **Docker-Indexer** [Template] (2vCPU/4GB RAM)
      * 16GB SSD Disk Store - Ubuntu 20.04 LTS
      * 64GB SSD Disk Store - mapped to /var/lib/docker/

### Docker VM Roles
Roles within the OpenCTI deployment are controlled through docker node labels.
The following roles have been defined:

* system_role=**core** - Core OpenCTI services such as watchtower, redis, minio, rabbitmq, opencti
* system_role=**indexer** - Elasticsearch specific role
* system_role=**processor** - Worker and Connector services

### Docker VM Specifications and Configuration
Docker Hosts assigned the **core** role are ideally placed on relatively low performance hardware. If there is a requirement to prioritise speed or economical cpu/ram/disk - then core could be run on lower specification hardware.

Coversely, hosts assigned the **indexer** role are ideally placed on performant hardware with cpu/ram/disk appropriate to an Elasticsearch deployment. This ideally means the use of SSD or NVMe disk, an appropriate allocation of memory, and at least 2vCPU per host. It would also be best practice to have more than one indexer Docker Host in your swarm.
When using the Docker Processor template described above, migrating the 64GB Disk Store to more performant disk is recommended (this will make ingestion of CTI objects much more performant and result in less bottlenecks with your deployment).


Processor assigned Docker Hosts generally need less disk, but will be doing a lot of the heavy lifting in terms of CPU, but may also require the ability to burst RAM (depending on what the OpenCTI connector is doing on that host). 
The Docker Processor template described above may be appropriate for this deployment, however the 64GB Disk Store could likely be reduced down to 20-30GB to enable storage of the Docker Images and Containers.