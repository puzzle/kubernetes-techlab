# Lab 10.2: DaemonSet

A DaemonSet is almost identical to a normal Deploment, but makes sure that on every (or some specified) Node extractly one Pod is running. When a new Node is added, the DaemonSet automatically deploys a Pod on the new Node.
When the DaemonSet is deleted, all related Pods are deleted.

One excellent Case to use a DaemonSet is a Daemon to grab Logs from Nodes (e.g. fluentd, logstash or a Splunk-Forwarder)

More Information abount DaemonSets can be found in the [Kubernetes DaemonSet Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

---

**End Lab 10.2**

<p width="100px" align="right"><a href="10_3_jobs.md">Jobs →</a></p>

[← back to Chapter-Overview "Additional Concepts"](10_additional_concepts.md)

[← back to Overview](../README.md)
