# Feature Specification: Duplicate Detection

**Feature Branch**: `002-duplicate-detection`
**Created**: 2025-12-31
**Status**: Draft
**Input**: User description: "duplicate detection"
**Dependency**: Requires Photo Scanning Engine (001-photo-scanning) to be completed first

## User Scenarios & Testing *(mandatory)*

### User Story 1 - View Detected Duplicates (Priority: P1)

After a photo library scan completes, a user wants to see which photos have been
identified as duplicates. They navigate to the "Duplicates" section of the dashboard
and see groups of similar photos displayed together, with each group showing
thumbnails of the photos that appear to be duplicates of each other.

**Why this priority**: Users need to see detected duplicates before they can take
any action on them. This is the foundational display capability for the feature.

**Independent Test**: Can be fully tested by running a scan on a library with known
duplicates and verifying the correct photos are grouped together in the UI.

**Acceptance Scenarios**:

1. **Given** a completed scan with duplicate photos detected, **When** the user
   views the Duplicates section, **Then** they see duplicate groups with thumbnail
   previews of each photo in the group.

2. **Given** duplicate groups are displayed, **When** the user views a group,
   **Then** they see how many photos are in the group and the similarity percentage.

3. **Given** no duplicates were detected, **When** the user views the Duplicates
   section, **Then** they see a message "No duplicates found in your library."

4. **Given** duplicate groups exist, **When** the user clicks on a group, **Then**
   they see larger previews of all photos in that group for comparison.

---

### User Story 2 - Approve Duplicate Removal (Priority: P2)

A user reviews a group of duplicate photos and decides which ones to keep and
which to remove. They select the photos to delete (or use "keep best" auto-select),
then approve the removal. The selected photos are moved to a review folder before
permanent deletion.

**Why this priority**: The core value proposition - helping users clean up their
library by removing duplicates they've reviewed and approved.

**Independent Test**: Can be fully tested by selecting duplicates for removal,
approving, and verifying they move to the review folder (not immediately deleted).

**Acceptance Scenarios**:

1. **Given** a duplicate group is displayed, **When** the user manually selects
   photos to remove, **Then** those photos are marked for removal while others
   remain unmarked.

2. **Given** a duplicate group is displayed, **When** the user clicks "Keep Best",
   **Then** the system auto-selects the highest quality photo to keep and marks
   others for removal.

3. **Given** photos are marked for removal, **When** the user clicks "Approve
   Removal", **Then** marked photos are moved to the "Trash Review" folder.

4. **Given** photos are in the Trash Review folder, **When** the user views that
   folder, **Then** they can still recover photos before permanent deletion.

---

### User Story 3 - Adjust Duplicate Sensitivity (Priority: P3)

A user finds the duplicate detection is too aggressive (grouping photos that aren't
really duplicates) or too lenient (missing obvious duplicates). They access settings
to adjust the similarity threshold, which controls how closely photos must match
to be considered duplicates.

**Why this priority**: Different users have different tolerance levels. A photographer
may want stricter matching; a casual user may want more aggressive cleanup.

**Independent Test**: Can be fully tested by changing the threshold, re-running
detection, and verifying different duplicate groups are identified.

**Acceptance Scenarios**:

1. **Given** the settings page, **When** the user views duplicate settings, **Then**
   they see a similarity threshold slider (default 90%).

2. **Given** a threshold of 95%, **When** detection runs, **Then** only nearly
   identical photos are grouped (stricter matching).

3. **Given** a threshold of 80%, **When** detection runs, **Then** more visually
   similar (but not identical) photos are grouped (looser matching).

4. **Given** the user changes the threshold, **When** they re-run detection,
   **Then** the new threshold is applied and duplicate groups are updated.

---

### User Story 4 - Batch Approve All Duplicates (Priority: P4)

A power user with many duplicate groups wants to quickly clean up without reviewing
each group individually. They use a "batch mode" to auto-select lower quality
versions across all groups and approve removal in one action.

**Why this priority**: Convenience feature for users with large libraries who trust
the detection accuracy. Not required for core functionality.

**Independent Test**: Can be fully tested by using batch mode on multiple groups
and verifying all lower-quality versions are marked for removal.

**Acceptance Scenarios**:

1. **Given** multiple duplicate groups exist, **When** the user clicks "Batch
   Review", **Then** the system auto-selects the best photo in each group.

2. **Given** batch selections are made, **When** the user reviews the summary,
   **Then** they see total photos to keep vs. remove before confirming.

3. **Given** the user approves batch removal, **When** the action completes,
   **Then** all marked photos across all groups are moved to Trash Review.

---

### Edge Cases

- What happens when two photos are exactly identical (same file)?
  - Group them together with 100% similarity; the system keeps the older version
    by default (original) and marks the newer one for removal.

- What happens when photos are similar but have different resolutions?
  - Group them together; "Keep Best" selects the highest resolution version.

- What happens when a photo is duplicated across multiple groups?
  - A photo can only belong to one duplicate group. The system groups by highest
    similarity pairing first.

- What happens when the user wants to keep a "duplicate" (e.g., intentional copies)?
  - User can dismiss the group without action, and those photos won't be flagged
    in future scans unless they choose to re-include them.

- What happens with edited versions of the same photo?
  - If similarity is above threshold (e.g., 90%), they're grouped. Users can adjust
    threshold if they want to exclude edited versions.

- What happens if duplicate detection runs on a very large library (100,000+ photos)?
  - Detection processes in batches with progress indication. Users can pause/resume
    and results are saved incrementally.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST analyze scanned photos to identify visually similar
  images using perceptual comparison.

- **FR-002**: System MUST group duplicate photos together and display them as
  reviewable sets with thumbnail previews.

- **FR-003**: System MUST calculate and display a similarity percentage for
  each duplicate group.

- **FR-004**: System MUST allow users to manually select which photos to keep
  and which to remove within each duplicate group.

- **FR-005**: System MUST provide a "Keep Best" auto-selection that chooses
  the highest quality photo based on resolution and file size.

- **FR-006**: System MUST move approved-for-removal photos to a "Trash Review"
  holding folder rather than permanently deleting immediately.

- **FR-007**: System MUST allow users to adjust the duplicate similarity
  threshold via a settings slider (range: 70% - 100%, default: 90%).

- **FR-008**: System MUST persist duplicate detection results locally so users
  can review them across sessions without re-running detection.

- **FR-009**: System MUST allow users to dismiss duplicate groups they don't
  want to act on (mark as "not duplicates").

- **FR-010**: System MUST provide batch approval capability for users who want
  to auto-select across all duplicate groups.

- **FR-011**: System MUST handle detection on large libraries with progress
  indication and pause/resume capability.

- **FR-012**: System MUST log all duplicate detection and removal actions for
  audit purposes.

### Key Entities

- **Duplicate Group**: A set of photos identified as duplicates of each other.
  Attributes include group identifier, list of photo items, similarity score,
  status (pending/reviewed/dismissed), and creation timestamp.

- **Photo Comparison**: Represents the similarity analysis between two photos.
  Attributes include photo A reference, photo B reference, similarity percentage,
  and comparison method used.

- **Removal Queue**: Photos approved for removal awaiting transfer to Trash Review.
  Attributes include photo reference, source duplicate group, approval timestamp,
  and removal status.

- **Trash Review Folder**: Virtual folder holding photos pending permanent deletion.
  Attributes include photo reference, original location, time added, and scheduled
  deletion date (if auto-delete enabled).

- **Detection Settings**: User preferences for duplicate detection behavior.
  Attributes include similarity threshold, auto-select preference, and batch mode
  enablement.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Duplicate detection correctly identifies 95% or more of actual
  duplicate photos in a test library (precision).

- **SC-002**: False positive rate (non-duplicates incorrectly grouped) is less
  than 5% at the default 90% threshold.

- **SC-003**: Users can review and approve/dismiss a duplicate group in under
  30 seconds.

- **SC-004**: "Keep Best" auto-selection correctly chooses the highest quality
  photo 98% of the time.

- **SC-005**: Duplicate detection completes on a 20,000 photo library within
  10 minutes after initial scan.

- **SC-006**: Zero photos are permanently deleted without first going through
  the Trash Review holding period.

- **SC-007**: Users with 100+ duplicate groups can complete batch approval in
  under 2 minutes.

- **SC-008**: Dismissed duplicate groups do not reappear in subsequent scans
  unless the user explicitly requests re-evaluation.

## Assumptions

- Photo Scanning Engine (001-photo-scanning) is completed and photos are indexed
  before duplicate detection can run.
- Perceptual similarity comparison is used (content-based, not filename-based).
- "Best" photo selection prioritizes resolution, then file size, then recency.
- The default similarity threshold of 90% provides a good balance for most users.
- Trash Review folder is a logical concept within the app, not a physical Google
  Photos album (until deletion is implemented).
- Users want approval before any deletions occur (no auto-delete by default).
- Duplicate detection runs locally to maintain privacy per constitution principles.
