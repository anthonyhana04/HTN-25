# SplitRooms

A mobile-first expense sharing app built with Expo React Native and AWS infrastructure. Split bills, track balances, and settle up with friends and roommates.

## Project Overview

SplitRooms is a **brand-free**, **CAD-only** expense sharing platform designed for the Canadian market. The app focuses on core bill splitting functionality with a clean, intuitive interface.

### Current Features
- Room-based expense organization
- Multiple split modes (equal, percentage, shares, per-item)
- Automatic balance calculations and settlement suggestions
- Local chat messaging within rooms
- Receipt upload preparation
- Demo user system

### Coming Soon
- **Auth** — Real user authentication system
- **OCR Parser** — Automatic receipt parsing and bill creation
- **Notifications** — Payment reminders and activity alerts
- **Realtime Chat** — Live messaging and updates

## Project Structure

```
/splitrooms
  /app           # Expo React Native (TypeScript)
  /infra         # AWS IaC (CDK or SAM) + seed + admin scripts
  /docs          # API contracts, data contracts, settlement algorithms
  .gitignore
  package.json   # Root scripts and dependencies
```

## Development Phases

### Phase 0 — Monorepo Scaffolding ✅
- [x] Repository initialization
- [x] Top-level project structure
- [x] Root configuration files

### Phase 1 — Mobile App Base (Expo)
**Lead: Member 1**

Core setup and dependencies:
```bash
cd app
npx create-expo-app@latest -t expo-template-blank-typescript
npm i zustand @react-native-async-storage/async-storage
npm i react-native-paper react-native-vector-icons
npm i expo-image-picker nativewind
```

App folder structure:
```
/app
  /assets
    directory.json     # User directory (exported from DB seed)
  /src
    /screens           # Welcome, Home, CreateRoom, RoomDetail, AddBill, SettleUp
    /components        # RoomCard, PinnedSummary, MessageList, AddBillSheet
    /state             # Zustand stores for users, rooms, bills, payments, messages
    /lib               # Business logic utilities
    /types             # TypeScript domain types
```

### Phase 2 — Fake Auth + Persistence
**Lead: Member 1**

Simple user selection system with AsyncStorage persistence:
- Welcome screen with user dropdown from `directory.json`
- Current user state management
- Profile editing capabilities
- Demo data reset functionality

### Phase 3 — Data Models + Utilities
**Lead: Member 3 (with team collaboration)**

Core TypeScript types and business logic:

```typescript
export type Currency = 'CAD';
export type Category = 'rent'|'groceries'|'order'|'subscription'|'utilities'|'other';

export type User = {
  id: string;
  name: string;
  email: string;
  avatarUrl?: string;
};

export type Room = {
  id: string;
  title: string;
  category: Category;
  dueDate?: string;
  recurrence?: 'monthly'|'none';
  memberIds: string[];
  createdAt: number;
  createdBy: string;
};

export type Bill = {
  id: string;
  roomId: string;
  title: string;
  amount: number;      // CAD cents
  currency: Currency;
  paidBy: string;      // user id
  date: string;        // ISO
  splitMode: 'equal'|'percent'|'shares'|'per-item';
};
```

Key utilities:
- `computeBalances(roomId)` — Calculate net balances per user
- `suggestTransfers(balances)` — Optimize settlement transfers
- Bankers' rounding to 2 decimal places

### Phase 4 — Screens & User Flows
**Lead: Member 1 + Member 2**

Complete UI implementation:

#### A. Welcome (Fake Auth)
- User selection from directory
- Session persistence

#### B. Home (Rooms List)
- Room cards with category and net balance
- "New Room" floating action button

#### C. Create Room
- Category selection and room details
- Member invitation by email lookup
- Validation against user directory

#### D. Room Detail (Tabbed Interface)
**Summary Tab:**
- Pinned balance summary card
- "Who owes whom" settlement suggestions
- Action buttons: Add Bill, Settle Up
- "Coming soon" feature badges

**Chat Tab:**
- Local message history
- System messages for bill/payment events
- "Realtime — Coming soon" indicator

**Bills Tab:**
- Bill history and details
- Receipt upload with "Parsing: Coming soon"

#### E. Add Bill
- Multiple split mode support:
  - **Equal** — Automatic even distribution
  - **Percent** — Custom percentage per user (sum = 100%)
  - **Shares** — Integer shares per user
  - **Per-item** — Item-level assignment
- Real-time validation and preview

#### F. Settle Up
- Minimal transfer suggestions
- Payment confirmation with proof images
- Status tracking and system notifications

### Phase 5 — Local State Management
**Lead: Member 2 with Member 1**

Zustand store implementation:
- `/src/state/rooms.ts` — Room CRUD operations
- `/src/state/bills.ts` — Bill and split management
- `/src/state/payments.ts` — Payment tracking
- `/src/state/messages.ts` — Chat and system messages

AsyncStorage persistence with hydration on app start.

### Phase 6 — Git Workflow
**Team collaboration guidelines**

Branch naming convention:
- `feat/app-fake-auth`
- `feat/app-rooms-home`
- `feat/app-create-room`
- `feat/app-room-detail`
- `feat/app-add-bill`
- `feat/app-settle-up`
- `feat/app-receipt-upload-stub`
- `feat/app-reset-demo-data`

Small, focused PRs with demo videos/screenshots.

### Phase 7 — Database Setup (Private)
**Lead: Member 4**

AWS infrastructure:
- DynamoDB single table design (`SharedRooms`)
- CDK or SAM infrastructure as code
- Seed script with demo data
- Directory export to `/app/assets/directory.json`

**Entity Design:**
```
User:     pk=USER#<userId>,    sk=PROFILE
Room:     pk=ROOM#<roomId>,    sk=META
Member:   pk=ROOM#<roomId>,    sk=MEMBER#<userId>
Message:  pk=ROOM#<roomId>,    sk=MSG#<ts>#<id>
Bill:     pk=ROOM#<roomId>,    sk=BILL#<billId>
Split:    pk=ROOM#<roomId>,    sk=BILL#<billId>#SPLIT#<userId>
Payment:  pk=ROOM#<roomId>,    sk=PAY#<ts>#<id>
```

Private API stubs (not consumed by app initially).

### Phase 8 — Documentation
**Lead: Member 3**

Comprehensive documentation:
- `/docs/data-contracts.md` — Type definitions and JSON examples
- `/docs/settlement.md` — Balance calculation algorithms and examples
- `/docs/future.md` — Roadmap for authentication, OCR, notifications, and realtime features

## Team Task Breakdown

### Member 1 — Mobile Foundation + Auth + Navigation
**Branches:** `feat/app-fake-auth`, `feat/app-rooms-home`

**Deliverables:**
- Expo app scaffold with dependencies
- User authentication store with AsyncStorage
- Welcome screen with user selection
- Home screen with rooms list and balance display
- Room detail navigation with tabbed interface
- Basic chat messaging functionality

**Acceptance:** Cold start → user selection → rooms list with calculated nets → room detail with working chat

### Member 2 — Core Features + State Management
**Branches:** `feat/app-create-room`, `feat/app-add-bill`, `feat/app-settle-up`, `feat/app-receipt-upload-stub`

**Deliverables:**
- Complete state management (rooms, bills, payments)
- Create room with member validation
- Add bill with all split modes and validation
- Settlement interface with transfer suggestions
- Payment confirmation with proof upload
- Receipt upload stub with "Coming soon" messaging
- Demo data reset functionality

**Acceptance:** Create room → add bills with different split modes → settle up with payment confirmation → system messages appear

### Member 3 — Business Logic + Documentation
**Branches:** `feat/app-logic-balances`, `docs/contracts`

**Deliverables:**
- Balance calculation utilities with proper rounding
- Settlement transfer optimization algorithms
- Comprehensive unit test coverage
- PinnedSummary component with balance visualization
- Complete project documentation
- Algorithm specifications with examples

**Acceptance:** Unit tests pass for 2-person and 3+ person scenarios; Summary displays correct balances and suggestions

### Member 4 — Infrastructure + Data Export
**Branches:** `infra/dynamodb-seed`

**Deliverables:**
- AWS infrastructure setup (DynamoDB + CDK/SAM)
- Database seeding with realistic demo data
- User directory export to app assets
- Private API stubs for future integration
- Deployment and data management scripts

**Acceptance:** DynamoDB contains seed data; app can validate member emails using exported directory

## Demo Flow

1. **App Launch** → Select "Misbah Ahmed" from user dropdown
2. **Home Screen** → View "Family plan" room with calculated balance
3. **Room Detail** → Navigate through Summary, Chat, and Bills tabs
4. **Add Bills:**
   - $20.00 CAD paid by Misbah (equal split)
   - $14.00 CAD paid by Anthony (equal split)
5. **Settlement** → View suggested transfer: "Anthony → Misbah: $3.00 CAD"
6. **Payment** → Confirm payment with proof image upload
7. **Feature Preview** → Point out "Coming soon" badges for future functionality

## Technical Constraints

- **Currency:** CAD only
- **Rounding:** Bankers' rounding to 2 decimal places
- **Platform:** React Native with Expo
- **State:** Local-first with AsyncStorage persistence
- **Auth:** Demo users only (real auth coming soon)
- **API:** Private infrastructure, not consumed by app initially

## Getting Started

1. Clone the repository
2. Navigate to `/app` directory
3. Install dependencies: `npm install`
4. Start development server: `npx expo start`
5. Use demo user credentials from welcome screen

---

*Built for Hack the North 2025 🍁*