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

---

### `createNote(payload)`

Creates a new RIQS Note.

**Endpoint:** `POST PraxisMonitorCommand/CreateRIQSNote`

**Payload:**
```json
{
  "Title": "string",
  "Description": "string",
  "ClientId": "string",
  "AssignedTo": "string (optional)",
  "OrderNumber": 0
}
```

**Returns:** `Observable<any>` — response body of the created note.

---

### `updateNote(payload)`

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

### `getRiqsNotes(filters?)`

Fetches a list of RIQS Notes, optionally filtered.

**Endpoint:** `POST PraxisMonitorQuery/GetRIQSNote`

**Parameters:**
| Param | Type | Default | Description |
|---|---|---|---|
| `filters` | `Filter[]` | `[]` | Array of filter objects to narrow results |

**Internal Payload sent to API:**
```json
{
  "FilterString": "{<generated from filters>}",
  "SortedBy": "{\"OrderNumber\": -1}",
  "PageSize": 1000000,
  "PageNumber": 0
}
```

**Returns:** `Observable<any[]>` — array of notes, sorted by `OrderNumber` descending.

---

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
