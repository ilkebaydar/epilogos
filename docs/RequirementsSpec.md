# Software Engineering — Requirements Specification

## Cover Page
| Field | Details |
|---|---|
| **Course** | BLG411E — Software Engineering |
| **Report name** | Requirements Specification (SRS) |
| **Project title** | Epilogos — Intelligent Post-Meeting Workflow Platform |
| **Group name / number** | 4 Bytes |
| **Group members** | Furkan Kural — 150210056, İlke Başak Baydar — 150140709, Muhammed Yunus Doğru — 150210092, Mehmet Enes Tekgöz — 150210089, Abdul Akhundzada — 150220926 |
| **Instructors** | Ayşe Tosun Kühn, Tolga Ovatman |
| **Submission date** | 2026-11-08 |
| **Document version** | 1.0 |

### Revision history
| Version | Date | Change | Author / approver |
|---|---|---|---|
| 0.9 | 2026-11-03 | First draft derived from the Project Plan (`docs/Project_Plan.md`) and the Project Charter. Open points are marked **TBD** and listed in Appendix A. | Drafted with an AI assistant; to be reviewed by the team |
| 0.10 | 2026-11-04 | Security and data changes proposed by the Security & Database Administrator: proposals for E-06, E-10 and E-14; wider masking scope; handling of LLM output; Jira assignee selection; export idempotency; sharing and recording-retention requirements (FR-41 to FR-44); new NFR-17 to NFR-20; new scenarios SC-30 to SC-42; new open questions E-15 to E-18. **Not yet approved by the team.** | Abdul Akhundzada (proposal, with an AI assistant); to be reviewed by the team |
| 0.11 | 2026-11-05 | Contradictions removed: registration, masking types and the recall target aligned with the Project Plan; NFR-04 made consistent with NFR-03; `exported` item state added; automatic recording deletion and login rate limiting removed (not feasible on the free hosting tier), manual deletion kept; E-09 and E-16 closed; scenario added for FR-18; references and traceability fixed. | Abdul's proposals and İlke's review, applied with an AI assistant; to be approved by the team |
| 0.12 | 2026-11-05 | E-10 answered: open self-registration with the role Member; the first Admin is created during system setup (FR-01, SC-44). FR-11 increment split according to the Plan: status stored in v0.2 (WBS-3.2), shown in v0.3 (WBS-3.3). | Team decision (İlke), applied with an AI assistant; to be approved by the team |
| 0.13 | 2026-11-05 | Realistic targets chosen by the team: masking recall ≥ 90% on the seeded-secret set of standard-format values, precision measured and reported without a target (NFR-03, E-15 closed). An Admin can change user roles (FR-03, SC-45, E-10 closed). The Plan revision must change the 95% target in §1.4, D-06, WBS-7.1 and R-04. | Team decision (İlke), applied with an AI assistant; to be approved by the team |
| 0.14 | 2026-11-05 | Realistic answers applied to E-01, E-02, E-03, E-05, E-11, E-12 and E-17: task/sub-task rule (FR-32), priority values and optional assignee (FR-22), decision fields (FR-23), meeting fields at upload (FR-09), token lifetimes (FR-02), usability success measure (NFR-10), minimum password length (NFR-17). Scenarios SC-46 to SC-50 added. E-07 closed with the proposed targets (NFR-14, NFR-15). Remaining open: E-08 and E-13 (instructor/customer) and the editing part of E-04. | Team decision (İlke), applied with an AI assistant; to be approved by the team |
| 1.0 | 2026-11-08 | E-04 answered (editing rules, FR-27, SC-51, SC-52); E-13 answered with the customer-request process (Appendix B); E-08 recorded as an assumption until instructor feedback (CON-05); proposals of v0.10–v0.14 approved; released as `requirements-spec-v1.0`. | 4 Bytes team |

### Conventions
- **shall** = mandatory, **should** = desirable, **may** = optional, **will** = statement of purpose.
- Every requirement has a unique ID, a **source** and a **rationale**. Sources refer to the Project Plan (*Plan*) by section, task (`T-##`) or WBS activity (`WBS-#.#`), to the Project Charter (*Charter*), or to an elicitation item (`E-##`). Requirements that were added after the Project Plan cite the elicitation item they come from (`E-##`) and are mapped to an existing WBS activity in §7 (Plan revision notes).
- Requirements describe *what* the system does, not *how* it is built. Technology choices are recorded in the Project Plan and will be detailed in the Design Specification.
- **TBD** marks a point that is not yet decided. It is not filled with an assumption; the open question is listed in Appendix A.

---

## 1. Introduction
### 1.1 Purpose

This document specifies the functional and non-functional requirements of Epilogos. It is the reference for the customer (to confirm that this is what is needed), for the team (what to build and test) and for the AI coding assistants used by the team (what to generate). Acceptance criteria are written as Gherkin scenarios so that they can be turned directly into acceptance tests in `tests/` (Plan WBS-13.1).

### 1.2 Scope

Engineering teams make technical decisions and assign follow-up tasks in meetings, but many of these outcomes never reach the backlog. Epilogos lets a user upload a recorded meeting. The system produces a timestamped transcript with speaker labels, masks secrets and personal data, extracts the decisions and action items, and lets the user review and approve them. Approved action items are exported to Atlassian Jira in one click (Plan §1.1, Charter §1).

Out of scope (Plan §1.1, Charter §3): live call bots and real-time transcription; biometric speaker enrollment and cross-meeting speaker identity; two-way Jira synchronization; native mobile applications; issue trackers other than Jira; paid, always-on production hosting.

### 1.3 Glossary
| Term | Definition |
|---|---|
| Meeting | A record created for one uploaded recording. It holds the recording, its processing job, its transcript and its extracted items. |
| Recording | An audio or video file of a meeting that ended before upload (MP4, MKV or WAV). |
| Job | The background processing of one recording, from upload until its items are ready or it fails. |
| Job status | One of `queued`, `preprocessing`, `transcribing`, `diarizing`, `masking`, `extracting`, `completed`, `failed` (Plan T-03). |
| Transcript segment | A part of the transcript with a start time, an end time and text. In local mode it also has a speaker label. |
| Speaker label | A name for one speaker inside one meeting, initially generated (e.g., "Speaker 1") and renamable by the user. It has no meaning outside its meeting. |
| Sensitive data | API keys, access tokens, passwords and e-mail addresses that appear in a transcript (Plan T-07), including values that are introduced by a spoken cue (for example "the password is ..."). |
| Masking | Replacing sensitive data in the transcript so that the original value is not stored, sent to an LLM or exported. Masking is irreversible: the original values are not kept anywhere in the system. |
| Action item | An extracted item with a title, a description, a priority (High, Medium or Low) and an optional assignee (Plan T-08, FR-22). |
| Decision | An extracted statement of a decision made in the meeting, with an optional rationale (FR-23). |
| Item state | The state of an action item: `pending`, `approved`, `rejected` or `exported` (FR-27). |
| Jira connection | The Jira Cloud site, credentials, project and issue type configured by an Admin. |
| Inference mode | `local` (all processing on the team's machine) or `cloud` (transcription and extraction through free cloud APIs), selected by configuration. The dual-mode deployment is pending instructor feedback (E-08). |
| Role | Admin, Member or Viewer (see §2.2). |
| Owner | The user who uploaded the recording of a meeting. |
| Share | A permission with which the owner of a meeting gives another user access to that meeting. |
---

## 2. Overall Description

### 2.1 Product perspective

### 2.2 Users and roles

### 2.3 Assumptions and dependencies

### 2.4 Constraints

---

## 3. Functional Requirements

### 3.1 Accounts and access control (T-01)

### 3.2 Recording upload (T-02)

| ID | Requirement | Source | Rationale |
|---|---|---|---|
| FR-07 | An Admin or Member **shall** be able to upload a recording in MP4, MKV or WAV format. | Plan T-02 | Formats listed in the scope. |
| FR-08 | The system **shall** reject a file in any other format, a file larger than 1 GB, or a recording longer than 90 minutes, and **shall** tell the user which rule was violated. | Plan T-02, CON-02 | Limits stated in the plan; the user must know how to fix the upload. |
| FR-09 | For every accepted upload the system **shall** create a meeting linked to the recording and to the uploading user. The user **shall** enter a title, and **may** enter the meeting date (if it is empty, the upload date is used) and, in local mode, the number of speakers. An upload without a title **shall** be rejected. | Plan T-02, WBS-6.1, E-05 | The title identifies the meeting in lists. The number of speakers is the speaker-count hint of WBS-6.1 and improves diarization. |

### 3.3 Background processing and progress (T-03)

| ID | Requirement | Source | Rationale |
|---|---|---|---|
| FR-10 | Every accepted upload **shall** start a background job; the user **shall not** have to keep the upload page open until processing ends. | Plan T-03 | Processing can take as long as the meeting itself. |
| FR-11 | The system **shall** store the job status (see Glossary) and show the current status to the user. While a job is `queued`, the system **shall** show its position in the queue. | Plan T-03, §1.4 Usability | The user must know when results will be ready. |
| FR-12 | When several recordings are submitted, the system **shall** queue them and process them one after another without any of them failing because of the others. | Plan §1.1 Concurrency, §1.4 Performance | A single GPU cannot run several jobs at once. |
| FR-13 | If a processing stage fails, the system **shall** retry it up to 3 times. If it still fails, the job **shall** become `failed` and the user **shall** see an error message that names the failed stage. | Plan §1.4 Reliability | Temporary errors should not lose the job; permanent errors must be visible. |
| FR-14 | An Admin or Member **shall** be able to re-submit a failed job. | Plan §1.1 Queue broker | Allows recovery without uploading the file again. |

### 3.4 Transcription and speakers (T-04, T-05, T-06)

### 3.5 Masking of sensitive data (T-07)

### 3.6 Extraction of action items and decisions (T-08)
| ID | Requirement | Source | Rationale |
|---|---|---|---|
| FR-22 | After masking, the system **shall** extract action items from the transcript. Each action item **shall** have a title, a description and a priority, and **may** have an assignee. The priority **shall** be High, Medium or Low; if it cannot be determined from the transcript, it **shall** be Medium. | Plan T-08, E-02 | These fields map to a Jira issue. The three values exist with the same names in Jira's default priority scheme. |
| FR-23 | The system **shall** extract the decisions made in the meeting. Each decision **shall** have a statement and **may** have a rationale. | Plan T-08, Charter §1, E-03 | Capturing decisions is a project objective. Decisions are only read in Epilogos (FR-28), so few fields are needed. |
| FR-24 | The system **shall** validate every extracted item against the defined item schema and **shall** present only items that pass validation. | Plan T-08, D-03 | Invalid items cannot be reviewed or exported reliably. |

### 3.7 Review and approval (T-09)

### 3.8 Jira integration (T-10)
| ID | Requirement | Source | Rationale |
|---|---|---|---|
| FR-29 | An Admin **shall** be able to configure the Jira connection (Jira Cloud site, account, API token, target project and issue type), and **shall** be able to test it. The test **shall** report success or the reason for failure. | Plan WBS-10.1 | Export problems should be found before the first export. |
| FR-30 | The system **shall** store Jira credentials encrypted, never in plain text. | Plan WBS-1.3, D-06 | The API token gives write access to the team's Jira. |
| FR-31 | An Admin or Member **shall** be able to export the approved action items of a meeting to the configured Jira project in one action. Items that are not approved **shall not** be exported. | Plan T-10, D-04 | One-click export is the main value of the product. |
| FR-32 | The system **shall** create each exported action item as a Jira task, with the Jira priority of the same name (FR-22). During review the user **may** choose a parent for an item: an issue that already exists in Jira (for example a task exported earlier from the same meeting). An item with a parent **shall** be created as a sub-task of that issue. | Plan T-10, E-01 | Tasks and sub-tasks are in the scope. A parent chosen by the user is simpler and more reliable than grouping by the LLM. |
| FR-33 | During review, the system **shall** let the user choose the Jira assignee of an item from a searchable list of the users of the target Jira project. The system **should** suggest a user by comparing the item's assignee name (the renamed speaker label in local mode, or the name extracted by the LLM in cloud mode) with the Jira users. If no user is chosen, the issue **shall** be created unassigned. | Plan T-10, WBS-10.1 | Jira needs an account, not a free-text name, and name matching alone is unreliable. The assignee does not need an Epilogos account. |
| FR-34 | The system **shall** store the Jira issue key of every exported item, and **should** show it next to the item. | Plan WBS-10.2 | Links the item to its issue and supports FR-35. |
| FR-35 | Exporting the same meeting again **shall not** create duplicate Jira issues for `exported` items (FR-27). This **shall** also hold when an earlier export stopped after Jira created an issue but before its key was stored. | Plan D-04 | Users may retry after a partial failure, and a crash can happen between the Jira call and the database write. |
| FR-36 | If Jira returns an error or a rate-limit response, the system **shall** retry, and if the export still fails it **shall** tell the user which items were not exported. | Plan §1.4 Reliability, WBS-10.2 | Partial failures must be visible and recoverable. |

### 3.9 Meeting dashboard and synchronized playback (T-11)

### 3.10 Meeting access and data lifecycle

---

## 4. Non-functional Requirements

---

## 5. User Stories and Acceptance Criteria

### 5.1 User stories

### 5.2 Acceptance criteria (Gherkin)

```gherkin
Feature: Accounts and access control   # US-01, US-02
```

```gherkin
Feature: Recording upload   # US-03

  @SC-05 @FR-07 @FR-09
  Scenario: A valid recording is accepted
    Given a logged-in Member
    When the Member uploads a 20-minute MP4 file of 300 MB with the title "Sprint 12 planning"
    Then the system creates a meeting linked to the file and to the Member
    And a processing job is started for it

  @SC-49 @FR-09
  Scenario: An upload without a title is rejected
    Given a logged-in Member
    When the Member uploads a valid MP4 file without a title
    Then the upload is rejected
    And the message names the missing title

  @SC-06 @FR-08
  Scenario Outline: An invalid recording is rejected with the reason
    Given a logged-in Member
    When the Member uploads <file>
    Then the upload is rejected
    And the message names the rule "<rule>"

    Examples:
      | file                         | rule        |
      | a PDF file                   | format      |
      | a 1.2 GB MKV file            | file size   |
      | a 95-minute WAV file         | duration    |
```

```gherkin
Feature: Background processing and progress   # US-04

  @SC-07 @FR-10 @FR-11
  Scenario: The user follows the job status
    Given a Member uploaded a valid recording
    When the Member opens the meeting
    Then the current job status is shown
    And the status changes until it is "completed" or "failed"

  @SC-08 @FR-11 @FR-12
  Scenario: Several uploads wait in the queue
    Given one job is being processed
    When a Member uploads two more valid recordings
    Then both new jobs are "queued" with queue positions 1 and 2
    And all three jobs end as "completed"

  @SC-09 @FR-13
  Scenario: A stage that keeps failing marks the job as failed
    Given the transcription stage fails on every attempt
    When the job is processed
    Then the stage is attempted at most 3 more times
    And the job becomes "failed"
    And the error message names the "transcribing" stage

  @SC-10 @FR-14
  Scenario: A failed job is re-submitted
    Given a job in status "failed"
    When a Member re-submits the job
    Then the job is "queued" again without a new upload
```

```gherkin
Feature: Transcript and speakers   # US-05
```

```gherkin
Feature: Masking of sensitive data   # US-06
```

```gherkin
Feature: Extraction of action items and decisions   # US-07
```

```gherkin
Feature: Review and approval   # US-08
```

```gherkin
Feature: Jira integration   # US-09, US-10
```

```gherkin
Feature: Meeting dashboard and synchronized playback   # US-11
```

```gherkin
Feature: Meeting access and data lifecycle   # US-12, US-13
```

---

## 6. Use Cases

### UC-01: Process a meeting recording

### UC-02: Review and export action items to Jira (critical)

### UC-03: Share a meeting and delete its recording

---

## 7. Traceability

Every requirement links back to the Project Plan (task and WBS activity) and forward to a user story and its acceptance scenarios. Tests named `test_SC##_...` and code references will be added to this matrix as development progresses. The increment column follows the increment plan in Project Plan §5.3 and will change if the plan revision changes it.

| Req. | Plan task | WBS activity | Story | Scenario(s) | Planned increment |
|---|---|---|---|---|---|
| FR-01 | T-01 | WBS-1.2 | US-01 | SC-01, SC-37, SC-44 | v0.2 |
| FR-02 | T-01 | WBS-1.2 | US-01 | SC-02, SC-50 | v0.2 |
| FR-03 | T-01 | WBS-1.1, 1.2 | US-02 | SC-03, SC-45 | v0.2 |
| FR-04 | T-01 | WBS-1.2 | US-02 | SC-04 | v0.2 |
| FR-05 | T-01 | WBS-1.2, 9.1 | US-02 | SC-03 | v0.2 |
| FR-06 | T-01 | WBS-1.2 | US-02 | SC-02, SC-03, SC-04, SC-45 | v0.2 |
| FR-07 | T-02 | WBS-2.1 | US-03 | SC-05 | v0.1 |
| FR-08 | T-02 | WBS-2.1 | US-03 | SC-06 | v0.1 |
| FR-09 | T-02 | WBS-2.1 | US-03 | SC-05, SC-49 | v0.1 |
| FR-10 | T-03 | WBS-3.2 | US-04 | SC-07 | v0.2 |
| FR-11 | T-03 | WBS-3.2, 3.3 | US-04 | SC-07, SC-08 | v0.2 (status stored, WBS-3.2); v0.3 (status and queue position shown, WBS-3.3) |
| FR-12 | T-03 | WBS-3.2 | US-04 | SC-08 | v0.2 |
| FR-13 | T-03 | WBS-3.2, 3.4 | US-04 | SC-09 | v0.4 |
| FR-14 | T-03 | WBS-3.2 | US-04 | SC-10 | v0.2 |
| FR-15 | T-04 | WBS-4.1 | US-05 | SC-11 | v0.1 |
| FR-16 | T-05 | WBS-5.1 | US-05 | SC-11 | v0.2 |
| FR-17 | T-06 | WBS-6.1, 6.2 | US-05 | SC-12 | v0.3 |
| FR-18 | T-06 | WBS-6.2 | US-05 | SC-43 | v0.3 |
| FR-19 | T-11 | WBS-11.2 | US-05 | SC-13 | v0.3 |
| FR-20 | T-07 | WBS-7.1 | US-06 | SC-14 | v0.3 |
| FR-21 | T-07 | WBS-7.1, 3.4 | US-06 | SC-15, SC-39 | v0.3 |
| FR-22 | T-08 | WBS-8.1, 8.2 | US-07 | SC-16, SC-48 | v0.3 |
| FR-23 | T-08 | WBS-8.1, 8.2 | US-07 | SC-16 | v0.3 |
| FR-24 | T-08 | WBS-8.1, 8.2 | US-07 | SC-17 | v0.3 |
| FR-25 | T-09 | WBS-9.1, 9.2 | US-08 | SC-18 | v0.4 |
| FR-26 | T-09 | WBS-9.1, 9.2 | US-08 | SC-19, SC-20 | v0.4 |
| FR-27 | T-09 | WBS-9.1, 10.2 | US-08 | SC-19, SC-20, SC-51, SC-52 | v0.3 |
| FR-28 | T-09 | WBS-9.2 | US-08 | SC-18 | v0.4 |
| FR-29 | T-10 | WBS-10.1 | US-09 | SC-21 | v0.3 |
| FR-30 | T-01 | WBS-1.3 | US-09 | SC-22 | v0.3 |
| FR-31 | T-10 | WBS-10.2 | US-10 | SC-23 | v0.4 |
| FR-32 | T-10 | WBS-9.2, 10.2 | US-10 | SC-46, SC-47 | v0.4 |
| FR-33 | T-10 | WBS-10.1, 9.2 | US-10 | SC-24, SC-41 | v0.4 |
| FR-34 | T-10 | WBS-10.2 | US-10 | SC-23 | v0.4 |
| FR-35 | T-10 | WBS-10.2 | US-10 | SC-25, SC-42 | v0.4 |
| FR-36 | T-10 | WBS-10.2 | US-10 | SC-26 | v0.4 |
| FR-37 | T-11 | WBS-11.1, 1.1, 1.2 | US-11, US-12 | SC-27, SC-30, SC-33 | v0.2 |
| FR-38 | T-11 | WBS-11.2 | US-11 | SC-28, SC-34 | v0.3 |
| FR-39 | T-11 | WBS-11.2 | US-11 | SC-28 | v0.3 |
| FR-40 | T-11 | WBS-11.2 | US-11 | SC-29 | v0.3 |
| FR-41 | T-01 | WBS-1.1, 1.2 | US-12 | SC-31, SC-32 | Set in the Plan revision |
| FR-42 | T-02 | WBS-2.1 | US-13 | SC-34, SC-35 | Set in the Plan revision |
| FR-43 | T-02 | WBS-2.1 | US-13 | SC-36 | Set in the Plan revision |
| FR-44 | T-01 | WBS-1.2 | US-12 | SC-30 | v0.2 |

Non-functional requirements are verified by the tests and measurements listed in §4. Their results are reported in the Software Test Report (WBS-13.4).

**Coverage:** all 44 functional requirements have at least one acceptance scenario. NFR-17 to NFR-20 are covered by SC-38, SC-30, SC-40 and SC-34.

**Plan revision notes:** the requirements below are not yet written in the Project Plan. Each one is added to an existing WBS activity, so the WBS, the estimates and the schedule do not change; the Plan revision only has to add them to the activity descriptions.

| Requirement | Added to | Note |
|---|---|---|
| FR-33 (Jira user list) | WBS-10.1 (user fetch), WBS-9.2 (selection on the review page) | WBS-10.1 already includes the Jira user search. |
| FR-41 (sharing), FR-44 | WBS-1.1, WBS-1.2 | Sharing control on the meeting page: WBS-11.1. |
| NFR-17 (minimum password length), NFR-18 (ownership checks) | WBS-1.2 | |
| NFR-19 (instructions inside a transcript) | WBS-8.1 (prompt tests) | |
| FR-42, FR-43, NFR-20 (manual deletion) | WBS-2.1 | Delete buttons: WBS-11.1. |
| FR-27 (`exported` state) | WBS-9.1, WBS-10.2 | |
| FR-32 (parent choice for sub-tasks) | WBS-9.2 (field on the review page), WBS-10.2 | |
| FR-09 (title, meeting date, number of speakers) | WBS-2.1, WBS-11.1 (upload form) | The number of speakers is the hint already planned in WBS-6.1. |

---

## Appendix A. Open Questions (Elicitation)

---

## Appendix B. Customer Requests and Change Requests
