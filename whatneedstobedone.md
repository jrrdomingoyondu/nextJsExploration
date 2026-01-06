Fullstack CRUD with Next.js 16.1.1 — Step-by-Step Mastery
This is your hands-on guide to mastering fullstack CRUD in Next.js 16.1.1 using the App Router, Prisma, and PostgreSQL. You’ll build a complete app end-to-end, learn modern patterns like Server Components and Server Actions, and ship confidently.

Overview and Goals
Outcome: Build a production-ready CRUD app (Users) with create, read, update, and delete.

Focus: App Router, Server Components, Server Actions, route handlers, validation, and error handling.

Assumptions: Comfortable with SQL and backend concepts. (If you’ve used Spring Boot: Controllers → Route Handlers, Repositories → Prisma Client, Services → Server Actions).

Why App Router: Enables server-first React with better data-fetching and streaming.

Tip: Keep your CRUD simple (Users) first. Add relations (e.g., Posts, Tasks) only after mastering single-entity flows.

Tech Stack and Architecture
Layer	Choice	Rationale	Key Files
Framework	Next.js 16.1.1	Fullstack React with App Router	app/
ORM	Prisma	Type-safe queries, migrations, DX	prisma/schema.prisma
DB	PostgreSQL	Reliable, familiar SQL	.env
UI	React	Server/Client components	app/*
Validation	Zod	Runtime validation for payloads	schemas/*

Export to Sheets

Project Setup and Configuration
1. Initialize the Project
Bash

npx create-next-app@latest nextjs-crud-16
# Choose TypeScript and ESLint for type safety.
2. Install Dependencies
Bash

npm install prisma @prisma/client zod
3. Environment Variables
Create a .env file:

Code snippet

DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DBNAME?schema=public"
Database and Prisma
1. Initialize Prisma
Bash

npx prisma init
2. Define Schema
Edit prisma/schema.prisma:

Code snippet

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
3. Migrate and Visualize
Bash

npx prisma migrate dev --name init
# Use 'npx prisma studio' to visualize your tables.
4. Prisma Client Helper
Create lib/prisma.ts to prevent multiple instances in development:

TypeScript

import { PrismaClient } from "@prisma/client";

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ["query", "info", "warn", "error"],
  });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
Routing and Layouts
Directory Structure
Plaintext

app/
  layout.tsx
  page.tsx
  users/
    page.tsx
    new/
      page.tsx
    [id]/
      page.tsx
  api/
    users/
      route.ts      // GET, POST
    users/[id]/
      route.ts      // PUT, DELETE
Shared Layout
app/layout.tsx:

TypeScript

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body style={{ maxWidth: 900, margin: "0 auto", padding: 24 }}>
        <nav style={{ display: "flex", gap: 12 }}>
          <a href="/">Home</a>
          <a href="/users">Users</a>
          <a href="/users/new">New User</a>
        </nav>
        <hr />
        {children}
      </body>
    </html>
  );
}
Validation and Schemas
Create schemas/user.ts:

TypeScript

import { z } from "zod";

export const createUserSchema = z.object({
  name: z.string().min(1, "Name is required").max(100),
  email: z.string().email(),
});

export const updateUserSchema = z.object({
  id: z.number().int().positive(),
  name: z.string().min(1).max(100).optional(),
  email: z.string().email().optional(),
});
API Routes (Route Handlers)
List and Create: app/api/users/route.ts
TypeScript

import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { createUserSchema } from "@/schemas/user";

export async function GET() {
  const users = await prisma.user.findMany({ orderBy: { id: "desc" } });
  return NextResponse.json(users);
}

export async function POST(req: Request) {
  try {
    const json = await req.json();
    const parsed = createUserSchema.safeParse(json);
    if (!parsed.success) {
      return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 });
    }
    const user = await prisma.user.create({ data: parsed.data });
    return NextResponse.json(user, { status: 201 });
  } catch (e) {
    return NextResponse.json({ error: "Unexpected error" }, { status: 500 });
  }
}
Server Components and Pages
List Users: app/users/page.tsx
TypeScript

import { prisma } from "@/lib/prisma";

export default async function UsersPage() {
  const users = await prisma.user.findMany({ orderBy: { id: "desc" } });

  return (
    <main>
      <h1>Users</h1>
      <ul>
        {users.map((u) => (
          <li key={u.id}>
            <a href={`/users/${u.id}`}>{u.name}</a> — {u.email}
          </li>
        ))}
      </ul>
    </main>
  );
}
Server Actions (Modern Alternative)
app/users/actions.ts:

TypeScript

"use server";
import { prisma } from "@/lib/prisma";
import { createUserSchema } from "@/schemas/user";
import { revalidatePath } from "next/cache";

export async function createUserAction(formData: FormData) {
  const payload = {
    name: String(formData.get("name") ?? ""),
    email: String(formData.get("email") ?? ""),
  };
  
  const parsed = createUserSchema.safeParse(payload);
  if (!parsed.success) throw new Error("Validation error");

  await prisma.user.create({ data: parsed.data });
  revalidatePath("/users");
}
Common Pitfalls
Mixing Server/Client: Keep components server-only when possible; move only interactive bits to the client.

Leaking Prisma Clients: Always use the singleton pattern in lib/prisma.ts.

Ignoring Errors: Always provide explicit status messages and 4xx/5xx codes in APIs.

Commands Runbook
Action	Command
Dev Server	npm run dev
Prisma Generate	npx prisma generate
DB Migrate	npx prisma migrate dev
Prisma Studio	npx prisma studio

