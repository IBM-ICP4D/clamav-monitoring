

ClamAV is an open source antivirus engine for detecting trojans, viruses, malware, and other malicious threats.

## Basic usage

1. Build your Docker image.
1. Deploy that image to your Kubernetes cluster.
1. Use Daemonsets to configure the new workload to run one scanner pod per node.
1. Ensure scan-required paths within other pods are mounted as named volumes so they will be included in the scan of the node.

