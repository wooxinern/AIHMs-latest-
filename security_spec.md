# Security Specification - AIHMs Guardian

## 1. Data Invariants

- **Identity Isolation**: A user can only access their own private data (`healthLogs`, `chatHistory`, etc.) unless they are a linked caregiver or an admin.
- **Relational Integrity**: A caregiver can only access a patient's data if the patient's record explicitly lists the caregiver's UID in the `caregiverIds` array.
- **Admin Supremacy**: Users with a record in the `/admins/` collection can access all resources for maintenance and emergency oversight.
- **Immutable Provenance**: Fields like `uid` on a user Profile or `authorId` on a `CommunityPost` cannot be changed after creation.
- **Temporal Sanitization**: `createdAt` and `updatedAt` fields must always match `request.time`.
- **Relational Locking**: A caregiver cannot link to a patient unless they have a valid `inviteToken` from that patient.

## 2. The Dirty Dozen Payloads

### P1: The Identity Thief (Cross-User Write)
- **Target**: `setDoc(/users/VictimUID, payload)`
- **Payload**: `{ "uid": "VictimUID", "email": "attacker@evil.com", "role": "admin" }`
- **Expected**: `PERMISSION_DENIED` (User can't write to a profile that isn't their own).

### P2: The Shadow Admin (Privilege Escalation)
- **Target**: `updateDoc(/users/AttackerUID, payload)`
- **Payload**: `{ "role": "admin" }`
- **Expected**: `PERMISSION_DENIED` (Users cannot change their own role to admin).

### P3: The PII Scraper (Unauthorized Read)
- **Target**: `getDoc(/users/VictimUID/healthLogs/LogID)`
- **Payload**: `N/A` (Authenticated but not linked)
- **Expected**: `PERMISSION_DENIED`.

### P4: The Ghost Message (Identity Spoofing)
- **Target**: `addDoc(/communityPosts, payload)`
- **Payload**: `{ "authorId": "VictimUID", "text": "I am a victim", "title": "Help" }`
- **Expected**: `PERMISSION_DENIED` (authorId must match request.auth.uid).

### P5: The History Rewriter (Immutable field violation)
- **Target**: `updateDoc(/communityPosts/PostID, payload)`
- **Payload**: `{ "authorId": "AttackerUID" }`
- **Expected**: `PERMISSION_DENIED` (authorId is immutable).

### P6: The Time Traveler (Timestamp Manipulation)
- **Target**: `addDoc(/users/UID/healthLogs, payload)`
- **Payload**: `{ "createdAt": "2020-01-01T00:00:00Z" }`
- **Expected**: `PERMISSION_DENIED` (createdAt must be request.time).

### P7: The Spam King (Resource Exhaustion)
- **Target**: `addDoc(/feedback, payload)`
- **Payload**: `{ "content": "A".repeat(2000000) }` (Too large)
- **Expected**: `PERMISSION_DENIED` (Size constraints).

### P8: The Orphan Maker (Invalid ID injection)
- **Target**: `doc(/users/./../invalid/path)`
- **Payload**: `N/A`
- **Expected**: `PERMISSION_DENIED` (isValidId regex guard).

### P9: The Shadow Like (Double Counting)
- **Target**: `updateDoc(/communityPosts/PostID, payload)`
- **Payload**: `{ "likes": 999999 }`
- **Expected**: `PERMISSION_DENIED` (Likes must be updated via increment(1) or validated logic).

### P10: The Token Bypass (Caregiver Linkage without token)
- **Target**: `updateDoc(/users/PatientUID, payload)`
- **Payload**: `{ "caregiverIds": ["AttackerUID"] }` (Without valid token in data or exists logic)
- **Expected**: `PERMISSION_DENIED`.

### P11: The PII Leaker (Global List query)
- **Target**: `getDocs(query(collection(db, 'users')))`
- **Payload**: `N/A`
- **Expected**: `PERMISSION_DENIED` (List reads on /users/ must be restricted or filtered).

### P12: The Comment Spoofer (Spoofing author)
- **Target**: `addDoc(/communityPosts/PostID/comments, payload)`
- **Payload**: `{ "authorId": "VictimUID", "text": "Hacked" }`
- **Expected**: `PERMISSION_DENIED`.


## 3. Test Runner (Conceptual)

```typescript
// firestore.rules.test.ts
import { assertFails, assertSucceeds, initializeTestEnvironment } from '@firebase/rules-unit-testing';

// ... test implementation for all 12 cases ...
```
