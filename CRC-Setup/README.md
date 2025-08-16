
````markdown
# OpenShift CRC Setup Guide
Download CRC from below URL:
https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/crc/2.51.0

This guide provides useful commands and configuration options for setting up and
customizing your local OpenShift Cluster using CodeReady Containers (CRC).

---

## Basic CRC Workflow


# Stop the CRC cluster if it's already running
$ crc stop

# Enable cluster monitoring for metrics
$ crc config set enable-cluster-monitoring true

# Set disk size to 50GB
$ crc config set disk-size 90

# View current CRC configuration
$ crc config view

# Start the CRC cluster with the new configuration
$ crc start
````

---

## Other Configuration Options

Customize the resources and features according to your machine's capacity and project needs.

```bash
# Allocate more CPUs
$ crc config set cpus 6

# Allocate more memory (in MiB) - 16384 MiB = 16 GB
$ crc config set memory 16384

# Set disk size to 50GB
$ crc config set disk-size 50
```

### Enable Monitoring and Shared Folders

```bash
# Enable Prometheus and Grafana metrics collection
$ crc config set enable-cluster-monitoring true

# Enable shared folders between host and CRC VM
$ crc config set enable-shared-dirs true
```

### Apply Configuration Changes

```bash
# Start (or restart) the CRC cluster to apply changes
$ crc start
```

---

## Notes

* These settings persist across sessions unless you reset or delete your CRC setup.
* Adjust CPU and memory according to your system's hardware capacity.
* Enabling monitoring is useful for performance insights but consumes more resources.

```
```
