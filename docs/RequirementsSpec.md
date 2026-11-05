# Software Engineering — Requirements Specification

## Cover Page

### Revision history

### Conventions

---

## 1. Introduction

### 1.1 Purpose

### 1.2 Scope

### 1.3 Glossary

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

### 3.7 Review and approval (T-09)

### 3.8 Jira integration (T-10)

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
