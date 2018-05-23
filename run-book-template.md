# Run Book / System Operation Manual

## Service or system overview

**DFS Identity Management Services:** 

### Business overview

_These services define the infrasture for and provide access to the DFS User Store and DFS Claims Store. These services should be able highly reliable and scalable to roughly a million users with only changes to the underlying infrastrutuce they are running on._

### Technical overview

_This system is an internal REST API using Node.js, Loopback, and MongoDB, all deployed in Docker containers to Kubernetes via Helm charts._

### Service Level Agreements (SLAs)

_This system should have 99.9% uptime, aside from scheduled manintance_

### Service owner

_DFS Identity Platform Team_

### Contributing applications, daemons, services, middleware

_Node.js runtime + Loopback for REST services + MongodB for storage + Docker + Kubernetes_

## System characteristics

_This services is core to many DFS software offerings and as such should be available 24/7 with only planned service outages_

#### Hours of operation - core features

_Everyday of the Year_

#### Hours of operation - secondary features

_Everyday of the Year_

### Data and processing flows

> How and where does data flow through the system? What controls or triggers data flows?

_Web request from Active Disclosure and other DFS products_

### Infrastructure and network design

> What servers, containers, schedulers, devices, vLANs, firewalls, etc. are needed?

_A Kubernetes Cluster with the ability to provision Persistent Volumes, DNS entries for each of the three services, Ambassador to map DNS entries to Kubernetes Services, Jenkins to build the containers_

### Resilience, Fault Tolerance (FT) and High Availability (HA)

> How is the system resilient to failure? What mechanisms for tolerating faults are implemented? How is the system/service made highly available?

_This system is fairly resistent to failure as the components are seperate enough that if one fails, it will not cause the others to fail. That said, if any component loses its connection to MongoDB, it will faill. Kuberntes comes with a number of built in safety mechanisms, including automatic restart of failed containers and pod autoscaling that will create more containers if the load on the existing containers becomes too high._

### Throttling and partial shutdown

#### Throttling and partial shutdown - external requests

_This system can be throttled via Ambassador. If parts of this system are shutdown, it will effect other external systems that rely on this system. If MongoDB is taken down or fails, the entire system and all its dependents will also fail._

#### Throttling and partial shutdown - internal components

_Each subsystem exposed a `/metrics` healthcheck endpoint. If a partial shutdown is required, you can stop any individual component without affecting the rest of this system, except MongoDB. If MongoDB is taken down or fails, the entire system and all its dependents will also fail._

### Expected traffic and load

> Details of the expected throughput/traffic: call volumes, peak periods, quiet periods. What factors drive the load: bookings, page views, number of items in Basket, etc.)

_(e.g. Max: 1000 requests per second with 400 concurrent users - Friday @ 16:00 to Sunday @ 18:00, driven by likelihood of barbecue activity in the neighborhood)_

#### Hot or peak periods

_

#### Warm periods

_

#### Cool or quiet periods

_

### Environmental differences

> What are the main differences between Production/Live and other environments? What kinds of things might therefore not be tested in upstream environments?

_There are no known environmental differences._

### Tools

> What tools are available to help operate the system?

_Helm, Jenkins, and Kubernetes for making changes to the deployment on the cluster. Elasticsearch for logs and The Kubernetes WebUI and Grafana for metrics and health statuses._

## Required resources
> What compute, storage, database, metrics, logging, and scaling resources are needed? What are the minimum and expected maximum sizes (in CPU cores, RAM, GB disk space, GBit/sec, etc.)?

### Required resources - compute

_(e.g. Min: 4 VMs with 2 vCPU each. Max: around 40 VMs)_

### Required resources - storage

_(e.g. Min: 10GB Azure blob storage. Max: around 500GB Azure blob storage)_

### Required resources - database

_(e.g. Min: 500GB Standard Tier RDS. Max: around 2TB Standard Tier RDS)_

### Required resources - metrics

_(e.g. Min: 100 metrics per node per minute. Max: around 6000 metrics per node per minute)_

### Required resources - logging

_(e.g. Min: 60 log lines per node per minute (100KB). Max: around 6000 log lines per node per minute (1MB))_

### Required resources - other

_(e.g. Min: 10 encryption requests per node per minute. Max: around 100 encryption requests per node per minute)_


## Security and access control

### Password and PII security

> What kind of security is in place for passwords and Personally Identifiable Information (PII)? Are the passwords hashed with a strong hash function and salted?

_This system does not store passwords and instead relys on third party IdP's to authenticate users._

### Ongoing security checks

> How will the system be monitored for security issues?

_The depencies for this system will be monitored for security vulnerabilities by NPM, which checks for newly reported and existing vulnerabilities every time packages are downloaded. The containers that the components of this system will be packaged into will be monitored for vulnerabilities by Twistlock._

## System configuration

### Configuration management

> How is configuration managed for the system?

_Helm Charts. Each componenent of this system has a chart defining how the service should operate._

### Secrets management

> How are configuration secrets managed?

_Secrets are generated on the fly and configured as Kubernetes Secrets objects for consumption by the services._

## System backup and restore

### Backup requirements

> Which parts of the system need to be backed up?

_Only the persistence layer needs to be backed up. We will Heptio Ark to snapshot the persisent volumes created by MongoDB._

### Backup procedures

> How does backup happen? Is service affected? Should the system be [partially] shut down first?

_(e.g. Backup happens from the read replica - live service is not affected)_

### Restore procedures

> How does restore happen? Is service affected? Should the system be [partially] shut down first?

_(e.g. The Booking service must be switched off before Restore happens otherwise transactions will be lost)_

## Monitoring and alerting

### Log aggregation solution

_The system will use Logstash for transport logs to an existing in-house Elastic Search cluster_

### Log message format

> What kind of log message format will be used? Structured logging with JSON? `log4j` style single-line output?

Log messages will use log4j compatible multi-line format with wrapped stack traces

### Events and error messages

> What significant events, state transitions and error events may be logged?

_All successful and unsuccessful requests as well as expections will be logged._

### Metrics

> What significant metrics will be generated?

_(e.g. Usual VM stats (CPU, disk, threads, etc.) + around 200 application technical metrics + around 400 user-level metrics)_

### Health checks

> How is the health of dependencies (components and systems) assessed? How does the system report its own health?

#### Health of dependencies

_(e.g. Use `/health` HTTP endpoint for internal components that expose it. Other systems and external endpoints: typically HTTP 200 but some synthetic checks for some services)_

#### Health of service

_(e.g. Provide `/health` HTTP endpoint: 200 --> basic health, 500 --> bad configuration + `/health/deps` for checking dependencies)_

## Operational tasks

### Deployment

> How is the software deployed? How does roll-back happen?

_This sytem is deployed with Helm and Jenkins. Rollback will be initiated via a Jenkins deployment._

### Batch processing

> What kind of batch processing takes place?

_None._

### Power procedures

> What needs to happen when machines are power-cycled?

_*** WARNING: we have not investigated this scenario yet! ***_

### Routine and sanity checks

> What kind of checks need to happen on a regular basis?

_All `/metrics` endpoints should be checked every 60 seconds._

### Troubleshooting

> How should troubleshooting happen? What tools are available?

_Use a combination of the `/metrics` endpoint checks, Grafana, and logs to diagnose problems._

## Maintenance tasks

### Patching

> How should patches be deployed and tested?

#### Normal patch cycle

_Use the standard patch test cycle together with deployment via Jenkins and Helm_

#### Zero-day vulnerabilities

_Zero-day vulnerabilities will be given highest priority and the normal patch cycle will be used to update the system._

### Daylight-saving time changes

> Is the software affected by daylight-saving time changes (both client and server)?

_This system is not affected by daylights saving time changes._

### Data cleardown

> Which data needs to be cleared down? How often? Which tools or scripts control cleardown? 

_No data in the system needs to be cleared down._
 
### Log rotation

> Is log rotation needed? How is it controlled? 

_This system uses logstash to transport logs and does save logs on disk. All local log information is lost when the container restarts._

## Failover and Recovery procedures

> What needs to happen when parts of the system are failed over to standby systems? What needs to during recovery? 

### Failover

_

### Recovery

_

### Troubleshooting Failover and Recovery

> What tools or scripts are available to troubleshoot failover and recovery operations?

_(e.g. Start with running `SELECT state__desc FROM sys.database__mirroring__endpoints` on the PRIMARY node and then use the scripts in the *db-failover* Git repo)_

