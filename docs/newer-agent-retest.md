# O-6: retesting a newer Agent version

**Status: proposed protocol; not executed.** [O-6](../README.md#9-open-items-and-limitations)
remains **Not tested**. This document adds no measurements or evidence grades.
It describes one bounded comparison for one explicitly selected candidate newer
than 8.343.1.1038, without assuming that the candidate fixes the reported stall.

## Required inputs and build compatibility

The public repository contains the report, not the Reproducer or measurement
harnesses. [E-05](../README.md#12-evidence-inventory) is held separately by the
investigators. Obtain authorised access before claiming an E-05 replication.
Otherwise, pin a separately implemented reproducer and document its differences;
its results are a new experiment, not a rerun of unavailable evidence bundles.

Before measurements, record the source commit and source-tree SHA-256 manifest,
exact candidate plugin version, resolved Agent runtime version, dependency locks,
Gradle/JDK/AGP/Kotlin/Compose versions, build variant and instrumentation settings.
Use the same application source, dependencies and UI topology in all three arms:

| Arm | Build | Purpose |
|---|---|---|
| A | Agent instrumented with plugin 8.341.1.1004 | Positive control for the reported stall |
| B | Agent instrumented with the pinned candidate plugin | Version under test |
| C | Agent and instrumentation plugin removed | Same-source negative control |

Retain the exact build/configuration diffs; allow only the changes defining each
arm. Keep beacon and runtime capture settings identical between A and B, using
the Reproducer's dummy-beacon arrangement, without production credentials/data.
If any arm cannot compile, launch or render the same test content, record a
**compatibility blocker**, including the failed command and diagnostic. Do not
count that arm as stall-free. A required dependency/source migration changes the
experiment: pin a new common baseline, rebuild all arms and revalidate controls.

## Establish that the intended instrumentation is present

Following [F-05](../README.md#62-mechanism) and the
[method summary](../README.md#11-method-summary), retain clean build logs using
`--no-build-cache --rerun-tasks`, the actual transform-task execution, and APK
SHA-256 values. Verify the installed APK identity before collecting a run.
Inspect decoded bytecode in every arm and retain the relevant excerpts/diffs:

- A: confirm the `NodeCoordinator.hit` → `onNodeCoordinatorHit` path.
- B: locate the candidate's actual Compose interaction path; record whether the
  old hook is present, relocated, replaced or absent. Do not assume old symbols.
- C: confirm removal of the Agent runtime and its woven interaction path.

Define and record how active Compose interaction capture will be established
for B. A missing old hook alone establishes neither successful instrumentation
nor a performance fix. If intended capture cannot be confirmed, label B
**instrumentation not established** and withhold a version-fix conclusion.

## Predeclare the runs, then collect paired measurements

Fix the screen/content, overlap topology, viewport and starting scroll position.
Record device model, ABI, Android build, refresh rate, emulator configuration if
applicable, and thermal/power/host-load conditions. Record an independent layout
census where available; a different workload requires a separate comparison.

Treat each device and gesture method as a separate condition. Specify gesture
coordinates, duration, input tool and reset procedure. Do not pool `adb input
swipe` with the real-gesture method: [F-09](../README.md#62-mechanism) measured
different pass counts. An emulator-only run says nothing about untested phones.

Before collecting data, fix the gesture window, stall threshold (80 ms for the
audit's DTSTALL), sampling settings, repeat count, invalid-run criteria, and
criteria for positive-control reproduction and candidate absence. Use three or more paired
rounds of A/B/C per condition, in a balanced predeclared order (e.g. ABC, BCA, CAB).
Restore the same initial state before each arm. After each process launch, retain
the first gesture as warm-up evidence and exclude it from steady-state summaries;
report it separately. Reset the viewport before every measured gesture; retain zero-stall runs.
Record invalid runs and reasons, and repeat the complete affected paired round.

For each gesture, retain raw timestamps, verified scroll displacement and:

- main-thread CPU delta over the same window (`schedstat`, as in the audit);
- independently measured peak main-thread stall and sampled stack attribution;
- Agent pass count, N and `calc_ms` when the corresponding probes are available;
- optional `gfxinfo`/Perfetto output as supporting evidence, with tool settings.

Apply [O-12](../README.md#9-open-items-and-limitations): a smooth callback-gap
reading alone cannot establish absence of a stall. Preserve raw stacks and the
version-specific attribution rule; renamed/unresolved frames are not evidence
of zero Agent work. Mark missing metrics **unavailable**, not zero. Document
probe modifications/overhead and separate instrumented-probe from unmodified runs.

## Interpret and retain the result

Check A and C before interpreting B, using the independent CPU, stall and stack
measurements described in [F-01/F-02](../README.md#61-causal-attribution).

| Observation | Permitted conclusion |
|---|---|
| A fails the predeclared reproduction criterion, C has unexplained comparable stalls, or required measurements are missing | Inconclusive; investigate control/measurement validity |
| An arm cannot build/run, or B's intended instrumentation is not established | Blocked/inconclusive; not evidence of a fix |
| Controls are valid and B shows the Agent-attributable stall | Behaviour persists for this version and condition |
| Controls are valid, intended capture is confirmed, and B meets the predeclared absence criterion | Steady-state stall not observed in these runs; report first-touch stalls separately; no claim about other versions, devices or workloads |

Retain the protocol as fixed before collection, source/build manifests, APK
hashes, bytecode evidence, tool versions, per-run raw data, warm-ups and exclusions.
Summarise paired per-gesture CPU/stall values and observed ranges; keep each
device/input condition separate. Record failed and inconclusive candidates too.
Provide evidence references accessible to the reviewer, with confidential assets
kept under their existing access terms; do not attach proprietary APKs publicly.

Any future O-6 update must name the exact tested version and environment, link
the evidence and justify its grade under the [grading scheme](../README.md#4-evidence-grading-scheme).
This protocol alone changes neither O-6's status nor the historical findings.
