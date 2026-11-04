# Deprecate Cruise Control Client for future removal

Cruise Control has Python client and CLI tool for interacting with the REST API. This proposes deprecating it and marking it for deletion in a future release.

## Current situation

In the repository, there is a Python library under cruise-control-client. In the past this was released via a [PyPI project](https://pypi.org/project/cruise-control-client/) at a different release cycle than the rest of Cruise Control. Its last release was in September 2021.

## Motivation

The Python library has not been updated since October 2023. The last notable commit added support for the `permissions` endpoint and never made it into a release. It is also missing other features that were added to Cruise Control. In particular it's missing support for the `remove_disk` endpoint. In addition it's missing dozens of arguments on other endpoints. Here is an overview:

| Endpoint | Missing Parameters |
|---|---|
| `rebalance` | `kafka_assigner`, `fast_mode`, `rebalance_disk`, `broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `concurrent_intra_broker_partition_movements`, `execution_progress_check_interval_ms`, `replication_throttle` |
| `add_broker` | `kafka_assigner`, `fast_mode`, `broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `execution_progress_check_interval_ms`, `replication_throttle`, `throttle_added_broker` |
| `remove_broker` | `kafka_assigner`, `fast_mode`, `broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `execution_progress_check_interval_ms`, `replication_throttle` |
| `admin` | `broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `concurrent_intra_broker_partition_movements`, `execution_progress_check_interval_ms`, `disable_concurrency_adjuster_for`, `enable_concurrency_adjuster_for`, `min_isr_based_concurrency_adjustment` |
| `demote_broker` | `broker_concurrent_leader_movements`, `execution_progress_check_interval_ms`, `replication_throttle`, `brokerid_and_logdirs` |
| `topic_configuration` | `fast_mode`, `broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `execution_progress_check_interval_ms`, `replication_throttle` |
| `fix_offline_replicas` | `fast_mode`, `broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `execution_progress_check_interval_ms`, `replication_throttle` |
| `proposals` | `kafka_assigner`, `rebalance_disk`, `fast_mode` |
| `load` | `populate_disk_info`, `capacity_only` |
| `partition_load` | `avg_load` |
| `user_tasks` | `fetch_completed_task` |
| `stop_proposal_execution` | `stop_external_agent` |

One especially important missing parameter is `replication_throttle` which lets users set the bandwidth used to move replicas. Other notables include concurrency limits (`broker_concurrent_leader_movements`, `max_partition_movements_in_cluster`, `concurrent_intra_broker_partition_movements`, `min_isr_based_concurrency_adjustment`) and fast mode (`fast_mode`).
Note that with `add_param` users can pass arbitrary parameters, so it enables using all these features. However it lacks validation and it's not as convenient to use.

At the moment the TSC also does not own the [PyPI project](https://pypi.org/project/cruise-control-client/) which effectively prevents it from making new releases under the same name. [PyPI Stats](https://pypistats.org/packages/cruise-control-client) reports ~120 downloads last month. The "with mirrors" numbers are higher but include automated CI and mirror traffic. Either way, usage is low.

## Proposal

This proposes deprecating cruise-control-client in the next release and deleting it in the next major release. Note that the timeline of the next major release is still unclear.

Once deprecated, a warning will be logged when the Python module is used. This warning will only be present in the source copy and not in the PyPI release.

## Affected/not affected projects

This only impacts cruise-control-client.

## Compatibility

The PyPI project will be left untouched. It should carry on working, as the REST API is stable and well documented, but it will not get any new releases.
Once cruise-control-client is deleted from the repository, users relying on the sources directly will need to checkout the tag of the last release that contains it.

## Rejected alternatives

- Write a replacement CLI tool: We could write a CLI in Java that replicates cruise-control-client but that wouldn't satisfy users of the Python library.
- Bring the client up to date: Looking at the GitHub profiles of all TSC members, it seems there isn't a lot of Python expertise. This will also require taking ownership of the PyPI project.
- Extract the Python client in its own repository under the cruise-control-for-kafka GitHub organization: This only makes sense if we intend to maintain it and do new releases.
