# Process Check

## Overview

The process check lets you:

* Collect resource usage metrics for specific running processes on any host: CPU, memory, I/O, number of threads, etc
* Use [Process Monitors](http://docs.datadoghq.com/monitoring/#process): configure thresholds for how many instances of a specific process ought to be running and get alerts when the thresholds aren't met (see **Service Checks** below).

## Setup
### Installation

The process check is packaged with the Agent, so simply [install the Agent](https://app.datadoghq.com/account/settings#agent) anywhere you want to use the check. If you need the newest version of the check, install the `dd-check-process` package.

### Configuration

Unlike many checks, the process check doesn't monitor anything useful by default; you must tell it which processes you want to monitor, and how.

While there's no standard default check configuration, here's an example `process.yaml` that monitors ssh/sshd processes:

```
init_config:

instances:
  - name: ssh
    search_string: ['ssh', 'sshd']

# To search for sshd processes using an exact cmdline
# - name: ssh
#   search_string: ['/usr/sbin/sshd -D']
#   exact_match: True
```

You can also configure the check to find any process by exact PID (`pid`) or pidfile (`pid_file`). If you provide more than one of `search_string`, `pid`, and `pid_file`, the check will the first option it finds in that order (e.g. it uses `search_string` over `pid_file` if you configure both).

To have the check search for processes in a path other than `/proc`, set `procfs_path: <your_proc_path>` in `datadog.conf`, NOT in `process.yaml` (its use has been deprecated there). Set this to `/host/proc` if you're running the Agent from a Docker container (i.e. [docker-dd-agent](https://github.com/DataDog/docker-dd-agent)) and want to monitor processes running on the server hosting your containers. You DON'T need to set this to monitor processes running _in_ your containers; the [Docker check](https://github.com/DataDog/integrations-core/tree/master/docker_daemon) monitors those.

See the [example configuration](https://github.com/DataDog/integrations-core/blob/master/process/conf.yaml.example) for more details on configuration options.

Restart the Agent to start sending process metrics and service checks to Datadog.

### Validation

Run the Agent's `info` subcommand and look for `process` under the Checks section:

```
  Checks
  ======
    [...]

    mcache
    -------
      - instance #0 [OK]
      - instance #1 [OK]
      - Collected 26 metrics, 0 events & 1 service check

    [...]
```

Each instance configured in `process.yaml` should have one `instance #<num> [OK]` line in the output, regardless of how many search_strings it might be configured with.

## Compatibility

The process check is compatible with all major platforms.

## Data Collected
### Metrics
See [metadata.csv](https://github.com/DataDog/integrations-core/blob/master/process/metadata.csv) for a list of metrics provided by this check.

All metrics are per `instance` configured in process.yaml, and are tagged `process_name:<instance_name>`.

### Events
The Process check does not include any event at this time.

### Service Checks
**process.up**:

The Agent submits this service check for each instance in `process.yaml`, tagging each with `process:<name>`.

For an instance with no `thresholds` specified, the service check has a status of either CRITICAL (zero processes running) or OK (at least one process running).

For an instance with `thresholds` specified, consider this example:

```
instances:
  - name: my_worker_process
    search_string: ['/usr/local/bin/worker']
    thresholds:
      critical: [1, 7]
      warning: [3, 5]
```

The Agent submits a `process.up` tagged `process:my_worker_process` whose status is:

- CRITICAL when there are less than 1 or more than 7 worker processes
- WARNING when there are 1, 2, 6, or 7 worker processes
- OK when there are 3, 4 or 5 worker processes

## Troubleshooting

If you have any questions about Datadog or a use case our [Docs](https://docs.datadoghq.com/) didn’t mention, we’d love to help! Here’s how you can reach out to us:

### Visit the Knowledge Base

Learn more about what you can do in Datadog on the [Support Knowledge Base](https://datadog.zendesk.com/agent/).

### Web Support

Messages in the [event stream](https://app.datadoghq.com/event/stream) containing **@support-datadog** will reach our Support Team. This is a convenient channel for referencing graph snapshots or a particular event. In addition, we have a livechat service available during the day (EST) from any page within the app.

### By Email

You can also contact our Support Team via email at [support@datadoghq.com](mailto:support@datadoghq.com).

### Over Slack

Reach out to our team and other Datadog users on [Slack](http://chat.datadoghq.com/).

## Further Reading
To get a better idea of how (or why) to monitor process resource consumption with Datadog, check out our [series of blog posts](https://www.datadoghq.com/blog/process-check-monitoring/) about it.
