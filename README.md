# Freelance Marketplace Server

A robust Node.js/Express.js server application providing backend services for a freelance marketplace platform. Features include authentication, messaging, gig management, orders, and reviews.

## Core Components

### 1. Database Configuration (configs/db.js)
- Uses Mongoose for MongoDB connection
- Configures strict query mode
- Connects to MongoDB using environment variable MONGODB_URI

### 2. Models
All models use Mongoose schemas:

#### User Model
- Username (unique)
- Email
- Password
- Country
- Image
- Seller status
- Timestamps enabled

#### Gig Model
- User reference
- Title
- Description
- Category
- Price
- Images
- Features
- Sales count
- Star rating
- Timestamps enabled

#### Message Model
- Conversation ID
- User ID
- Message content
- No timestamps

#### Conversation Model
- Unique conversation ID (UUID)
- Seller ID
- Buyer ID
- Read status
- Timestamps enabled

#### Order Model
- Gig reference
- Buyer/Seller references
- Price
- Payment intent
- Completion status

#### Review Model
- Gig reference
- User reference
- Star rating
- Description

### 3. Controllers

#### Auth Controller
- Register: User registration with password hashing
- Login: Authentication with JWT
- Logout: Cookie clearing
- Status check: Current user verification

#### Gig Controller
- Create: New gig creation (sellers only)
- Delete: Gig removal
- Get: Single gig retrieval
- List: Filtered gig listing

#### Message Controller
- Create: New message creation
- List: Conversation messages retrieval

#### Order Controller
- List: Order history
- Payment: Stripe integration
- Status update: Payment confirmation

#### Review Controller
- Create: New review submission
- Get: Gig reviews retrieval
- Delete: Review removal

### 4. Middlewares

#### Authentication Middleware
- JWT verification
- Cookie validation
- User session management

#### User Middleware
- Token validation
- User role verification
- Request augmentation with user data

#### Error Middleware
- Centralized error handling
- Status code management
- Error response formatting

### 5. Routes

#### Auth Routes

POST /auth/register
POST /auth/login
POST /auth/logout
GET /auth/me

#### Gig Routes

POST /gigs
DELETE /gigs/:id
GET /gigs/single/:id
GET /gigs

#### Message Routes

POST /messages
GET /messages/:conversationId

#### Order Routes

GET /orders
POST /orders/create-payment-intent/:id
PATCH /orders

#### Review Routes

POST /reviews
GET /reviews/:gigId
DELETE /reviews/:id

### 6. Utilities

#### CustomException
- Custom error handling
- Status code specification
- Error message formatting

## Security Features
1. JWT-based authentication
2. Password hashing with bcrypt
3. Cookie-based session management
4. Role-based access control
5. Input validation
6. Error handling

## API Response Format
```javascript
{
    error: boolean,
    message: string,
    data?: any
}
```

## Error Handling
Centralized error handling through errorMiddleware with:
- Custom status codes
- Descriptive error messages
- Standardized error response format

## Dependencies
- express: Web framework
- mongoose: MongoDB ODM
- jsonwebtoken: Authentication
- bcrypt: Password hashing
- cookie-parser: Cookie handling
- stripe: Payment processing
- uuid: Unique ID generation
- satelize: Geolocation

## Environment Variables
Required environment variables:
```
MONGODB_URI: Database connection string
JWT_SECRET: JWT signing key
STRIPE_SECRET: Stripe API key
NODE_ENV: Environment mode
```

## API Endpoints Details

### Auth Endpoints

#### POST /auth/register
Creates a new user account
Request body:
```javascript
{
    username: string,
    email: string,
    password: string,
    phone: string,
    image: string, // optional
    isSeller: boolean,
    description: string
}
```

#### POST /auth/login
Authenticates a user
Request body:
```javascript
{
    username: string,
    password: string
}
```

#### POST /auth/logout
Logs out current user
No request body required

#### GET /auth/me
Gets current user information
Requires authentication token

### Gig Endpoints

#### POST /gigs
Creates a new gig
Requires seller authentication
Request body:
```javascript
{
    title: string,
    description: string,
    category: string,
    price: number,
    cover: string,
    images: string[],
    shortTitle: string,
    shortDesc: string,
    deliveryTime: string,
    revisionNumber: number,
    features: string[]
}
```

#### DELETE /gigs/:id
Deletes a gig
Requires gig owner authentication

#### GET /gigs/single/:id
Gets detailed gig information

#### GET /gigs
Lists gigs with optional filters
Query parameters:
```javascript
{
    category: string,
    search: string,
    max: number,
    min: number,
    sort: string,
    userId: string
}
```

## Database Schema Details

### User Schema
```javascript
{
    username: { type: String, required: true, unique: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true },
    image: { type: String, required: false },
    country: { type: String, required: true },
    phone: { type: String, required: true },
    description: { type: String, required: true },
    isSeller: { type: Boolean, default: false }
}
```

### Gig Schema
```javascript
{
    userId: { type: ObjectId, ref: 'User', required: true },
    title: { type: String, required: true },
    description: { type: String, required: true },
    totalStars: { type: Number, default: 0 },
    starNumber: { type: Number, default: 0 },
    category: { type: String, required: true },
    price: { type: Number, required: true },
    cover: { type: String, required: true },
    images: { type: [String], required: false },
    shortTitle: { type: String, required: true },
    shortDesc: { type: String, required: true },
    deliveryTime: { type: String, required: true },
    revisionNumber: { type: Number, required: true },
    features: { type: [String], required: false },
    sales: { type: Number, default: 0 }
}
```

### Message Schema
```javascript
{
    conversationId: { type: String, required: true },
    userId: { type: ObjectId, ref: 'User', required: true },
    description: { type: String, required: true }
}
```

### Conversation Schema
```javascript
{
    conversationId: { type: String, default: uuid },
    sellerId: { type: ObjectId, ref: 'User', required: true },
    buyerId: { type: ObjectId, ref: 'User', required: true },
    readBySeller: { type: Boolean, required: true },
    readByBuyer: { type: Boolean, required: true },
    lastMessage: { type: String, required: false }
}
```

### Order Schema
```javascript
{
    gigId: { type: ObjectId, ref: 'Gig', required: true },
    image: { type: String, required: false },
    title: { type: String, required: true },
    price: { type: Number, required: true },
    sellerId: { type: ObjectId, ref: 'User', required: true },
    buyerId: { type: ObjectId, ref: 'User', required: true },
    isCompleted: { type: Boolean, default: false },
    payment_intent: { type: String, required: true }
}
```

### Review Schema
```javascript
{
    gigId: { type: ObjectId, ref: 'Gig', required: true },
    userId: { type: ObjectId, ref: 'User', required: true },
    star: { type: Number, required: true, max: 5 },
    description: { type: String, required: true }
}
```

## Installation

1. Clone the repository
2. Install dependencies:
```bash
npm install
```
3. Create .env file with required environment variables
4. Run the server:
```bash
# Development
npm run dev

# Production
npm start
```
