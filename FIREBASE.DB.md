// Database Collections Structure

users/{userId}
{
  username: string,
  email: string,
  image: string,
  country: string,
  phone: string,
  description: string,
  isSeller: boolean,
  createdAt: timestamp,
  updatedAt: timestamp
}

gigs/{gigId}
{
  userId: string,          // Reference to users collection
  title: string,
  description: string,
  totalStars: number,
  starNumber: number,
  category: string,
  price: number,
  cover: string,
  images: array[string],   // Array of image URLs
  shortTitle: string,
  shortDesc: string,
  deliveryTime: string,
  revisionNumber: number,
  features: array[string],
  sales: number,
  createdAt: timestamp
}

conversations/{conversationId}
{
  sellerId: string,        // Reference to users collection
  buyerId: string,         // Reference to users collection
  readBySeller: boolean,
  readByBuyer: boolean,
  lastMessage: string,
  createdAt: timestamp,
  updatedAt: timestamp
}

messages/{conversationId}/messages/{messageId}
{
  userId: string,          // Reference to users collection
  description: string,
  createdAt: timestamp
}

orders/{orderId}
{
  gigId: string,          // Reference to gigs collection
  image: string,
  title: string,
  price: number,
  sellerId: string,       // Reference to users collection
  buyerId: string,        // Reference to users collection
  isCompleted: boolean,
  paymentIntentId: string,
  createdAt: timestamp
}

reviews/{gigId}/reviews/{reviewId}
{
  userId: string,         // Reference to users collection
  star: number,          // 1-5 rating
  description: string,
  createdAt: timestamp
}


## Security Rules

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users Collection
    match /users/{userId} {
      allow read;
      allow create: if request.auth != null;
      allow update, delete: if request.auth.uid == userId;
    }
    
    // Gigs Collection
    match /gigs/{gigId} {
      allow read;
      allow create: if request.auth != null && 
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isSeller == true;
      allow update, delete: if request.auth.uid == resource.data.userId;
    }
    
    // Conversations Collection
    match /conversations/{conversationId} {
      allow read, write: if request.auth != null &&
        (request.auth.uid == resource.data.buyerId || 
         request.auth.uid == resource.data.sellerId);
    }
    
    // Messages Collection
    match /messages/{conversationId}/messages/{messageId} {
      allow read, write: if request.auth != null &&
        get(/databases/$(database)/documents/conversations/$(conversationId)).data.buyerId == request.auth.uid ||
        get(/databases/$(database)/documents/conversations/$(conversationId)).data.sellerId == request.auth.uid;
    }
    
    // Orders Collection
    match /orders/{orderId} {
      allow read, create: if request.auth != null;
      allow update: if request.auth != null &&
        (request.auth.uid == resource.data.buyerId || 
         request.auth.uid == resource.data.sellerId);
    }
    
    // Reviews Collection
    match /reviews/{gigId}/reviews/{reviewId} {
      allow read;
      allow create: if request.auth != null && 
        request.auth.uid == request.resource.data.userId &&
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isSeller == false;
      allow delete: if request.auth.uid == resource.data.userId;
    }
  }
}


## Additional Storage Rules

rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /users/{userId}/{allPaths=**} {
      allow read;
      allow write: if request.auth != null && request.auth.uid == userId;
    }
    
    match /gigs/{gigId}/{allPaths=**} {
      allow read;
      allow write: if request.auth != null && 
        get(/databases/$(database)/documents/gigs/$(gigId)).data.userId == request.auth.uid;
    }
  }
}
