# Lab 10.2: DaemonSet

A DaemonSet is almost identical to a normal deployment, but makes sure that on every (or some specified) Node exactly one pod is running. When a new node is added, the daemonset automatically deploys a pod on the new node.
When the daemonset is deleted, all related pods are deleted.

One excellent case to use a daemonset is a daemon to grab Logs from Nodes (e.g. fluentd, logstash or a Splunk-Forwarder)

More information about daemonsets can be found in the [Kubernetes DaemonSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

---

**End Lab 10.2**

<p width="100px" align="right"><a href="10_3_jobs.md">Jobs →</a></p>

[← back to Chapter-Overview "Additional Concepts"](10_additional_concepts.md)

[← back to Overview](../README.md)
