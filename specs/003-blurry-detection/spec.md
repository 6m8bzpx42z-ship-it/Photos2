# Feature Specification: Blurry Detection

**Feature Branch**: `003-blurry-detection`
**Created**: 2025-12-31
**Status**: Draft
**Input**: User description: "blurry detection"
**Dependency**: Requires Photo Scanning Engine (001-photo-scanning) to be completed first

## User Scenarios & Testing *(mandatory)*

### User Story 1 - View Detected Blurry Photos (Priority: P1)

After a photo library scan completes, a user wants to see which photos have been
identified as blurry or out of focus. They navigate to the "Blurry Photos" section
of the dashboard and see a list of photos flagged as low quality, displayed with
thumbnails and blur severity indicators.

**Why this priority**: Users need to see detected blurry photos before they can
decide what to do with them. This is the core detection and display capability.

**Independent Test**: Can be fully tested by running detection on a library with
known blurry photos and verifying the correct photos are flagged in the UI.

**Acceptance Scenarios**:

1. **Given** a completed scan with blurry photos detected, **When** the user views
   the Blurry Photos section, **Then** they see flagged photos with thumbnails
   and blur severity levels.

2. **Given** blurry photos are displayed, **When** the user views a photo, **Then**
   they see the blur score and a visual indicator (e.g., "Slightly Blurry",
   "Very Blurry", "Extremely Blurry").

3. **Given** no blurry photos were detected, **When** the user views the Blurry
   Photos section, **Then** they see a message "No blurry photos found. Great job!"

4. **Given** blurry photos exist, **When** the user clicks on a photo, **Then**
   they see a larger preview to verify it really is blurry.

---

### User Story 2 - Approve Blurry Photo Removal (Priority: P2)

A user reviews the detected blurry photos and selects which ones to remove from
their library. They can select individual photos or use "Select All" to mark
all blurry photos for removal. Selected photos are moved to a review folder
before permanent deletion.

**Why this priority**: The core cleanup value - helping users remove unwanted
blurry photos they've reviewed and approved.

**Independent Test**: Can be fully tested by selecting blurry photos for removal,
approving, and verifying they move to the review folder.

**Acceptance Scenarios**:

1. **Given** blurry photos are displayed, **When** the user selects photos to
   remove, **Then** those photos are marked for removal with visual indication.

2. **Given** multiple blurry photos exist, **When** the user clicks "Select All",
   **Then** all displayed blurry photos are marked for removal.

3. **Given** photos are marked for removal, **When** the user clicks "Approve
   Removal", **Then** marked photos are moved to the "Trash Review" folder.

4. **Given** a photo was incorrectly flagged, **When** the user clicks "Not
   Blurry", **Then** that photo is unmarked and won't appear in future scans.

---

### User Story 3 - Adjust Blur Sensitivity (Priority: P3)

A user finds the blur detection is too strict (flagging acceptable photos) or
too lenient (missing obviously blurry photos). They access settings to adjust
the blur sensitivity threshold, which controls how strict the detection is.

**Why this priority**: Different users have different quality standards. A
photographer needs strict detection; a casual user may only want very blurry
photos flagged. The PRD specifically mentions this as a user setting.

**Independent Test**: Can be fully tested by changing the sensitivity, re-running
detection, and verifying different photos are flagged.

**Acceptance Scenarios**:

1. **Given** the settings page, **When** the user views blur settings, **Then**
   they see a sensitivity slider (1-100, default strict as per PRD).

2. **Given** sensitivity at 90 (strict), **When** detection runs, **Then** even
   slightly blurry photos are flagged.

3. **Given** sensitivity at 50 (moderate), **When** detection runs, **Then** only
   moderately to very blurry photos are flagged.

4. **Given** sensitivity at 20 (lenient), **When** detection runs, **Then** only
   extremely blurry photos are flagged.

5. **Given** the user changes sensitivity, **When** they re-run detection, **Then**
   the new threshold is applied and flagged photos are updated.

---

### User Story 4 - Batch Review Blurry Photos (Priority: P4)

A power user with many blurry photos wants to quickly review them in a batch
interface rather than one by one. They access a grid view showing all blurry
photos together and can rapidly select/deselect photos for removal.

**Why this priority**: Efficiency feature for users with large numbers of blurry
photos. The PRD emphasizes batch-based review (not individual clicking).

**Independent Test**: Can be fully tested by using batch view on multiple photos
and verifying efficient selection without per-photo page loads.

**Acceptance Scenarios**:

1. **Given** many blurry photos exist, **When** the user accesses batch review,
   **Then** they see a grid of thumbnails (e.g., 4x4) for rapid scanning.

2. **Given** the batch grid is displayed, **When** the user clicks thumbnails,
   **Then** they toggle selection state without loading individual pages.

3. **Given** the user has made selections, **When** they click "Review Selection",
   **Then** they see a summary of photos to remove before final approval.

---

### Edge Cases

- What happens when a photo is intentionally blurry (artistic effect)?
  - User can mark it as "Not Blurry" to exclude from future detection. These
    exclusions are persisted locally.

- What happens with motion blur vs. focus blur?
  - Both are detected as "blurry" since both result in image quality loss. The
    system doesn't distinguish between blur types.

- What happens when a photo has partial blur (e.g., background blur/bokeh)?
  - Detection focuses on the main subject area. Photos with intentional background
    blur (bokeh) where the subject is sharp should not be flagged.

- What happens with very low resolution photos?
  - Low resolution alone doesn't trigger blur detection. Only actual blur/focus
    issues are detected regardless of resolution.

- What happens if blur detection runs on a very large library?
  - Detection processes in batches with progress indication. Users can pause/resume
    and results are saved incrementally.

- What happens with screenshots or text images?
  - Screenshots with crisp text are not flagged. Only photos with camera-captured
    content are analyzed for blur.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST analyze scanned photos to identify blurry or out-of-focus
  images using image sharpness analysis.

- **FR-002**: System MUST display detected blurry photos with thumbnails and blur
  severity indicators (Slightly/Very/Extremely Blurry).

- **FR-003**: System MUST calculate and display a blur score for each detected
  photo (0-100 scale, higher = more blurry).

- **FR-004**: System MUST allow users to select individual photos or use "Select
  All" to mark blurry photos for removal.

- **FR-005**: System MUST move approved-for-removal photos to a "Trash Review"
  holding folder rather than permanently deleting immediately.

- **FR-006**: System MUST allow users to adjust blur sensitivity via a settings
  slider (range: 1-100, default: 70 for strict detection per PRD).

- **FR-007**: System MUST allow users to mark photos as "Not Blurry" to exclude
  them from future detection.

- **FR-008**: System MUST persist blur detection results and exclusions locally
  so users can review them across sessions.

- **FR-009**: System MUST provide a batch grid view for efficient review of
  multiple blurry photos (minimum 16 photos visible at once).

- **FR-010**: System MUST handle detection on large libraries with progress
  indication and pause/resume capability.

- **FR-011**: System MUST log all blur detection and removal actions for audit
  purposes.

- **FR-012**: System MUST distinguish between photos and screenshots, only
  analyzing camera-captured photos for blur.

### Key Entities

- **Blurry Photo Flag**: Represents a photo identified as blurry. Attributes
  include photo reference, blur score (0-100), severity level, detection
  timestamp, and status (pending/removed/excluded).

- **Blur Settings**: User preferences for blur detection behavior. Attributes
  include sensitivity threshold (1-100) and whether detection is enabled.

- **Exclusion List**: Photos marked as "Not Blurry" by the user. Attributes
  include photo reference, exclusion timestamp, and reason (if provided).

- **Removal Queue**: Photos approved for removal awaiting transfer to Trash Review.
  Attributes include photo reference, blur score, approval timestamp, and
  removal status.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Blur detection correctly identifies 95% or more of actually blurry
  photos in a test library (recall).

- **SC-002**: False positive rate (sharp photos incorrectly flagged) is less than
  10% at the default strict sensitivity setting.

- **SC-003**: Users can review and approve/exclude a blurry photo in under 10
  seconds per photo.

- **SC-004**: Batch review allows users to process 50+ photos in under 5 minutes.

- **SC-005**: Blur detection completes on a 20,000 photo library within 15 minutes
  after initial scan.

- **SC-006**: Zero photos are permanently deleted without first going through
  the Trash Review holding period.

- **SC-007**: Excluded photos ("Not Blurry" marked) do not reappear in subsequent
  scans unless user explicitly requests re-evaluation.

- **SC-008**: Blur severity levels match user perception at least 90% of the time
  (Slightly/Very/Extremely correlates with actual blur amount).

## Assumptions

- Photo Scanning Engine (001-photo-scanning) is completed and photos are indexed
  before blur detection can run.
- Blur detection uses image sharpness analysis (not subjective quality assessment).
- The default sensitivity of 70 provides strict detection as mentioned in the PRD.
- Blur detection runs locally to maintain privacy per constitution principles.
- Trash Review folder is shared with Duplicate Detection feature (same holding area).
- Screenshots and non-camera photos are excluded from blur analysis.
- Bokeh (intentional background blur) with sharp subjects should not trigger detection.
