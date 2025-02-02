
# Skill Swap Platform: Full-Stack & Backend Flow 🔄

This document outlines the **Full-Stack Interaction** and **Backend Workflow** for the Skill Swap Platform. It includes diagrams and detailed explanations of how data flows between the frontend, backend, and database.

---

## Table of Contents
- [Full-Stack Flow](#-full-stack-flow)
  - [Diagram](#full-stack-flow-diagram)
  - [Explanation](#full-stack-flow-explanation)
- [Backend Flow](#-backend-flow)
  - [Diagram](#backend-flow-diagram)
  - [Explanation](#backend-flow-explanation)

---

## 🚀 Full-Stack Flow

### Full-Stack Flow Diagram
```mermaid
sequenceDiagram
  participant Frontend
  participant Backend
  participant Database
  participant Ably
  participant Twilio

  Frontend->>Backend: GET /api/matches (with JWT)
  Backend->>Database: Query compatible users
  Database-->>Backend: Return matches
  Backend-->>Frontend: Display matches

  Frontend->>Ably: Subscribe to trade_123 channel
  Frontend->>Backend: POST /api/transactions
  Backend->>Database: Create transaction
  Backend->>Twilio: Send SMS notification
  Twilio-->>Backend: Delivery status
  Backend-->>Frontend: 201 Created
```

### Full-Stack Flow Explanation
1. **Frontend Requests Matches**:
   - The frontend sends a `GET /api/matches` request to the backend with a JWT token for authentication.
   - The backend queries the database for users with complementary skills.

2. **Database Returns Matches**:
   - The database returns a list of users whose `offers` match the current user's `needs` and vice versa.
   - The backend formats this data and sends it back to the frontend.

3. **Frontend Displays Matches**:
   - The frontend renders the matches in a card-based UI, allowing the user to browse and select potential trades.

4. **Realtime Chat Initialization**:
   - When a user selects a match, the frontend subscribes to an Ably channel for realtime messaging.

5. **Transaction Proposal**:
   - The frontend sends a `POST /api/transactions` request to propose a trade.
   - The backend creates a `pending` transaction in the database and deducts credits from the sender.

6. **Notification Trigger**:
   - The backend sends an SMS notification to the receiver via Twilio.
   - Once the receiver confirms the trade, credits are transferred, and the transaction status is updated to `completed`.

---

## 🛠 Backend Flow

### Backend Flow Diagram
```mermaid
graph TD
  A[Auth Middleware] -->|Validate JWT| B[Match Query]
  B -->|Find Users| C[Database]
  C -->|Return Matches| D[Format Response]
  D -->|Send to Frontend| E[Frontend]

  F[Transaction Request] -->|Validate Input| G[Create Transaction]
  G -->|Deduct Credits| H[Database]
  H -->|Notify Receiver| I[Twilio]
  I -->|Update Status| J[Database]
  J -->|Send Confirmation| K[Frontend]
```

### Backend Flow Explanation
1. **Authentication**:
   - All requests pass through an auth middleware that validates the JWT token.
   - Unauthorized requests are rejected with a `401 Unauthorized` response.

2. **Match Query**:
   - The backend queries the database for users whose `offers` and `needs` overlap with the current user's profile.
   - Example Prisma query:
     ```typescript
     const matches = await prisma.user.findMany({
       where: {
         AND: [
           { offers: { hasSome: currentUser.needs } },
           { needs: { hasSome: currentUser.offers } }
         ],
         NOT: { id: currentUser.id }
       }
     });
     ```

3. **Transaction Creation**:
   - When a trade is proposed, the backend validates the input (e.g., hours, skill).
   - It deducts credits from the sender and creates a `pending` transaction in the database.

4. **Notification System**:
   - The backend triggers an SMS notification via Twilio to inform the receiver of the trade proposal.
   - Once both parties confirm, the transaction status is updated to `completed`, and credits are transferred.

5. **Realtime Chat**:
   - The backend initializes Ably channels for realtime messaging between users.
   - Messages are persisted in the database for future reference.

---

## 🗂 Database Schema
```prisma
model User {
  id          String       @id @default(uuid())
  email       String?      @unique
  name        String?
  image       String?
  discordId   String?      @unique
  credits     Int          @default(5)
  offers      String[]     // Skills they offer
  needs       String[]     // Skills they need
  transactionsSent Transaction[] @relation("SentTransactions")
  transactionsReceived Transaction[] @relation("ReceivedTransactions")
  createdAt   DateTime     @default(now())
}

model Transaction {
  id          String      @id @default(uuid())
  fromUserId  String      // User who sent credits
  fromUser    User        @relation("SentTransactions", fields: [fromUserId], references: [id])
  toUserId    String      // User who received credits
  toUser      User        @relation("ReceivedTransactions", fields: [toUserId], references: [id])
  hours       Int         // Hours exchanged
  skill       String      // Skill traded
  status      String      @default("pending") // pending/completed/cancelled
  createdAt   DateTime    @default(now())
}
```

---

## 🛠 Setup

### Requirements
- Node.js v18+
- PostgreSQL database

### Installation
```bash
git clone https://github.com/yourusername/skill-swap.git
cd skill-swap
npm install
cp .env.example .env.local
npx prisma migrate dev
npm run dev
```

### Environment Variables (`.env.local`)
```ini
DATABASE_URL="postgresql://user:pass@localhost:5432/skillswap"
NEXTAUTH_SECRET="your_secret_key"
DISCORD_CLIENT_ID="your_discord_id"
DISCORD_CLIENT_SECRET="your_discord_secret"
ABLY_API_KEY="your_ably_key"
TWILIO_ACCOUNT_SID="your_twilio_sid"
TWILIO_AUTH_TOKEN="your_twilio_token"
```

---

## 🤝 Contributing
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit changes (`git commit -m 'Add some feature'`)
4. Push to branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

License: [MIT](LICENSE)
