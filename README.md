# Skill Swap Platform 🔄

An open-source platform for trading skills using a credit system. Built with **Next.js, Tailwind, TypeScript, and PostgreSQL**.

[Live Demo](#) | [API Documentation](#) | [Contribution Guide](CONTRIBUTING.md)

---

## Table of Contents
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Workflow](#-workflow)
  - [Frontend Navigation](#frontend-navigation-flow)
  - [Backend Logic](#backend-workflow)
  - [Full-Stack Interaction](#full-stack-flow)
- [Setup](#-setup)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)

---

## 🚀 Features
- **Skill Trading**: Exchange services using credits (1 credit = 1 hour).
- **OAuth Login**: Sign up via Discord/Google.
- **Realtime Chat**: Negotiate trades instantly.
- **Escrow System**: Secure credit transactions.
- **Notifications**: SMS/email alerts for trades.

---

## 🛠 Tech Stack
| Layer          | Technologies                                                                 |
|----------------|-----------------------------------------------------------------------------|
| **Frontend**   | Next.js 14 (App Router), Tailwind CSS, TypeScript, shadcn-ui, Framer Motion |
| **Backend**    | Next.js API Routes, Prisma ORM                                              |
| **Database**   | PostgreSQL (hosted on Supabase)                                            |
| **Auth**       | NextAuth.js                                                                |
| **Realtime**   | Ably                                                                       |
| **APIs**       | Twilio (SMS), Resend (Email)                                               |

---

## 🔄 Workflow

### Frontend Navigation Flow
```mermaid
graph TD
  A[Landing Page] -->|Login/Signup| B[Dashboard]
  B --> C[Profile Settings]
  B --> D[Browse Matches]
  D -->|Select User| E[Chat Interface]
  B -->|Propose Trade| F[Transaction Modal]
  E -->|Confirm Trade| G[Transaction History]
