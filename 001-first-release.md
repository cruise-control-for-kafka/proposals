# First Release Under the Linux Foundation

This proposal defines the scope and priorities for the first Cruise Control release under the Linux Foundation.

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
Broader improvements to the codebase, APIs, and feature set, including the Java package rename from `com.linkedin` to `io.cruisecontrol` and Maven Central publication are deferred to subsequent releases.

- **Upgrade to Kafka 4.3**: Upgrade Cruise Control to Kafka 4.3 to ensure compatibility with the latest Kafka version and resolve outstanding Kafka-related CVEs.
  This work has already been [merged](https://github.com/cruise-control-for-kafka/cruise-control/pull/2342).

- **Upgrade to Jetty 12**: Migrate from pre-v12 Jetty to Jetty 12 because earlier versions are end-of-life and no longer receive security patches.
  This work has already been [merged](https://github.com/cruise-control-for-kafka/cruise-control/pull/2307).

- **Remove duplicate and conflicting dependency declarations**: Audit the Gradle build configuration to eliminate redundant version pins that are silently overridden by transitive dependencies.
  Introduce a version BOM (`platform()`/`enforcedPlatform()`) to centralize dependency version management across subprojects.
  Without this there is no reliable way to confirm that upgrading a dependency version actually takes effect across the entire build, which undermines confidence that CVE-affected versions have been fully replaced.

- **Address remaining critical CVEs**: Upgrade or pin any remaining dependencies with known critical CVEs that are not resolved by the Kafka or Jetty upgrades (e.g. Log4j, Jackson, Commons Beanutils). 
  This may involve adding dependency constraints or forcing specific versions to ensure CVE-affected transitive dependencies are fully replaced.

- **Fix known flaky tests**: Assess known flaky tests and fix the ones deemed to be most impactful.
  There has already been a [PR merged](https://github.com/cruise-control-for-kafka/cruise-control/pull/2338) that addresses known unstable Executor tests that intermittently fail and disrupt CI reliability for other pull requests.
  Additional unstable tests are currently being identified and addressed.
  Flaky tests block the merge of CVE-fixing pull requests and erode confidence in the test suite so stabilizing the most impactful ones is a prerequisite for a reliable release.

The first release under the Linux Foundation will be versioned `3.1.0`, continuing from LinkedIn's final release of `3.0.4`. 
This preserves version continuity across the transition and avoids ambiguity about which release is newer.
It also distinguishes the Linux Foundation release line from LinkedIn's internal release line of `3.0.x`.

The target release date is in the first half of October.

## Compatibility

### Kafka compatibility

The Kafka 4.3 upgrade establishes the baseline Kafka version for the first release.
Older Kafka client versions are expected to remain compatible at runtime but multi-version build and test coverage is deferred to a future release.

### Jetty compatibility

The Jetty 12 migration from Jetty 9 involves many code changes due to package renaming but the behavior exposed to Cruise Control users is expected to remain unchanged.

## Rejected alternatives

### Deferring the release until more changes are ready

An alternative approach would be to wait until technical debt reduction, API improvements, or other feature work is also complete before cutting a release.
This was rejected because users have been waiting for CVE fixes for an extended period and further delay increases their exposure.
The release process itself also needs to be validated under the new Linux Foundation structure and the sooner that happens the sooner subsequent releases can be shipped with confidence.
A narrow, focused first release minimizes risk and gets fixes to users as quickly as possible.
