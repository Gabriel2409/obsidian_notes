---
sr-due: 2023-05-16
sr-interval: 3
sr-ease: 250
---

#sd

## Definition

Performance in a data system is measured by looking at a given metric

## Example of metrics

### Throughput

In batch processing systems (such as Hadoop), we usually care about the number of records we can process per second = the **throughput**

### Response time and latency

In online services, we usually care more about the time it takes between a request and a response (called the **response time**).

- **Response Time** = actual time to process the request (the **service time**) + network delays + queuing delays
- **Latency** = Duration that a request is waiting to be handled

Because these can vary a lot depending on when / where the request is made, we use a distribution and describe them using **percentiles**. For ex, p999 = 200ms means that 99.9% of response times are below 200ms.
In most cases, most requests are fast enough but some of them are very slow (high percentiles = **tail latencies**).

Percentiles are often used in **Service Level Objectives (SLO)** and **Service Level Agreements (SLA)** contracts that define tha expected performance and availability of the service with potentially some partial refunds if the performances are not met
