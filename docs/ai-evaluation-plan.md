# AI evaluation plan

**Status:** Draft 0.1, 27 October 2026. No model has been selected, no footage has been labelled, and no metric in this file has been measured.

This plan is the gate between "a detector returned a box" and any statement about quality or camera capacity. Numeric acceptance thresholds are intentionally absent. OD-018 forbids inventing them.

## Purpose

Release 1 answers a narrow question:

> On this camera, did a person cross into this restricted zone under the configured rule?

It does not answer who the person is, what they intend, or whether the site is safe.

Two different qualities must not be collapsed into one number:

1. **Detection and tracking quality.** Boxes and track continuity.
2. **Rule quality.** Whether the zone, reference point, persistence, and cooldown turn those tracks into events an operator can use.

A good detector with a bad polygon will still fail the pilot.

## What Release 1 will not evaluate

- Facial recognition, re-identification, demographics, or emotion. Those outputs are out of scope and should fail a review if they appear.
- Generic "suspicious behavior" or an anomaly score. That is Release 2 and only for a named use case.
- Natural-language search.
- A leaderboard claim against a public dataset as if it were the customer's warehouse.

Public datasets may be used for developer debugging if their licenses allow commercial development. They are not acceptance evidence for a pilot site.

## Model adapter requirements

Before a candidate model is measured:

- It implements the person-detector adapter in [architecture.md](architecture.md).
- The weight license and the code license have been read and recorded (OD-004). AGPL code is not added by default.
- The file checksum and version are what the event row will store.
- A fake detector exists for rule tests so geometry is not blocked on the model.

If several models are compared, the comparison uses the same clips, the same zones, and the same sample rate. Changing three variables at once is not a comparison.

## Dataset requirements

Define the dataset **before** looking at a score and deciding whether it is "good enough".

### Minimum labelled set for a pilot site

For each pilot camera:

- At least 20 scripted entries into the zone, across the lighting conditions the site actually has. If the site has a night mode, include it. Twenty is a starting floor for a first look, not a statistically sufficient safety case.
- Crossings near the polygon edge, not only through the middle.
- At least 10 periods of people outside the zone who must not generate an entry.
- At least 5 cases of partial occlusion or two people, if the site can produce them safely.
- At least one hour of ordinary activity, or the real operating pattern, labelled for unexpected entries. Scripted walks overstate performance if they are the only evidence.
- Periods with no people, to estimate false alerts on an empty scene.
- A camera disconnect and reconnect during the hour, used for health metrics rather than for detection precision.

Do not stage trespassing on someone else's property. Use authorized staff who know they are in a test.

### Ground truth

A person with access to the footage marks:

| Label | Meaning |
| --- | --- |
| True entry | A person moved from outside the zone to inside it, using the same zone polygon as the product |
| True non-entry | People visible but no qualifying entry |
| Ignore | The reviewer cannot tell, or the test was invalid |

Ambiguous cases stay ambiguous. They are not forced into true or false to inflate a score.

Ground truth is stored outside the product's training path. Release 1 has no training path. The labels are for evaluation. Record who labelled, when, and which video file and time range.

### Splits

If the same clips are later used to tune thresholds and to claim a result, the claim is contaminated. For the pilot:

- Use one labelled collection to choose confidence, persistence, and cooldown.
- Use a later collection, recorded after those settings were frozen, to report the numbers the owner will see.

If that second collection does not exist, the report must say "tuning set only; not a confirmation set."

Release 2 dataset rules are stricter and are specified in [SRS.md](SRS.md) under AI-010. They are not required to start the pilot measurement.

## Metrics

Report each metric with the camera, the time window, the model version, the sample rate, and the hardware.

| Metric | Definition |
| --- | --- |
| Precision of entry events | True entry events divided by all entry events the system created, on the labelled window. Duplicate alerts for one entry count as false if the rule promised one |
| Recall of entries | True entries that produced at least one timely event, divided by all ground-truth entries |
| False alerts per camera-hour | Entry events that are not true entries, divided by hours of processed video. This is the number operators feel |
| Missed entries | Ground-truth entries with no matching event. List them. Do not hide them in a single percentage |
| Duplicate rate | Extra events for one ground-truth entry |
| Event latency | Defined in the measurement boundary below |
| Frames received, sampled, inferred, dropped | Counters from the worker |
| Evidence capture success | Events with a snapshot divided by events. Also list failures |
| Reconnects | Count and duration of gaps |
| Resource use | CPU, RAM, GPU if any, and disk growth |

Matching rule for a true entry: an event for the same camera and zone whose server frame time falls within a window around the labelled crossing. Proposed matching window is 5 seconds plus the configured persistence. Freeze the window in the report so it is not adjusted after seeing misses.

If the ground truth is too small, publish counts, not a percentage with false precision. "4 of 5 scripted entries" is clearer than "80% accurate."

### Measurement boundary for latency

Provisional target from the product brief, **to be tested, not promised:** 95th percentile under 2 seconds.

Clock start: the time the server finishes receiving the frame that first satisfies the persistence rule.

Clock stop: the time the event row is committed and readable through the API.

This excludes how long the person has already been in the zone before persistence is satisfied. It excludes operator reaction. It excludes network delay from the camera to the server, which must be reported separately if it is large. If the stream does not provide a trustworthy capture clock, do not invent one. Use server receive time.

The target applies only to the documented hardware, codec, resolution, sample rate, and model. A result on a laptop does not transfer to a different server.

### Provisional workload

| Profile | Use |
| --- | --- |
| Two authorized test cameras | First integration and failure tests. AC-001 |
| Five cameras, 720p, 10 FPS input, inference sampling configured down as required | A capacity experiment. Not a commitment that five cameras will pass |

Both profiles use the same software build. Record the exact settings. If five cameras only pass because inference was reduced to one frame every few seconds, the report says that, because zone timing changes.

## Benchmark environment

Record this sheet before the run. Empty cells are not "typical" values.

| Item | Value |
| --- | --- |
| Date and software build or commit | |
| CPU model and core count | |
| RAM | |
| GPU and driver, or "none" | |
| Disk type | |
| Operating system | |
| Container image digests | |
| Camera make, model, firmware | |
| Resolution, codec, FPS, bitrate | |
| Network path from camera to server | |
| Model file name, version, checksum, runtime | |
| Confidence, persistence, cooldown, sample rate | |
| Zone polygon id | |

No row in this sheet has been filled, because no run has been performed.

## Quality-threshold decision

After the confirmation set is labelled and the sheet is filled, the product owner sets, in writing:

- The minimum recall they can accept for that site, knowing the misses.
- The maximum false alerts per camera-hour the operator will still review.
- The latency percentile they require for that site.
- Whether night, rain, or a dirty lens is in scope. If it was not tested, it is out of scope for the claim.

Until that note exists, marketing and sales do not use a number. A pilot may still proceed with a written statement of observed counts and the limitation that thresholds are not set.

If the first site's false-alert rate makes operators stop looking, the response is to change the zone, the persistence, or the use case. It is not to hide alerts.

## Test types that are not a field trial

These belong in automated tests and do not count as AI quality evidence:

- Synthetic rectangles from the fake detector crossing a polygon. This tests the rule engine.
- A short fixture clip in CI, if license and size allow, as a smoke test that the adapter returns a person box. A smoke test is not a recall measurement.
- Unit tests for cooldown and tracking-loss behavior.

Field numbers always outrank fixture numbers when they disagree.

## Failure and degradation measurements

Run these on the two-camera profile before trusting a capacity number:

| Condition | What to record |
| --- | --- |
| Unplug one camera | The other camera's dropped frames and health. The failed camera's state and reconnect |
| Pause the database | Event failure is visible. No false "healthy and empty" |
| Fill the evidence disk | Events still commit with evidence status failed, or the design's documented alternative. Silent loss is a defect |
| Kill one camera child | Supervisor restarts only that camera |
| Leave it running overnight | Gap count, disk growth, memory growth |

## Model promotion gates

Release 1 has one configured model. Changing it requires:

1. The same evaluation sheet, before and after, on the pilot cameras or the closest labelled set.
2. A written note of license, checksum, and configuration.
3. The previous model file kept until the owner accepts the new one, so rollback is a configuration change, not a research project.
4. Events after the change showing the new version. Old events keep the old version.

Release 2 adds dataset version, split leakage checks, an approval record, and post-deploy monitoring of false-alert rate. Those gates are requirements now so the event schema stores model version from the first event. They are not a training system.

### Post-deploy watching

During a pilot, review weekly:

- False alerts per camera-hour, from operator "False Alert" states plus a sample of "Resolved".
- Missed events the staff noticed by other means. The product cannot see its own misses. Ask the operator.
- Evidence failure count.
- Time spent in `Degraded`, `Reconnecting`, and `Error`.

A rise in false alerts after a model or zone change is a rollback trigger, not a prompt to retrain on that week's mistakes inside the live system.

## Human review

Operators can mark False Alert and Resolved. Those marks are operational labels. They become evaluation labels only if someone copies them into the evaluation set with a definition. Do not silently train on them. Release 1 does not train.

The user interface must say that the event is a possible person detection, not a confirmed intrusion. AC-020.

## Reporting

An evaluation report is a dated document stored with the build identifier. It contains the environment sheet, counts, misses, and limitations. It does not contain a sentence such as "the model is production grade."

This plan, by itself, is not that report.
