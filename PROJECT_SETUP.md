# Toucher: Development Environment Setup

This document provides the commands and descriptions required to initialize and configure the Toucher development environment from scratch.

## 1. Monorepo Initialization
**Command:** `npx create-turbo@latest . --package-manager pnpm`
**Description:** Initializes a Turborepo workspace. This creates the `apps/` and `packages/` structure.

## 2. Shared Infrastructure (The "Source of Truth")
To share code and UI tokens across the ecosystem, configure the following in the `packages/` directory:

### A. Shared Logic (`packages/shared`)
- **Purpose:** Store Zod schemas, TypeScript types, and business logic (e.g., scoring algorithms).
- **Setup:** Create a standard TypeScript package and export your schemas.
- **Usage:** Import in both Next.js and Expo apps to ensure identical validation rules.

### B. Shared UI Config (`packages/ui` or `packages/config`)
- **Purpose:** Share Tailwind CSS theme tokens (colors, spacing, fonts).
- **Setup:** Move your `tailwind.config.js` properties here.
- **Usage:** Reference this shared config in `apps/clubhouse` (Web) and `apps/green` (Mobile via NativeWind) to ensure visual parity.

## 3. Web Interface & shadcn/ui
**Commands:**
- Init Next.js: `npx create-next-app@latest apps/clubhouse --typescript --tailwind --eslint`
- Init shadcn: `cd apps/clubhouse && npx shadcn@latest init`
**Description:** Sets up the Clubhouse portal. Shadcn components will live locally in this app but should consume tokens from the shared UI package.

## 4. Mobile App (The Green)
**Command:** `npx create-expo-app@latest apps/green --template tabs`
**Styling:** Install **NativeWind** to enable Tailwind support on React Native.
**Description:** The mobile interface. It uses NativeWind to apply the same styling tokens defined in the shared package.

## 5. Content Management (Strapi)
**Command:** `npx create-strapi-app@latest apps/cms --quickstart`
**Description:** Installs Strapi headless CMS.

## 6. Database & ORM (Prisma)
**Command:** `pnpm add -D prisma && npx prisma init`
**Description:** Installs and initializes Prisma ORM.

## 7. Authentication (Clerk)
Clerk handles authentication across the ecosystem.

### A. Web Interface (Clubhouse)
1. **Install SDK:** `pnpm add @clerk/nextjs --filter clubhouse`
2. **Environment Variables (`apps/clubhouse/.env.local`):**
   ```env
   NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
   CLERK_SECRET_KEY=sk_test_...
   ```
3. **Wrap Layout:** In `apps/clubhouse/app/layout.tsx`, wrap `{children}` with `<ClerkProvider>`.
4. **Middleware:** Create `apps/clubhouse/middleware.ts`:
   ```typescript
   import { clerkMiddleware } from "@clerk/nextjs/server";
   export default clerkMiddleware();
   export const config = {
     matcher: ["/((?!.*\\..*|_next).*)", "/", "/(api|trpc)(.*)"],
   };
   ```

### B. Mobile App (Green)
1. **Install SDK:** `pnpm add @clerk/clerk-expo expo-secure-store --filter green`
2. **Environment Variables (`apps/green/.env`):**
   ```env
   EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
   ```
3. **Token Cache:** Create a secure token cache using `expo-secure-store` to persist sessions.
4. **Wrap Root Layout:** In `apps/green/app/_layout.tsx`, wrap the `Slot` or `Stack` with `<ClerkProvider publishableKey={...} tokenCache={tokenCache}>`.
5. **Usage:** Use `useAuth()` and `useUser()` hooks to manage session state.

## 8. Starting the Environment
**Command:** `pnpm install && pnpm run dev`
**Description:** Installs dependencies and starts all apps.
