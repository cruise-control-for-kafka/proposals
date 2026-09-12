# First Release Under the Linux Foundation and Beyond

This proposal defines the scope and priorities for the first Cruise Control release under the Linux Foundation.
It also specifies a roadmap of development priorities for after the initial release.

## Current situation

The Cruise Control project has recently transitioned to the Linux Foundation and established governance through [GOVERNANCE](https://github.com/cruise-control-for-kafka/cruise-control/blob/main/GOVERNANCE.md) and [CHARTER](https://github.com/cruise-control-for-kafka/cruise-control/blob/main/CHARTER.md) files.
Communications channels (mailing lists, Slack, etc.) are currently being set up with the Linux Foundation.

Despite this progress, the project still has several issues that need to be addressed:

- **Outstanding CVEs**: The codebase has accumulated several CVEs, mostly from stale dependencies (e.g. older Jetty, Kafka versions, etc).

- **Dependency conflicts**: The Gradle build configuration contains redundant dependency version pins that are silently overridden by transitive dependencies, making it difficult to verify that CVE-affected versions have actually been replaced.

- **Unstable CI**: Known flaky tests intermittently fail and disrupt CI reliability, making it harder to merge the pull requests needed to prepare the release.

## Motivation

The project has been without a release for an extended period of time and users need a version that addresses the outstanding CVEs. 
The longer this takes the greater the exposure. 
The focus of this first release is to resolve these vulnerabilities as quickly as possible. 
The existing build and publication infrastructure is currently functional so the scope is limited to the CVE fixes and the work needed to fix and ship them quickly and reliably.

## Proposal

The following items are targeted for the first Cruise Control release under the Linux Foundation.
These target the changes required to eliminate known CVEs, stabilize CI, and clean up the build configuration.
Broader improvements to the codebase, APIs, and feature set, including the Java package rename from `com.linkedin` to `io.cruisecontrol` and Maven Central publication, are deferred to subsequent releases, specified in [Roadmap (Post-release)](#roadmap-post-release) section below. 

- **Upgrade to Kafka 4.3**: Upgrade Cruise Control to Kafka 4.3 to ensure compatibility with the latest Kafka version and resolve outstanding Kafka-related CVEs.
  There is already an [open pull request](https://github.com/cruise-control-for-kafka/cruise-control/pull/2342).

- **Upgrade to Jetty 12**: Migrate from pre-v12 Jetty to Jetty 12 because earlier versions are end-of-life and no longer receive security patches.
  There is already an [open pull request](https://github.com/cruise-control-for-kafka/cruise-control/pull/2307).

- **Remove duplicate and conflicting dependency declarations**: Audit the Gradle build configuration to eliminate redundant version pins that are silently overridden by transitive dependencies.
  Introduce a version BOM (`platform()`/`enforcedPlatform()`) to centralize dependency version management across subprojects.
  Without this there is no reliable way to confirm that upgrading a dependency version actually takes effect across the entire build, which undermines confidence that CVE-affected versions have been fully replaced.

- **Address remaining critical CVEs**: Upgrade or pin any remaining dependencies with known critical CVEs that are not resolved by the Kafka or Jetty upgrades (e.g. Log4j, Jackson, Commons Beanutils). 
  This may involve adding dependency constraints or forcing specific versions to ensure CVE-affected transitive dependencies are fully replaced.

- **Fix known flaky tests**: Assess known flaky tests and fix the ones deemed to be most impactful.
  There is already an [open pull request](https://github.com/cruise-control-for-kafka/cruise-control/pull/2338) that addresses known unstable Executor tests that intermittently fail and disrupt CI reliability for other pull requests.
  Flaky tests block the merge of CVE-fixing pull requests and erode confidence in the test suite so stabilizing the most impactful ones is a prerequisite for a reliable release.

The first release under the Linux Foundation should be versioned `4.0.0`.
The last release under LinkedIn was `3.0.4`.
The Jetty 12 and Kafka 4.3 upgrades may introduce behavioral changes that further justify a major version increment.
Continuing forward from `3.0.4` rather than resetting to `1.0.0` avoids ambiguity about which version is newer and preserves version continuity across the transition.
Cruise Control versioning is independent of Kafka versioning so the project can release on its own cadence, and the version number match is coincidental.

### Roadmap (Post-release)

The following phases outline development priorities **after** the first release.
These items are not in scope for the initial release but are documented here to provide visibility into the project's direction and to invite community input.
Each phase is expected to be proposed and discussed in further detail through separate proposals as the project progresses.

#### Phase 1: Foundation and Maintenance

This phase reduces long-standing technical debt and improves long-term maintainability.

- **Fix Vert.x webserver implementation**: Investigate and repair the Vert.x webserver implementation which appears to be broken and may no longer be in active use.
 There is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/2333).

- **Migrate off internal Kafka APIs**: Reduce upgrade friction and improve compatibility across Kafka releases by replacing dependencies on private Kafka APIs with supported public APIs.
  The migration process has already started and is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/2282).

- **Test against multiple Kafka versions**: Update the test suite to run against multiple Kafka versions to improve compatibility coverage and detect version-specific issues earlier.
 This will become much easier once the migration off of internal Kafka APIs work is complete and Test Container support is fully in place.

- **Deprecate stale features**: Review the existing feature set and identify features that are no longer used, relevant, or maintained, then plan their deprecation or removal.

- **Introduce automated dependency management**: Evaluate and adopt Dependabot or Renovate to automate dependency update detection and improve visibility into security fixes.

- **Rename Java packages from `com.linkedin` to `io.cruisecontrol`**: The codebase currently uses the `com.linkedin.kafka.cruisecontrol` package prefix inherited from the project's origin at LinkedIn.
  Now that Cruise Control is under the Linux Foundation, the package namespace should reflect the project's current identity by migrating to `io.cruisecontrol`.

- **Publish to Maven Central**: In addition to JFrog, publish Cruise Control artifacts to Maven Central to improve accessibility, discoverability, and consumption for downstream consumers and align with standard open-source distribution practices.
  Establishing the Maven Central publication pipeline ensures that the validated release process includes the distribution channel most users expect.

#### Phase 2: API and Integration 

This phase improves API consistency, integration tooling, and operational configuration.

- **Improve REST API status codes**: Align HTTP response codes with standard REST conventions to simplify automation, improve error handling, and make API responses more predictable for clients and operators.
 There is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/2357).

- **Expose execution start timestamps in Executor State**: Add structured start-time information to Executor State responses to improve API consistency and simplify task monitoring, duration tracking, and automation. 
 There is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/2271).

- **Generate and publish a Cruise Control client SDK**: Cruise Control already ships its OpenAPI 3.0 spec (as YAML source files) in published artifacts and has the openapi-generator plugin configured in its build, but currently only uses it to resolve the spec into a single JSON file. 
 Configuring the generator to also produce typed Java client models (POJOs) and publishing them as a separate artifact would allow downstream projects to work with typed objects rather than parsing raw JSON.

- **Add PEM-based TLS credential support**: Enable Cruise Control to consume PEM-formatted certificates and private keys directly. This would simplify certificate management, improve compatibility with cloud-native and Kubernetes-based certificate workflows, and reduce the need for keystore conversion and password management.

#### Phase X: Other potential enhancements

These items are candidates for development beyond the initial roadmap phases.

- **Support least-privilege authorization models**: Evaluate mechanisms to reduce Cruise Control's Kafka permissions by replacing blanket superuser access with configurable, operation-specific ACLs where feasible, while preserving support for different authorization implementations and cluster configurations.

- **Post-quantum cryptography support**:  Investigate, evaluate, and develop support for enabling Post Quantum Cryptography within Cruise Control. 
  This will be on both sides: HTTP server side for TLS connections from the clients to the API and the Kafka side, the latter being dependent on upstream Kafka.

- **Improve self-healing status visibility**: Expose the lifecycle and progress of self-healing operations through APIs and notifications, enabling operators to track active fixes, determine when remediation has completed, and integrate more effectively with external automation systems.
  There is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/2215).

- **Add per-anomaly self-healing metrics**: Add per-anomaly self-healing metrics to improve visibility into when healing operations start and finish making it easier to monitor and troubleshoot automated recovery workflows.
 There is already an [open pull request](https://github.com/cruise-control-for-kafka/cruise-control/pull/2268). 

- **Dynamic configuration updates**: Extend support for runtime configuration changes to cover all settings, removing the need to restart Cruise Control to apply updates.

- **Store Cruise Control state**: Persist execution state across restarts to enable recovery of in-progress operations and reduce disruption from crashes or maintenance events.
  This is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/2365).

- **Bulk AdminClient throttle operations**: Improves the performance and scalability of throttle management by batching AdminClient operations, reducing overhead during large-scale cluster changes. 
  There is already an [open pull request](https://github.com/cruise-control-for-kafka/cruise-control/pull/2305).

- **Add TopicLeaderReplicaDistributionGoal**: Improves balancing of topic leadership across brokers to better distribute traffic for high-volume topics without requiring large-scale replica movement. 
 There is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/pull/1751).

- **Add ProduceRequestRateDistributionGoal and FetchRequestRateDistributionGoal**: Introduce goals that balance produce and fetch request rates across brokers, improving workload distribution for clusters where client request volume is a primary source of load.
  There is already an [open issue](https://github.com/cruise-control-for-kafka/cruise-control/issues/859).

- **Prepare for KIP-1150 (Diskless Topics)**: Diskless topics change how Kafka stores data by using object storage as the primary storage solution and local disks as cache only.
  It may be useful to integrate cache metrics to Cruise Control and balance based on the memory state too, next to disks.
  There is an [open KIP](https://cwiki.apache.org/confluence/spaces/KAFKA/pages/345377898/KIP-1150+Diskless+Topics).

## Compatibility

### Kafka compatibility

The Kafka 4.3.0 upgrade establishes the baseline Kafka version for the first release.
Older Kafka client versions are expected to remain compatible at runtime but multi-version build and test coverage is deferred to a future release.

### Jetty compatibility

The Jetty 12 migration is a breaking change from earlier Jetty versions.
Users who depend on Jetty-specific behavior or extensions may need to update their configurations.

## Rejected alternatives

### Deferring the release until more changes are ready

An alternative approach would be to wait until technical debt reduction, API improvements, or other feature work is also complete before cutting a release.
This was rejected because users have been waiting for CVE fixes for an extended period and further delay increases their exposure.
The release process itself also needs to be validated under the new Linux Foundation structure and the sooner that happens the sooner subsequent releases can be shipped with confidence.
A narrow, focused first release minimizes risk and gets fixes to users as quickly as possible.
