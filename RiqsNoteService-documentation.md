# RiqsNoteService Documentation

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

## Properties

### `navbarChanged$`
- **Type:** `BehaviorSubject<boolean>`
- **Purpose:** Emits when the navbar state should refresh (e.g., after notification changes).

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
 
**Success Response:**
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

**Success Response:**
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

**Returns:** `Observable<any>`

Updates an existing RIQS Note.

**Endpoint:** `POST PraxisMonitorCommand/UpdateRIQSNote`

**Payload:**
```json
{
  "NoteId": "string (GUID)",
  "Title": "string",
  "Description": "string",
  "AssignedTo": "string (optional)"
}
```

**Returns:** `Observable<any>` — response body of the updated note.

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
| `StatusCode` | `number` | Application-level status code. `0` = success |
| `ErrorMessage` | `string \| null` | Error message if request fails, otherwise `null` |
| `TotalRecordCount` | `number` | Total number of notes available (useful for pagination) |
| `Results` | `RIQSNote[]` | Array of returned notes |

---

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

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `itemId` | `string` | The ID of the note to delete |

**Payload sent:**
```json
{
  "ItemId": "string"
}
```

**Returns:** `Observable<any>` — response body confirming deletion.

---

### `getRiqsNoteComments(noteId)`

Fetches all comments for a specific RIQS Note.

**Endpoint:** `POST PraxisMonitorQuery/GetRIQSNoteComments`

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `noteId` | `string` | The ID of the note to fetch comments for |

**Internal Payload sent to API:**
```json
{
  "FilterString": "{\"RIQSNoteId\": \"<noteId>\"}",
  "SortedBy": "{\"CreateDate\": -1}",
  "PageSize": 1000000,
  "PageNumber": 0
}
```

**Returns:** `Observable<any[]>` — list of comments, sorted by `CreateDate` descending (newest first).

---

### `saveRiqsNoteComment(payload)`

Creates a new comment on a RIQS Note.

**Endpoint:** `POST PraxisMonitorCommand/CreateRIQSNoteComment`

**Payload:**
```json
{
  "RIQSNoteId": "string (GUID)",
  "Comment": "string",
  "CreatedBy": "string (userId)"
}
```

**Returns:** `Observable<any>` — response body of the created comment.

---

### `updateRiqsNoteComment(payload)`

Updates an existing comment on a RIQS Note.

**Endpoint:** `POST PraxisMonitorCommand/UpdateRIQSNoteComment`

**Payload:**
```json
{
  "ItemId": "string (comment GUID)",
  "Comment": "string (updated text)"
}
```

**Returns:** `Observable<any>` — response body of the updated comment.

---

### `deleteRiqsNoteComment(itemId)`

Deletes a comment by ID.

**Endpoint:** `POST PraxisMonitorCommand/DeleteRIQSNoteComment`

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `itemId` | `string` | The ID of the comment to delete |

**Payload sent:**
```json
{
  "ItemId": "string"
}
```

**Returns:** `Observable<any>` — response body confirming deletion.

---

### `getUnreadNotifications(userId, clientId)`

Fetches all unread notifications for a user scoped to a specific client's RIQS Notes.

**Endpoint:** `POST NotificationService/api/Notifier/GetUnreadNotificationsBySubscriptionFilter`

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `userId` | `string` | The ID of the user to fetch notifications for |
| `clientId` | `string` | The client context to filter notifications |

**Payload sent:**
```json
{
  "UserId": "string",
  "SubscriptionFilterData": {
    "Context": "RIQSNote",
    "ActionName": "NotifyUsers",
    "Value": "string (clientId)"
  }
}
```

**Returns:** `Observable<INotificationReadStatus[]>`

Each item in the array has this shape:
```typescript
{
  NotificationId: string;   // The notification's ID
  NoteId: string;           // Extracted from DenormalizedPayload via regex
  ReadByUserIds: string[];  // Users who have already read this notification
}
```

> **Note:** `NoteId` is parsed from the `DenormalizedPayload` field using the pattern `RIQSNoteId = <guid>`. If no match is found, `NoteId` will be an empty string.

---

### `markNotificationAsRead(id)`

Marks a single notification as read.

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `id` | `string` | The notification ID to mark as read |

**Delegates to:** `NotificationService.markNotificationAsRead([id])`

**Returns:** `Observable<any>` — response body confirming the update.

---

### `updateRIQSNoteOrder(noteId, orderNumber)`

Updates the display order of a RIQS Note.

**Endpoint:** `POST PraxisMonitorCommand/UpdateRIQSNoteOrder`

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `noteId` | `string` | The ID of the note to reorder |
| `orderNumber` | `number` | The new order position |

**Payload sent:**
```json
{
  "NoteId": "string",
  "OrderNumber": 0
}
```

**Returns:** `Observable<any>` — response body confirming the order update.

---

### `updateNotePinnedStatus(noteId, isPinned)`

Toggles the pinned status of a RIQS Note.

**Endpoint:** `POST PraxisMonitorCommand/PinRIQSNote`

**Parameters:**
| Param | Type | Description |
|---|---|---|
| `noteId` | `string` | The ID of the note to pin/unpin |
| `isPinned` | `boolean` | The **current** pinned state (the service will automatically invert it) |

**Payload sent:**
```json
{
  "NoteId": "string",
  "IsPinned": true
}
```

> **Important:** The service sends `!isPinned` to the API — so pass the **current** value, not the desired one.

**Returns:** `Observable<any>` — response body confirming the pin status update.

---

## Models

### `INotification`
Represents a raw notification from the notification service.
```typescript
interface INotification {
  Id: string;
  DenormalizedPayload: string;  // Contains "RIQSNoteId = <guid>"
  ReadByUserIds: string[];
}
```

### `INotificationReadStatus`
Simplified notification shape returned by `getUnreadNotifications()`.
```typescript
interface INotificationReadStatus {
  NotificationId: string;
  NoteId: string;
  ReadByUserIds: string[];
}
```

---

## Dependencies

| Dependency | Purpose |
|---|---|
| `HttpClient` | All HTTP requests |
| `CommonService` | Shared utility methods |
| `GqlQueryBuilderService` | Builds `FilterString` from `Filter[]` arrays |
| `NotificationService` | Marks notifications as read |
| `ShellDomainProvider` | Provides base URLs for APIs |
