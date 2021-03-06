# Performance and Scalability testing
Percona XtraDB high availability cluster with MySQL database.
***

## Contents

* [**Introduction**](#introduction)
* [**Getting Started**](#getting-started)
* [**Target Metrics**](#target-metrics)
* [**Running the tests**](#running-the-tests)
* [**Scenario-1**](#scenario-1)
* [**Scenario-2**](#scenario-2)
* [**Conclusion**](#conclusion)
* [**Notes**](#notes)

## Introduction
As part of the Program Increment - 1 (PI-1) of the Mojaloop Productionisation project, two major Proof of Concept(PoC) items were done; one a PoC for High Availability/Scalability for Transactional DB and another PoC for durable Message Stream Processing. This document gives a brief account of the approach taken to do performance, scalability and resiliency testing and the scenarios involved. There are also charts and numbers provided based on the results of these tests done towards the end of PI-1. The metrics recorded during this exercise could serve as a baseline for comparison during future performance tests.

## Getting Started
- Please follow the instructions in the [README.md](https://github.com/mojaloop/central-ledger/blob/master/README.md) for setting up a local environment.
- Download and install JMeter.
- Setup JMeter with test scripts as required. For this exercise, scripts used are from the [test-scripts repo](https://github.com/mojaloop/test-scripts).
- Additional setup required is detailed here: [KUBERNETES.md](https://github.com/mojaloop/central-ledger/blob/master/KUBERNETES.md).
- Review the Transfers process: [TransferGuide](https://github.com/mojaloop/central-ledger/blob/master/TransferGuide.md).
- The baseline performance for the **'Prepare step'** with postgres database and pre-PI 1 code-base was established at **202** transactions (TPS) per second. Point of note here is that this system was not scalable and as can be seen in the charts below, the comparison is provided for the scenario with only one central-ledger component as that is the only applicable case for the base code.

## Target Metrics
- **Response time** (Latency): This is one of the most important metric that greatly impacts user experience. Response time is the total time it takes after the client sends a request till it gets a response. For PI-1, only the 'Prepare' part of the transfer is calibrated and the Latency or Response time for this indicates the time taken for the 'Prepare step' to be processed.
- **Throughput**: In the current context Throughput refers to the Transactions per Second (TPS) handled by the central processes.
- **Load size**: This includes the number of threads (simulataneous requests) and the loop count or the time regarding how many total requests need to be made. The current exercise is not a pure load testing effort but more of performance testing, so this is just the total number of records.
- **Scalability**: This is a factor of the number of target services so that correlation between scaling and Throughput, Latency and such other related metrics. In the current scenario, scaling was charted for number of central-ledger services and DFSPs separately as part of relevant scenarios.
- **Error rate**: This metric indicates the percentage of requests that had an error response and this gives an insight into reliability and availability.

## Running the tests
- The first step is to get the JMeter scripts ready that can execute the prepare step of the Transfers process (as mentioned under the [getting-started](#getting-started) section) and as necessary do some validation (or log results for verification).
- The second step required is to create the data necessary for executing transfers.
- Get a deployment ready with the updated code-base to be used for testing.
- Configure the scripts to point to the appropriate central services API end-points in the target system (kubernetes cluster on AWS).

Below is the system overview:
![System Overview](/metrics-images/SystemOverview_PoC_HA_Scalability.jpg)

## Scenario-1
This section deals with the PoC for HA/Scalability for Transactional DB. This describes the activities done to establish **data integrity**, **resiliency** and **scalability** of the system that uses the code from the PoC. For the performance aspect, a comparison is also provided with the base code (pre PI-1). There are three parts to this scenario, as described in the following three sub-sections - **Failover Testing**, **Scalability Testing** and **Performance Testing**.

#### Testing Failover
To provide conclusive proof of the Percona XtraDB cluster's ability to recover seamlessly from a disaster on a host, the following actions were taken:
- Ensure all required services of the setup and the database instances are running and stable.
- From the Kubernetes UI, "delete" one of the database pod's.
- Monitor the UI to ensure the pod is "killed".
- After confirming the pod is unavailable, monitor automated recovery (self-healing).
- Once automated recovery of the effected pod is completed, and the JMeter process is completed, a dump is made of all 3 database clusters individually, as well as the accessible database point via the ingress load balancer.
- A report is obtained from the JMeter process containing all the prepare statements (and the UUIDs) made during this test.
- Data from the 3 Databases in the DB cluster is compared to ensure data integrity, resiliency and seamless recovery.
- The Jmeter report is also compared with the database dumps to ensure no data was lost during the process, thereby establishing **high availability** and **data integrity**.

#### Scalability/Performance Testing
Base performance was established to understand the baseline using three Percona DB cluster setup in the Master/Master/Master configuration, using one central-ledger service.
A baseline was established for 200 DFSP users (threads in this case) achieving an average throughput of **211.5** TPS. The following figures for the below benchmark to indicate the scalability of the Percona cluster, using 150-500 concurrent DFSP users and increase the number of central-directory services. The below numbers are achieved by running 150-500 threads on JMeter which simulates a corresponding number of concurrent DFSP users

- With one central-ledger instance, an average throughput of **211.50** Transactions Per Second (TPS) was observed, whereas for this same scenario with the base code (postgres), the throughput was **202** TPS.
- With two central-ledger instances, an average throughput of **456.8** TPS was observed.
- With three central-ledger instances, an average throughput of **646.03** TPS was observed.
- With four central-ledger instances, an average throughput of **725.3** TPS was observed.
- With five central-ledger instances, an average throughput of **732** TPS was observed.

A linear increase with the increase in central-ledger instances on Kubernetes and the throughput (TPS) was observed. With five central-ledger instances the graph was starting to "flatten". More investigation is needed to identify the root case of this, whether that is system resources or JMeter limitations or something else. Below is a chart that shows the performance metrics charted with TPS against number of central-services instances. 
![Here is a chart that shows the performance metrics](/metrics-images/PoC_DB_Performance_HA_Scalability.jpg)

## Scenario-2
This section deals with the PoC for durable Message Stream Processing. This describes the activities done to establish **reliability** (error rate), **Scalability** and **Performance** of the system that uses the code from the PoC. For the performance aspect, a comparison is also provided with the base code (pre PI-1).

Below is a chart that shows the Stream Processing Performance Scenarios with TPS against the number of DFSPs and number of central-ledger services separately. 
![Here is a chart that shows the performance metrics](/metrics-images/StreamProcessing–PerformanceScenarios.jpg)

Below is a chart that shows the Stream Processing Performance comparison as-is with TPS against number of DFSPs. 
![Here is a chart that shows the performance metrics](/metrics-images/StreamProcessing–PerformanceComparisonToAs-Is.jpg)

## Conclusion
Following the above steps during the testing and verification process on the transactional database, **High Availability (HA)**, **Scalability (HS)**, **Resiliency** and **Data-Integrity** were established and also the advantages and factors in favor of implementing the PoCs that were done.

## Notes
- For future performance analysis runs (in Sprints 2.3, 2.4), Charting needs to include Latency and scenarios with average latency not greater than 1 second or 1.5 seconds need to be charted.
- The JMeter scripts were run from a AWS VM that was on the same Data Center as that of the target system to ensure consistent numbers.
- Control tests were executed on the Amazon Web Services Cluster specifically setup to establish a baseline.
- There is a need to identify the range that is relevant for this project for variables such as *'number of threads'* (10-500?), *'loop count'*, *'scalability factor'* and such others.
- For the Scenario-1, three m4.xlarge AWS VMs were used and for Scenario-2, three i3.xlarge AWS VMs were used.
