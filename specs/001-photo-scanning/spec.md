# Feature Specification: Photo Scanning Engine

**Feature Branch**: `001-photo-scanning`
**Created**: 2025-12-31
**Status**: Draft
**Input**: User description: "photo scanning engine"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - First-Time Authentication (Priority: P1)

A user opens the Photo Cleaner application for the first time and needs to connect
their Google Photos account. They click "Connect Google Photos", are redirected to
Google's login page, grant read access permissions, and return to the app with their
account successfully linked.

**Why this priority**: Without authentication, no other features can function. This is
the foundational capability that unlocks the entire product.

**Independent Test**: Can be fully tested by completing the OAuth flow and verifying
the app displays "Connected" status with the user's Google account email.

**Acceptance Scenarios**:

1. **Given** a user has not connected their Google account, **When** they click
   "Connect Google Photos", **Then** they are redirected to Google's OAuth consent
   screen.

2. **Given** a user completes Google OAuth consent, **When** they are redirected back
   to the app, **Then** their account shows as connected with their email displayed.

3. **Given** a user denies OAuth consent, **When** they are redirected back to the
   app, **Then** they see a friendly message explaining connection is required and
   can retry.

4. **Given** a user's OAuth token expires, **When** they next use the app, **Then**
   the system automatically refreshes the token without requiring user action.

---

### User Story 2 - Scan Photo Library (Priority: P2)

A user with a connected Google Photos account initiates a scan of their entire photo
library. They click "Start Scan", see a progress indicator showing how many photos
have been processed, and upon completion see a summary of their library (total photos,
total videos, date range covered).

**Why this priority**: Scanning is the core engine that powers all detection features.
Without the ability to crawl and index photos, no cleanup can occur.

**Independent Test**: Can be fully tested by running a scan and verifying the summary
shows accurate counts matching what's visible in Google Photos.

**Acceptance Scenarios**:

1. **Given** a connected Google account with photos, **When** the user clicks "Start
   Scan", **Then** the system begins retrieving photo metadata and shows progress.

2. **Given** a scan is in progress, **When** the user views the progress indicator,
   **Then** they see the number of photos processed and estimated time remaining.

3. **Given** a scan completes successfully, **When** the user views the summary,
   **Then** they see total photo count, total video count, and the date range of
   their library.

4. **Given** a large library (20,000+ items), **When** the scan runs, **Then** it
   completes without timing out and processes all items.

---

### User Story 3 - Resume Interrupted Scan (Priority: P3)

A user starts a scan but their laptop goes to sleep, loses internet, or they close
the app mid-scan. When they return, they can resume from where they left off rather
than starting over from scratch.

**Why this priority**: Large libraries can take significant time to scan. Losing
progress due to interruptions would frustrate users and waste their time.

**Independent Test**: Can be fully tested by starting a scan, forcibly stopping the
app, restarting, and verifying the scan continues from the last checkpoint.

**Acceptance Scenarios**:

1. **Given** a scan was interrupted, **When** the user reopens the app, **Then** they
   see an option to "Resume Scan" with progress preserved.

2. **Given** the user chooses to resume, **When** scanning continues, **Then** only
   unprocessed photos are fetched (no duplicated work).

3. **Given** the user prefers to start fresh, **When** they click "Start New Scan",
   **Then** all previous scan data is cleared and scanning begins from the start.

---

### User Story 4 - View Scan History (Priority: P4)

A user wants to see when their library was last scanned and compare current scan
results to previous scans. They access a history view showing past scan dates,
photo counts, and any changes detected.

**Why this priority**: Helps users understand their library growth over time and
provides confidence that the app is working correctly across sessions.

**Independent Test**: Can be fully tested by running multiple scans and verifying
the history shows each scan with accurate timestamps and counts.

**Acceptance Scenarios**:

1. **Given** the user has completed multiple scans, **When** they view scan history,
   **Then** they see a list of past scans with dates and photo counts.

2. **Given** a new scan shows different counts than previous, **When** viewing
   history, **Then** the user can see the delta (photos added/removed since last
   scan).

---

### Edge Cases

- What happens when the user's Google Photos library is empty?
  - Display a friendly message: "No photos found. Add some photos to Google Photos
    and scan again."

- What happens when network connectivity is lost during a scan?
  - Pause the scan, save progress, and show "Connection lost. Scan paused." with
    a "Retry" button.

- What happens when the user revokes app permissions via Google settings?
  - On next app use, detect invalid token and prompt user to reconnect with clear
    messaging about why.

- What happens when rate limits are hit on Google Photos API?
  - Implement automatic backoff, show "Scan slowed due to rate limits" message,
    and continue processing at reduced speed.

- What happens when a photo has corrupted or missing metadata?
  - Log the issue, skip the problematic item, and include it in a "skipped items"
    count in the scan summary.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST authenticate users via Google OAuth2 with read-only
  access to Google Photos Library.

- **FR-002**: System MUST securely store OAuth refresh tokens locally so users
  don't need to re-authenticate on every app launch.

- **FR-003**: System MUST automatically refresh expired access tokens without
  user intervention.

- **FR-004**: System MUST crawl the user's entire Google Photos library and
  retrieve metadata for all photos and videos.

- **FR-005**: System MUST display scan progress showing items processed and
  estimated time remaining.

- **FR-006**: System MUST persist scan progress locally to enable resumption
  after interruption.

- **FR-007**: System MUST generate a scan summary showing total photos, total
  videos, and library date range.

- **FR-008**: System MUST handle API rate limits gracefully with automatic
  retry and backoff.

- **FR-009**: System MUST log all scan operations for debugging and audit purposes.

- **FR-010**: System MUST allow users to disconnect their Google account and
  clear all stored data.

### Key Entities

- **User Account**: Represents the connected Google account. Attributes include
  email address, connection status, and last authenticated timestamp.

- **OAuth Token**: Represents authentication credentials. Attributes include
  access token, refresh token, and expiration timestamp.

- **Photo Item**: Represents a single photo or video from Google Photos. Attributes
  include unique identifier, filename, creation date, media type (photo/video),
  and file size.

- **Scan Session**: Represents a single scanning operation. Attributes include
  start time, end time, status (in-progress/completed/interrupted), items
  processed count, and checkpoint for resumption.

- **Scan History Entry**: Represents a completed scan record. Attributes include
  scan date, total photos, total videos, and duration.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can connect their Google Photos account in under 60 seconds
  from app launch to successful connection.

- **SC-002**: Scan can process a library of 20,000 photos within 15 minutes on
  a standard home internet connection.

- **SC-003**: Interrupted scans resume with less than 1% duplicated work (items
  re-processed unnecessarily).

- **SC-004**: 100% of photos and videos in the user's library are discovered
  and indexed by the scan.

- **SC-005**: Users never need to manually re-authenticate after initial setup
  (automatic token refresh works reliably).

- **SC-006**: Scan progress indicator accuracy is within 10% of actual completion
  percentage.

- **SC-007**: System handles network interruptions gracefully with zero data loss
  and clear user messaging.

## Assumptions

- Users have a Google account with Google Photos enabled.
- Users are running the application on a device with internet connectivity.
- Google Photos API quotas are sufficient for typical personal library sizes
  (up to 100,000 items).
- The application runs on a local machine (laptop/desktop) per Phase 1 requirements.
- OAuth2 is the appropriate authentication mechanism for Google Photos access.
- Read-only access is sufficient for scanning; write access will be added in
  future features (deletion, album creation).
