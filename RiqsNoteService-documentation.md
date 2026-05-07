# RiqsItNote Documentation

## Scope

### Overview

RiQS-It is a lightweight, informal communication tool that functions like a digital whiteboard where users can create and manage "Post-It" style notes. It is accessible as a persistent overlay on any screen — not a separate module — and is triggered via a floating button fixed to the bottom-right corner of the interface (draggable by the user). When activated, RiQS-It slides in from the right as an overlay and can be resized or expanded to full screen.

---

### Whiteboards

There are two whiteboards available to each user:

| Whiteboard | Visibility | Description |
|---|---|---|
| **Personal** | Private — owner only | A personal space where only the user can see and manage their own notes |
| **Unit** | Shared — all unit members | A shared space where all users of a given unit can create and view notes |

---

### Accessing RiQS-It

- RiQS-It is **not** a module in the left sidebar
- A **floating button** is permanently visible in the bottom-right corner of the screen (default position on login), draggable anywhere on screen
- Clicking the button opens RiQS-It as a **right-side overlay** sliding in from the right
- The overlay can be:
  - Expanded to **full screen** via an expand icon
  - **Resized** by dragging from the left edge
  - **Closed** via an always-visible `X` button, which collapses the panel and restores the floating button

---

### Notes

Creating a note is quick and simple. When creating a note, a user can:

- Select a **background color** for the note
- Write the **text content** of the note
- Add **attachments** — from RiQS-Pedia, uploaded from desktop, or taken as a photo on mobile
- **Save** the note to place it on the whiteboard
- **Cancel** to discard

Once placed on the whiteboard, a note:

- Can be **moved freely** around the board
- Retains its position **persistently** across sessions (including after logout)

---

### Personal Whiteboard

- Each user has a private whiteboard visible **only to themselves**
- Users can create, edit, and delete their own notes
- Editable fields: note content and background color
- Each note automatically displays the **date of creation** in the top-right corner

---

### Unit Whiteboard

- All users of a unit share access to a single unit whiteboard
- Each note displays the **creator's name and date of creation** in the top-right corner
- Permissions:

| Action | Who can perform |
|---|---|
| Create a note | All unit users |
| Move a note | All unit users |
| Edit own note | Note creator only |
| Delete own note | Note creator only |
| Delete any note | Power-Users and Management users only |
---
### Note Ordering and Pinning Rules:

- **Pinning** a note fixes it in place on the unit whiteboard. Only Management and Power users can pin or unpin any note
- A pinned note **cannot be moved by anyone**, including the original author — the pinned status overrides all other move permissions
- Professionals can only reorder **their own notes**; they cannot move notes created by others

---
### Notifications

- When a new note is created on the **unit whiteboard**, all unit users receive a notification via the floating RiQS-It button (badge indicator)
- Clicking the notification badge opens RiQS-It directly to the **Unit** tab
- Once the unit view is opened, the notification is considered **read and is dismissed**

---
## Permissions

| Operation | Endpoint | Permission |
|---|---|---|
| `createNote` | `CreateRIQSNote` | All authenticated users |
| `updateNote` | `UpdateRIQSNote` | Note owner only |
| `getRiqsNotes` | `GetRIQSNote` | All authenticated users |
| `deleteRiqsNote` | `DeleteRIQSNote` | Own note — any user. Any note — Power and Management users |
| `getRiqsNoteComments` | `GetRIQSNoteComments` | All authenticated users |
| `saveRiqsNoteComment` | `CreateRIQSNoteComment` | All authenticated users |
| `updateRiqsNoteComment` | `UpdateRIQSNoteComment` | Comment owner only |
| `deleteRiqsNoteComment` | `DeleteRIQSNoteComment` | Own comment — any user. Any comment — Professional and Management users |
| `getUnreadNotifications` | `GetUnreadNotificationsBySubscriptionFilter` | Authenticated user (own notifications) |
| `markNotificationAsRead` | `UpdateNotificationStatusToRead` | Authenticated user (own notifications) |
| `updateRIQSNoteOrder` | `UpdateRIQSNoteOrder` | **Personal view:** all authenticated users. **Unit view:** Management and Power users can reorder any note. Professionals can only reorder their own notes. If a note is pinned, it cannot be moved by anyone regardless of authorship |
| `updateNotePinnedStatus` | `PinRIQSNote` | Management and Power users only (unit view). Pinned status overrides all move permissions — the original author can no longer move their own note while it is pinned |
| `getPreSignedUrlForUpload` | `GetPreSignedUrlForUpload` | All authenticated users |

---

## RiqsNoteService Documentation
**File:** `riqs-note.service.ts`  
**Injectable:** Root-level singleton  
**Purpose:** Manages all HTTP operations for RIQS Notes, Comments, Notifications, and ordering/pinning.
---

## Base URLs

| Name | URL |
|---|---|
| Query (GET operations) | `PraxisBusinessService/PraxisMonitorQuery/` |
| Command (POST/mutate) | `PraxisBusinessService/PraxisMonitorCommand/` |
| Notifications | `NotificationService/api/Notifier/GetUnreadNotificationsBySubscriptionFilter` |
---

## Methods

### Create Note

### `createNote(payload)`
 
Creates a new RIQS Note.

> **Permission:** All authenticated users can create a note.

**Endpoint:** `POST PraxisMonitorCommand/CreateRIQSNote`
 
**Payload:**
```json
{
  "Content": "<article class=\"riqsit-note\"><h1 class=\"editor-title\">Note Title</h1><section class=\"editor-body\"><p>Note body text here</p></section></article>",
  "BackgroundColor": "none",
  "Possition": {
    "XCoordinate": 0,
    "YCoordinate": 0
  },
  "Attachments": [],
  "NoteType": "personal",
  "AdditionalMetadata": {},
  "MentionedUsers": [],
  "ClientId": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070"
}
```
 
**Payload Fields:**
 
| Field | Type | Required | Description |
|---|---|---|---|
| `Content` | `string` (HTML) | Yes | Full HTML content of the note. Must follow the structure: `<article class="riqsit-note">` → `<h1 class="editor-title">` for title + `<section class="editor-body">` for body |
| `BackgroundColor` | `string` | Yes | Background color of the note card. Use `"none"` for default/transparent |
| `Possition.XCoordinate` | `number` | Obsolete | ~~Vertical position of the note (canvas Y-axis)~~ — **Not used, pass `0`** |
| `Possition.YCoordinate` | `number` | Obsolete | ~~Vertical position of the note (canvas Y-axis)~~ — **Not used, pass `0`** |
| `Attachments` | `Attachment[]` | Yes | List of file attachments. Pass `[]` if none |
| `NoteType` | `string` | Yes | Type of note. Known values: `"personal"` |
| `AdditionalMetadata` | `object` | Yes | Extra key-value metadata. Pass `{}` if unused |
| `MentionedUsers` | `MentionedUser[]` | Yes | List of users mentioned via `@` in the note body. Pass `[]` if none |
| `ClientId` | `string` (GUID) | Yes | The client this note belongs to |

**Attachment Payload:**
```json
"Attachments": [
  {
    "CreatedOn": "2026-05-06T12:09:55.378Z",
    "DocumentId": "3d459679-51ca-4eeb-a488-468aeb505e96",
    "DocumentName": "invoice.pdf",
    "FileType": "pdf",
    "IsDeleted": false,
    "IsUploadedFromWeb": true
  }
]
```

**Attachment Payload Fields:**
| Field | Type | Required | Description |
|---|---|---|---|
| `CreatedOn` | `string` (ISO 8601) |  Yes | Upload timestamp e.g. `"2026-05-06T12:09:55.378Z"` |
| `DocumentId` | `string` (GUID) |  Yes | Unique ID of the uploaded document |
| `DocumentName` | `string` |  Yes | Original filename with extension e.g. `"invoice.pdf"` |
| `FileType` | `string` |  Yes | File extension without dot. Known values: `"pdf"`, `"png"` |
| `IsDeleted` | `boolean` |  Yes | Soft delete flag. Always `false` on create |
| `IsUploadedFromWeb` | `boolean` |  Yes | `true` if uploaded via the web client |

**MentionedUsers Payload:**
```json
"MentionedUsers": [
  {
    "PraxisUserId": "6bbfed43-49bb-41c5-aa21-ebc5e36740e4",
    "UserId": "988a4fa6-2494-4491-afda-672c1be0db6c",
    "DisplayName": "AdminB 2",
    "Email": "a.adminb@yopmail.com",
    "PraxisImage": {
      "FileId": "",
      "FileName": null,
      "FileSize": null,
      "CreatedOn": null,
      "Thumbnails": [],
      "IsUploadedFromWeb": true
    }
  }
]
```

**MentionedUsers Object:**
| Field | Type | Required | Description |
|---|---|---|---|
| `PraxisUserId` | `string` (GUID) | Yes | The user's ID in the Praxis system |
| `UserId` | `string` (GUID) | Yes | The user's platform-level ID |
| `DisplayName` | `string` | Yes | Name shown in the `@mention` e.g. `"AdminB QA"` |
| `Email` | `string` | Yes | User's email address |
| `PraxisImage.FileId` | `string` | Yes | Profile image file ID. Pass `""` if no image |
| `PraxisImage.FileName` | `null` | Yes | Pass file name or `null` |
| `PraxisImage.FileSize` | `null` | Yes | Pass file size or `null` |
| `PraxisImage.CreatedOn` | `null` | Yes | Pass CreatedOn or  `null` |
| `PraxisImage.Thumbnails` | `any[]` | Yes | Pass `[]` |
| `PraxisImage.IsUploadedFromWeb` | `boolean` | Yes | Always `true` |


**Content HTML Structure:**
```html
<article class="riqsit-note">
  <h1 class="editor-title">Your Note Title</h1>
  <section class="editor-body">
    <p>Your note body content goes here.</p>
  </section>
</article>
```
 
**Success Response:** See [Success Response](#success-response).
 
**Response Fields:**
 
| Field | Type | Description |
|---|---|---|
| `Errors.IsValid` | `boolean` | `true` means the operation succeeded without validation errors |
| `Errors.Errors` | `any[]` | List of validation errors. Empty on success |
| `Errors.RuleSetsExecuted` | `string \| null` | Which validation rule sets ran. Usually `null` |
| `ErrorMessages` | `string[]` | High-level error messages. Empty on success |
| `StatusCode` | `number` | App-level status code. `0` = success |
| `RequestUri` | `string \| null` | Echo of the request URI, if returned |
| `ExternalError` | `any \| null` | Error from an external service, if any |
| `HttpStatusCode` | `number` | HTTP status code mirrored in body. `0` or `200` = success |
 
**Returns:** `Observable<CreateNoteResponse>` — the response body mapped from the HTTP response.

---




### Update Note
### `updateNote(payload)`

Updates an existing RIQS Note.

> **Permission:** Only the note owner can update a note.

**Endpoint:** `POST PraxisMonitorCommand/UpdateRIQSNote`

**Payload:**
```json
{
  "ItemId": "8b9d3e92-67c2-4ae9-8072-1946f68d322d",
  "Content": "<article class=\"riqsit-note\"><h1 class=\"editor-title\">Note Title</h1><section class=\"editor-body\"><p>Note body text here</p></section></article>",
  "BackgroundColor": "none",
  "Possition": {
    "XCoordinate": 0,
    "YCoordinate": 0
  },
  "Attachments": [],
  "NoteType": "personal",
  "AdditionalMetadata": {},
  "MentionedUsers": [],
  "ClientId": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070"
}
```

**Payload Fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `ItemId` | `string` (GUID) | Yes | ID of the note to update |
| `Content` | `string` (HTML) | Yes | Updated HTML content of the note. Same structure as `createNote` |
| `BackgroundColor` | `string` | Yes | Background color of the note card. Use `"none"` for default/transparent |
| `Possition.XCoordinate` | `number` | Obsolete | ~~Horizontal position on canvas~~ — Not used, always pass `0` |
| `Possition.YCoordinate` | `number` | Obsolete | ~~Vertical position on canvas~~ — Not used, always pass `0` |
| `Attachments` | `Attachment[]` | Yes | Updated list of attachments. Pass `[]` if none. See `createNote` for object shape |
| `NoteType` | `string` | Yes | Type of note. Known values: `"personal"` |
| `AdditionalMetadata` | `object` | Yes | Extra key-value metadata. Pass `{}` if unused |
| `MentionedUsers` | `MentionedUser[]` | Yes | Updated list of mentioned users. Pass `[]` if none. See `createNote` for object shape |
| `ClientId` | `string` (GUID) | Yes | The client this note belongs to |

**Success Response:** See [Success Response](#success-response).

**Returns:** `Observable<any>`

---

### Get RIQS Notes
### `getRiqsNotes(filters?)`

Fetches a list of RIQS Notes, optionally filtered.
> **Permission:** All authenticated users can fetch riqs notes.

**Endpoint:** `POST PraxisMonitorQuery/GetRIQSNote`

**Payload:**
```json
{
  "FilterString": "{<generated from filters>}",
  "SortedBy": "{\"OrderNumber\": -1}",
  "PageSize": 1000000,
  "PageNumber": 0
}
```

**Parameters:**
| Param | Type | Default | Description |
|---|---|---|---|
| `FilterString` | `string` | `"{}"` | JSON-like string used to filter results (generated via `prepareFilters`) |
| `SortedBy` | `string` | `"{\"OrderNumber\": -1}"` | JSON string defining sort order (e.g. descending by `OrderNumber`) |
| `PageSize` | `number` | `1000000` | Number of records to return |
| `PageNumber` | `number` | `0` | Page index (0-based) |

**Success Response:**
```json
{
  "StatusCode": 0,
  "ErrorMessage": null,
  "TotalRecordCount": 1,
  "Results": [
    {
      "ItemId": "16e33d85-73de-46e1-b31d-36e19f0e2ad1",
      "Content": "<article class=\"riqsit-note\"><h1 class=\"editor-title\">mention user</h1><section class=\"editor-body\"><p class=\"mentioned-user\">@RIQS Management - QA </p></section></article>",
      "BackgroundColor": "none",
      "Possition": {
        "XCoordinate": 0,
        "YCoordinate": 0
      },
      "Attachments": [],
      "NoteType": "unit",
      "AdditionalMetadata": {},
      "MentionedUsers": [
        {
          "PraxisUserId": "e10fe0b5-1ddb-4e41-a380-df442f7fc538",
          "UserId": "adac55ad-6021-4100-84e0-6cebe11e25a9",
          "DisplayName": "RIQS Management - QA",
          "Image": null,
          "Email": "riqs.manage@yopmail.com"
        }
      ],
      "CreateDate": "2026-05-03T09:35:49.455Z",
      "CreatedBy": "568725bb-89ce-412b-8db8-3184badca278",
      "LastUpdateDate": "2026-05-03T09:35:49.455Z",
      "LastUpdatedBy": null,
      "CommentCount": 0,
      "OrderNumber": 70,
      "IsPinned": null,
      "PinnedBy": null,
      "PinnedByDisplayName": null
    }
  ]
}
```
**Response Fields:**

| Field | Type | Description |
|---|---|---|
| `StatusCode` | `number` | Appplication-level status code. `0` = success |
| `ErrorMessage` | `string \| null` | Error message if request fails, otherwise `null` |
| `TotalRecordCount` | `number` | Total number of notes available (useful for pagination) |
| `Results` | `RIQSNote[]` | Array of returned notes |

**RIQSNote Object:**

| Field | Type | Description |
|---|---|---|
| `ItemId` | `string` (GUID) | Unique ID of the note |
| `Content` | `string` (HTML) | Full HTML content of the note |
| `BackgroundColor` | `string` | Background color of the note |
| `Possition` | `object` | Position info (obsolete, always `{0,0}`) |
| `Attachments` | `Attachment[]` | List of attached files |
| `NoteType` | `string` | Type of note (`personal`, `unit`, etc.) |
| `AdditionalMetadata` | `object` | Additional key-value metadata |
| `MentionedUsers` | `MentionedUser[]` | List of mentioned users |
| `CreateDate` | `string` (ISO datetime) | Note creation timestamp |
| `CreatedBy` | `string` (GUID) | User who created the note |
| `LastUpdateDate` | `string` (ISO datetime) | Last updated timestamp |
| `LastUpdatedBy` | `string \| null` | User who last updated the note |
| `CommentCount` | `number` | Number of comments on the note |
| `OrderNumber` | `number` | Used for sorting/order positioning |
| `IsPinned` | `boolean \| null` | Whether the note is pinned |
| `PinnedBy` | `string \| null` | User ID who pinned the note |
| `PinnedByDisplayName` | `string \| null` | Display name of the user who pinned |

**Returns:** `Observable<any[]>` — array of notes, sorted by `OrderNumber` descending.

---

### Delete RIQS Note
### `deleteRiqsNote(itemId)`
Deletes a RIQS Note by ID.

**Endpoint:** `POST PraxisMonitorCommand/DeleteRIQSNote`

> **Permission:** 
- Every user can delete their own note.
- Power and management users can delete any note.

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `itemId` | `string` | The ID of the note to delete |

**Payload:**
```json
{
  "ItemId": "string"
}
```

**Returns:** `Observable<any>` — response body confirming deletion.

---


### Get RIQS Note Comments
### `getRiqsNoteComments(filters?)`

Fetches a list of RIQS Note comments, filtered.
> **Permission:** All authenticated users can fetch riqs note comments.

**Endpoint:** `POST PraxisMonitorQuery/GetRIQSNoteComments`

**Payload:**
```json
{
    "FilterString": "{RIQSNoteId:'dbae1999-0316-4b28-98e4-3c8cc574abb6'}",
    "SortedBy": "{\"CreateDate\": -1}",
    "PageSize": 1000000,
    "PageNumber": 0
}
```

**Parameters:**
| Param | Type | Default | Description |
|---|---|---|---|
| `FilterString` | `string` | `"{}"` | JSON-like string used to filter results (generated via `prepareFilters`) |
| `SortedBy` | `string` | `"{\"OrderNumber\": -1}"` | JSON string defining sort order (e.g. descending by `OrderNumber`) |
| `PageSize` | `number` | `1000000` | Number of records to return |
| `PageNumber` | `number` | `0` | Page index (0-based) |

**Success Response:**
```json
{
  "StatusCode": 0,
  "ErrorMessage": null,
  "TotalRecordCount": 1,
  "Results": [
    {
      "ItemId": "16e33d85-73de-46e1-b31d-36e19f0e2ad1",
      "Content": "<article class=\"riqsit-note\"><h1 class=\"editor-title\">mention user</h1><section class=\"editor-body\"><p class=\"mentioned-user\">@RIQS Management - QA </p></section></article>",
      "BackgroundColor": "none",
      "Possition": {
        "XCoordinate": 0,
        "YCoordinate": 0
      },
      "Attachments": [],
      "NoteType": "unit",
      "AdditionalMetadata": {},
      "MentionedUsers": [
        {
          "PraxisUserId": "e10fe0b5-1ddb-4e41-a380-df442f7fc538",
          "UserId": "adac55ad-6021-4100-84e0-6cebe11e25a9",
          "DisplayName": "RIQS Management - QA",
          "Image": null,
          "Email": "riqs.manage@yopmail.com"
        }
      ],
      "CreateDate": "2026-05-03T09:35:49.455Z",
      "CreatedBy": "568725bb-89ce-412b-8db8-3184badca278",
      "LastUpdateDate": "2026-05-03T09:35:49.455Z",
      "LastUpdatedBy": null,
      "CommentCount": 0,
      "OrderNumber": 70,
      "IsPinned": null,
      "PinnedBy": null,
      "PinnedByDisplayName": null
    }
  ]
}
```
**Response Fields:**

| Field | Type | Description |
|---|---|---|
| `StatusCode` | `number` | Application-level status code. `0` = success |
| `ErrorMessage` | `string \| null` | Error message if request fails, otherwise `null` |
| `TotalRecordCount` | `number` | Total number of notes available (useful for pagination) |
| `Results` | `RIQSNote[]` | Array of returned notes |

---

**RIQSNote Object:**

| Field | Type | Description |
|---|---|---|
| `ItemId` | `string` (GUID) | Unique ID of this record (note or related entity) |
| `RIQSNoteId` | `string` (GUID) | ID of the parent RIQS Note |
| `Content` | `string` (HTML) | HTML content of the note |
| `Attachments` | `Attachment[]` | List of attached files |
| `MentionedUsers` | `MentionedUser[]` | List of mentioned users |
| `ClientId` | `string` (GUID) | Client this note belongs to |
| `IsEdited` | `boolean` | Indicates whether the note has been edited |
| `AdditionalMetadata` | `object` | Additional key-value metadata |
| `CreateDate` | `string` (ISO datetime) | Creation timestamp |
| `CreatedBy` | `string` (GUID) | User who created the note |
| `LastUpdateDate` | `string` (ISO datetime) | Last updated timestamp |
| `LastUpdatedBy` | `string \| null` | User who last updated the note |

---

**Attachment Object:**

| Field | Type | Description |
|---|---|---|
| `DocumentId` | `string` (GUID) | Unique ID of the document |
| `DocumentName` | `string` | File name with extension |
| `CreateDate` | `string \| null` | Legacy field (often null) |
| `CreatedOn` | `string` (ISO datetime) | Upload timestamp |
| `FileType` | `string` | File type (e.g. `png`, `pdf`) |
| `IsDeleted` | `boolean` | Soft delete flag |
| `IsUploadedFromWeb` | `boolean` | Upload source indicator |

---

**MentionedUser Object:**

| Field | Type | Description |
|---|---|---|
| `PraxisUserId` | `string` (GUID) | Praxis system user ID |
| `UserId` | `string` (GUID) | Platform user ID |
| `DisplayName` | `string` | User display name |
| `Image` | `any \| null` | User profile image |
| `Email` | `string` | User email address |

**Returns:** `Observable<GetRIQSNoteCommentsResponse>` — emits the response body containing the list of comments for the given note.

---

### Create Note Comment
### `saveRiqsNoteComment(payload)`

Creates a new comment for a specific RIQS Note.
> **Permission:** All authenticated users can create note comments.

**Endpoint:** `POST PraxisMonitorQuery/CreateRIQSNoteComment`

**Payload:**
```json id="p3k8wx"
{
  "Content": "<article class=\"riqsit-note\"><section class=\"editor-body\"><p>Comment text here</p></section></article>",
  "BackgroundColor": "none",
  "Possition": {
    "XCoordinate": 0,
    "YCoordinate": 0
  },
  "Attachments": [],
  "AdditionalMetadata": {},
  "MentionedUsers": [],
  "ClientId": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070",
  "RIQSNoteId": "404b5109-4a1c-4770-af04-0db5e7e16825"
}
```

**Parameters:**
| Param | Type | Default | Description |
|---|---|---|---|
| `Content` | `string` | - | HTML content of the comment |
| `BackgroundColor` | `string` | `"none"` | Background color of the comment |
| `Possition` | `object` | `{ XCoordinate: 0, YCoordinate: 0 }` | Position info (obsolete, always 0,0) |
| `Attachments` | `Attachment[]` | `[]` | List of attached files |
| `AdditionalMetadata` | `object` | `{}` | Extra metadata key-value pairs |
| `MentionedUsers` | `MentionedUser[]` | `[]` | List of mentioned users in comment |
| `ClientId` | `string` | - | Client identifier |
| `RIQSNoteId` | `string` | - | Parent RIQS Note ID |

**Success Response:** See [Success Response](#success-response).
 
**Returns:** `Observable<any>` — emits the command response confirming whether the comment was created successfully.

---

### Update RIQS Note Comment
### `updateRiqsNoteComment(payload)`

Updates an existing comment on a RIQS Note.
> **Permission:** Only owner of note comment can update a note comment.

**Endpoint:** `POST PraxisMonitorQuery/UpdateRIQSNoteComment`

**Payload:**
```json id="p3k8wx"
{
    "Content": "<article class=\"riqsit-note\"><section class=\"editor-body\"><p>ssdfsdfsdx xxx</p></section></article>",
    "BackgroundColor": "none",
    "Possition": {
        "XCoordinate": 0,
        "YCoordinate": 0
    },
    "Attachments": [
        {
            "CreatedOn": "2026-05-06T15:09:20.13Z",
            "DocumentId": "5780154d-9cf9-4789-9374-5dffd158199c",
            "DocumentName": "image (3).png",
            "FileType": "png",
            "IsDeleted": false,
            "IsUploadedFromWeb": true
        }
    ],
    "AdditionalMetadata": {},
    "MentionedUsers": [],
    "ClientId": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070",
    "ItemId": "415f16a6-db95-48da-9a81-5316b1854a6b"
}
```

**Parameters:**

| Param | Type | Default | Description |
|------|------|---------|-------------|
| `Content` | `string (HTML)` | required | HTML content of the comment inside `<article class="riqsit-note">` structure |
| `BackgroundColor` | `string` | `"none"` | Background color of the note/comment card |
| `Possition` | `object` | `{ XCoordinate: 0, YCoordinate: 0 }` | Position metadata (currently unused, always 0,0) |
| `Attachments` | `Attachment[]` | `[]` | List of attached files (images, PDFs, etc.) |
| `AdditionalMetadata` | `object` | `{}` | Extra metadata key-value pairs |
| `MentionedUsers` | `MentionedUser[]` | `[]` | List of tagged/mentioned users |
| `ClientId` | `string (GUID)` | required | Client identifier for the request |
| `ItemId` | `string (GUID)` | required | ID of the comment being updated |

**Success Response:** See [Success Response](#success-response).
 
**Returns:** `Observable<any>` — emits the command response confirming whether the comment was successfully updated.

---


### Delete RIQS Note Comment
### `deleteRiqsNoteComment(itemId)`

Deletes an existing comment on a RIQS Note.
> **Permission:** Professional and Management users can delete any note comment. All other users may only delete their own comment .

**Endpoint:** `POST PraxisMonitorCommand/DeleteRIQSNoteComment`

**Payload:**
```json
{
    "ItemId": "62371ebc-1f0e-4d3e-a32f-13559b9d6748"
}
```

**Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `ItemId` | `string (GUID)` | required | ID of the comment to be deleted |

**Success Response:** See [Success Response](#success-response).

**Returns:** `Observable<any>` — emits the command response confirming whether the comment was successfully deleted.

---
### Get Unread Notifications
### `getUnreadNotifications(userId, clientId)`

Retrieves all notifications for a user filtered by a RIQS Note subscription context, and maps them to a read-status shape.

**Endpoint:** `POST api/Notifier/GetUnreadNotificationsBySubscriptionFilter`

**Payload:**
```json
{
    "UserId": "da86ecd0-08eb-4995-8803-dfa6c09ad134",
    "SubscriptionFilterData": {
        "Context": "RIQSNote",
        "ActionName": "NotifyUsers",
        "Value": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070"
    }
}
```

**Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `UserId` | `string (GUID)` | required | ID of the user whose notifications are being fetched |
| `SubscriptionFilterData.Context` | `string` | `"RIQSNote"` | The notification domain/context to filter by |
| `SubscriptionFilterData.ActionName` | `string` | `"NotifyUsers"` | The action event name to filter by |
| `SubscriptionFilterData.Value` | `string (GUID)` | required | Client ID used as the subscription filter value |

**Success Response:**
```json
[
    {
        "Id": "69f716f51603c677feafaa34",
        "CorrelationId": null,
        "Payload": {
            "UserId": "da86ecd0-08eb-4995-8803-dfa6c09ad134",
            "SubscriptionFilters": [
                {
                    "Context": "RIQSNote",
                    "ActionName": "NotifyUsers",
                    "Value": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070"
                }
            ],
            "NotificationType": "UserSpecificReceiverType",
            "ResponseKey": "b9a1fcf1-2193-48b0-8722-c3d84c0ae070",
            "ResponseValue": "{\"Success\":true}",
            "UserItemId": "da86ecd0-08eb-4995-8803-dfa6c09ad134"
        },
        "DenormalizedPayload": "{ RIQSNoteId = 16e33d85-73de-46e1-b31d-36e19f0e2ad1 }",
        "CreatedTime": "2026-05-03T09:35:49.842Z",
        "ReadByUserIds": ["da86ecd0-08eb-4995-8803-dfa6c09ad134"],
        "ReadByRoles": null,
        "IsRead": true
    }
]
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `Id` | `string` | Unique notification record ID |
| `Payload.UserId` | `string (GUID)` | The target user for this notification |
| `Payload.NotificationType` | `string` | Delivery type; always `"UserSpecificReceiverType"` here |
| `DenormalizedPayload` | `string` | Human-readable string containing the associated `RIQSNoteId` |
| `CreatedTime` | `string (ISO 8601)` | Timestamp when the notification was created |
| `ReadByUserIds` | `string[] \| null` | List of user IDs who have read this notification; `null` if unread |
| `IsRead` | `boolean` | Whether the notification has been read by the current user |

**Mapped Return Shape (`INotificationReadStatus`):**

The raw response is mapped before being emitted. Each notification item is transformed as follows:

| Field | Source | Description |
|-------|--------|-------------|
| `NotificationId` | `item.Id` | The notification record ID |
| `NoteId` | Parsed from `item.DenormalizedPayload` via regex `/RIQSNoteId\s*=\s*([a-zA-Z0-9-]+)/` | The associated RIQS Note ID; empty string if not found |
| `ReadByUserIds` | `item.ReadByUserIds` | Passed through as-is from the raw response |

**Returns:** `Observable<INotificationReadStatus[]>` — emits the mapped list of notification read statuses for the given user and client subscription.

---
### Mark Notification As Read
### `markNotificationAsRead(id)`

Marks one or more notifications as read for the current user.

**Endpoint:** `POST /api/notification/v3//api/Notifier/UpdateNotificationStatusToRead`

**Payload:**
```json
{
    "UserId": "3f8841aa-d437-46a4-b6ad-a62353dd3a5c",
    "NotificationIds": ["69ce891947303dc3081a5b90"]
}
```

**Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `UserId` | `string (GUID)` | required | ID of the user marking the notification as read |
| `NotificationIds` | `string[]` | required | Array of notification IDs to mark as read; wraps the single `id` argument in an array |

**Success Response:** See [Success Response](#success-response).

**Returns:** `Observable<any>` — emits the command response confirming whether the notification was successfully marked as read.

---
### Update RIQS Note Order
### `updateRIQSNoteOrder(noteId, orderNumber)`

Updates the display order of a RIQS Note, used when a note is dragged and dropped to a new position.

> **Permission:** Professional and Management users can reorder any note. All other users may only reorder notes they own.

**Endpoint:** `POST PraxisMonitorCommand/UpdateRIQSNoteOrder`

**Payload:**
```json
{
    "NoteId": "49695467-d170-4781-af8b-1863c1d64a29",
    "OrderNumber": 15.3125
}
```

**Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `NoteId` | `string (GUID)` | required | ID of the note being reordered |
| `OrderNumber` | `number (float)` | required | New order value calculated from surrounding notes (see Order Calculation below) |

**Order Number Calculation**

When a note is moved to a new position, its `OrderNumber` is calculated as the midpoint between the order numbers of its new neighbours:

```
OrderNumber = (prevNote.OrderNumber + nextNote.OrderNumber) / 2
```

This allows inserting notes between existing ones without renumbering the entire list.

| Scenario | Calculation | Example |
|----------|-------------|---------|
| Between two notes | `(prev + next) / 2` | `(10 + 20) / 2 = 15` |
| Moved to first position | `nextNote.OrderNumber / 2` | `10 / 2 = 5` |
| Moved to last position | `prevNote.OrderNumber + some increment` | `20 + 10 = 30` |

**Success Response:** See [Success Response](#success-response).

**Returns:** `Observable<any>` — emits the command response confirming whether the note order was successfully updated.

> **Note:** Using midpoint insertion keeps `OrderNumber` values as floats and avoids reindexing the full list on every reorder. Over many reorders, values may converge; a full reindex may be needed if precision is exhausted.

---
### Pin / Unpin RIQS Note
### `updateNotePinnedStatus(noteId, isPinned)`

Toggles the pinned state of a RIQS Note.

> **Permission:** Only Professional and Management users can pin any note in unit view.

> **Note:** The `IsPinned` value sent in the payload is the **inverse** of the note's current state — i.e. `!isPinned` — effectively toggling it.

**Endpoint:** `POST PraxisMonitorCommand/PinRIQSNote`

**Payload:**
```json
{
    "NoteId": "dbae1999-0316-4b28-98e4-3c8cc574abb6",
    "IsPinned": false
}
```

**Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `NoteId` | `string (GUID)` | required | ID of the note to pin or unpin |
| `IsPinned` | `boolean` | required | The new pinned state; always passed as `!currentIsPinned` to toggle |

**Success Response:** See [Success Response](#success-response).

**Returns:** `Observable<any>` — emits the command response confirming whether the note's pinned status was successfully updated.

---
### Get Pre-Signed URL For Upload
### `getPreSignedUrlForUpload(payload)`

Requests a pre-signed URL from the storage service to allow direct file upload for a RIQS Note attachment.

**Endpoint:** `POST StorageQuery/GetPreSignedUrlForUpload`

**Payload:**
```json
{
    "ItemId": "83dfb00a-680b-4bfb-9131-6bedc6d94da4",
    "Name": "image (3).png",
    "Tags": "[\"File-Of-Note\",\"File\"]",
    "MetaData": "{\"FileInfo\":{\"Value\":\"image (3).png\",\"Type\":\"image/png\"}}",
    "ParentDirectoryId": ""
}
```

**Parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `ItemId` | `string (GUID)` | required | Unique ID for the file being uploaded; typically pre-generated on the client |
| `Name` | `string` | required | Original filename including extension |
| `Tags` | `string (JSON)` | required | JSON-stringified array of tags categorising the file (e.g. `["File-Of-Note", "File"]`) |
| `MetaData` | `string (JSON)` | required | JSON-stringified object containing file info such as the original filename and MIME type |
| `ParentDirectoryId` | `string` | `""` | ID of the parent directory in storage; empty string for root-level uploads |

**Returns:** `Observable<any>` — emits the pre-signed URL response from the storage service, which the client then uses to upload the file directly.

---
## Common Response

### Success Response
All endpoints return the following response on success:

```json
{
    "Errors": {
        "IsValid": true,
        "Errors": [],
        "RuleSetsExecuted": null
    },
    "ErrorMessages": [],
    "StatusCode": 0,
    "RequestUri": null,
    "ExternalError": null,
    "HttpStatusCode": 0
}
```

