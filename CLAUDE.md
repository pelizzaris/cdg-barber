# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FSW Barber is a barbershop booking platform built with Next.js 14 (App Router), TypeScript, Prisma, PostgreSQL, and NextAuth.js for Google authentication. Users can browse barbershops, view services, and make appointments.

## Key Commands

### Development
```bash
npm run dev              # Start development server (localhost:3000)
npm run build           # Build for production
npm run lint            # Run ESLint
npm run start           # Start production server
```

### Database (Prisma)
```bash
npx prisma migrate dev  # Run migrations
npx prisma db seed      # Seed database with initial data
npx prisma generate     # Generate Prisma Client
npx prisma studio       # Open Prisma Studio GUI
```

### Docker
```bash
docker-compose up -d    # Start PostgreSQL container
```

## Architecture & Patterns

### Folder Structure Convention

The `/app` directory uses prefixed folders to organize non-route code:
- `_actions/` - Server Actions for data mutations (create/delete operations)
- `_data/` - Data fetching functions used in Server Components
- `_components/` - Reusable React components (includes `/ui` for shadcn components)
- `_lib/` - Library configurations (Prisma, NextAuth)
- `_providers/` - React context providers
- `_constants/` - Application constants

**Important**: Folders prefixed with `_` do NOT create routes. They're organizational only.

### Server Actions vs Data Functions

**Server Actions** (`_actions/`):
- Handle mutations (create, update, delete)
- Called from Client Components
- Use `"use server"` directive
- Always call `revalidatePath()` after mutations
- Examples: `createBooking`, `deleteBooking`

**Data Functions** (`_data/`):
- Handle read operations
- Called from Server Components during rendering
- Return typed data from Prisma queries
- Examples: `getConfirmedBookings`, `getConcludedBookings`

### Authentication Flow

- NextAuth.js configured in `app/_lib/auth.ts`
- Google OAuth provider only
- Session management: Use `getServerSession(authOptions)` in Server Components/Actions
- Session includes custom `id` field added via callback
- API route: `app/api/auth/[...nextauth]/route.ts`

### Database Schema

**Core Models:**
- `User` - Authenticated users (via NextAuth)
- `Barbershop` - Barbershop details (name, address, phones, image)
- `BarbershopService` - Services offered by barbershops (name, price, description)
- `Booking` - Appointments linking User + BarbershopService + date/time

**Authentication Models:** `Account`, `Session`, `VerificationToken` (NextAuth tables)

### Booking Flow Architecture

1. User selects barbershop and service → Calendar UI appears
2. User picks date/time → Client Component calls `createBooking` Server Action
3. Server Action:
   - Verifies authentication with `getServerSession()`
   - Creates Booking record in database
   - Calls `revalidatePath()` to update UI on bookings and barbershop pages
4. Success confirmation displayed

### Search & Filtering

The barbershops page (`/barbershops`) accepts URL parameters:
- `?title=name` - Search barbershops by name
- `?service=serviceName` - Filter by service type

Home page includes:
- Search input component redirecting to `/barbershops?title=...`
- Quick search buttons redirecting to `/barbershops?service=...`

## Development Guidelines

### Git Hooks (Husky + lint-staged)
Pre-commit hook automatically runs:
- ESLint with auto-fix on staged `.ts` and `.tsx` files
- Prettier formatting on staged files

Configuration: `.lintstagedrc.json`

### Environment Variables Required

```
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE"
GOOGLE_CLIENT_ID="your_google_oauth_client_id"
GOOGLE_CLIENT_SECRET="your_google_oauth_client_secret"
NEXT_AUTH_SECRET="any_random_secret_string"
```

### Prisma Decimal Handling

`BarbershopService.price` is `Decimal` type. When using in components:
- Convert to number: `price.toNumber()`
- Prisma returns `Decimal` objects, not native numbers

### Component Patterns

**Client Components** (`"use client"`):
- Interactive UI (forms, buttons with onClick)
- Use hooks (useState, useEffect)
- Call Server Actions for mutations
- Examples: `ServiceItem`, `BookingItem`, `Search`

**Server Components** (default):
- Fetch data directly from database
- No interactivity or hooks
- Better performance, less JS to client
- Examples: `app/bookings/page.tsx`, `app/barbershops/[id]/page.tsx`

### Styling
- Tailwind CSS with custom config in `tailwind.config.ts`
- shadcn/ui components in `app/_components/ui/`
- dark mode support via `next-themes`
