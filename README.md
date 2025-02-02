# Skill Swap Platform 🔄

An open-source platform for trading skills using a credit system. Built with **Next.js, Tailwind, TypeScript, and PostgreSQL**.

---

## Table of Contents
- [Backend Workflow](#backend-workflow)
  - [Authentication](#authentication)
  - [Skill Matching](#skill-matching)
  - [Transaction Flow](#transaction-flow)
  - [Realtime Chat](#realtime-chat)
- [Full-Stack Interaction](#full-stack-interaction)
- [Setup](#setup)

---

## 🔄 Backend Workflow

### Authentication
- **NextAuth.js** handles Discord/Google OAuth authentication.
- **JWT tokens** are used for session management to secure the API routes.

#### Example of Session Handling:
```typescript
import { getSession } from "next-auth/react";

export async function requireAuthentication(req: NextApiRequest, res: NextApiResponse) {
  const session = await getSession({ req });
  if (!session) {
    return res.status(401).json({ error: "Not authenticated" });
  }
  return session;
}
```

### Skill Matching
**Endpoint**: `GET /api/matches`  
This endpoint finds users whose skills align with the current user's needs, enabling skill exchanges.

#### Example Logic:
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

#### Workflow:
- Current user’s offers and needs are matched with others.
- Returns a list of potential users for skill exchange.

### Transaction Flow
**Endpoint**: `POST /api/transactions`  
This API handles the flow of credits during a transaction.

#### Steps:
1. **Deduct credits** from the sender's account.
2. Create a `pending` transaction record in the database.
3. **Notify the receiver** via SMS/email using Twilio/Resend APIs.
4. **Release credits** once both parties confirm the trade.

#### Example Flow:
```typescript
const transaction = await prisma.transaction.create({
  data: {
    senderId: currentUser.id,
    receiverId: matchUser.id,
    credits: 1,
    status: 'pending',
  }
});
```

### Realtime Chat
**Realtime** communication between users is powered by **Ably**. Messages are sent and received instantly for negotiating trades.

#### Message Schema Example:
```json
{
  "senderId": "user_123",
  "content": "Can you start tomorrow?",
  "channel": "trade_456"
}
```

- Each message is sent to an **Ably channel** associated with the trade.
- Real-time updates are reflected across both users' interfaces.

---

## 🔄 Full-Stack Interaction

The full-stack workflow diagram illustrates the interactions between the frontend, backend, and external services.

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

#### Flow Explanation:
1. **Frontend** sends a request to the **Backend** to fetch matches.
2. **Backend** queries the **Database** and returns matching users.
3. **Frontend** subscribes to a specific **Ably channel** for live chat.
4. **Frontend** triggers a **POST request** to start a transaction.
5. **Backend** creates a transaction record in the **Database** and sends a notification using **Twilio**.
6. The transaction is confirmed and **Frontend** receives the status update.

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

