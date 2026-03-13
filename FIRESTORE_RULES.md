# 🔥 Required Firestore Security Rules

## ⚠️ IMPORTANT — Update these rules to fix the Follow button permission error

Go to Firebase Console → Firestore → Rules and paste this:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Any logged-in user can read/create/update ALL posts
    // Only the post author can delete their own post
    match /posts/{postId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null;
      allow delete: if request.auth != null && request.auth.uid == resource.data.authorId;

      // Comments: any logged-in user can read/write
      match /comments/{commentId} {
        allow read, write: if request.auth != null;
      }
    }

    // Users can read all profiles
    // Users can update their own full profile
    // ANY logged-in user can update followersList/followers on any profile
    //   (needed so follow/unfollow works without Cloud Functions)
    match /users/{userId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update: if request.auth != null && (
        request.auth.uid == userId ||
        // Allow any logged-in user to update only follower-related fields
        request.resource.data.diff(resource.data).affectedKeys()
          .hasOnly(['followersList', 'followers'])
      );
    }
  }
}
```

## Steps:
1. Go to https://console.firebase.google.com
2. Select your project: **loginauthproject-34e2a**
3. Click **Firestore Database** in the left menu
4. Click the **Rules** tab at the top
5. Replace ALL existing rules with the rules above
6. Click **Publish**

## Why this update is needed:
The follow button was showing a permission error because when User A follows User B, it needs to write to **User B's** document (to update their `followersList`). The old rules only allowed users to update their **own** doc. The new rules add a special exception that allows any logged-in user to update only the `followersList` and `followers` fields on any user doc — nothing else.
