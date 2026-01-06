# nextJsExploration


# Fullstack CRUD with Next.js 16.1.1

This guide walks you step‑by‑step through building a **fullstack CRUD application** using **Next.js 16.1.1**, **Prisma**, and **PostgreSQL**. By the end, you’ll master the App Router, Server Components, Server Actions, and modern fullstack patterns.

---

## 📚 Table of Contents
1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Project Setup](#project-setup)
4. [Database & Prisma](#database--prisma)
5. [Routing & Layouts](#routing--layouts)
6. [Validation](#validation)
7. [API Routes](#api-routes)
8. [Frontend Pages](#frontend-pages)
9. [Forms & Client Components](#forms--client-components)
10. [Server Actions (Advanced)](#server-actions-advanced)
11. [Testing & Debugging](#testing--debugging)
12. [Deployment](#deployment)
13. [Learning Path](#learning-path)
14. [Runbook](#runbook)

---

## Overview
- **Goal:** Build a CRUD app (Users) with create, read, update, delete.
- **Focus:** Next.js App Router, Prisma ORM, PostgreSQL, TypeScript, Zod validation.
- **Outcome:** Production‑ready app deployable on Vercel.

---

## Tech Stack

| Layer       | Choice         | Why?                                   |
|-------------|----------------|----------------------------------------|
| Framework   | Next.js 16.1.1 | Fullstack React with App Router         |
| ORM         | Prisma         | Type‑safe queries, migrations           |
| Database    | PostgreSQL     | Reliable SQL backend                    |
| Validation  | Zod            | Runtime validation for payloads         |
| UI          | React          | Server + Client components              |

---

## Project Setup

```bash
npx create-next-app@latest nextjs-crud-16
cd nextjs-crud-16
npm install prisma @prisma/client zod
Create .env:

env
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DBNAME?schema=public"
Database & Prisma
Initialize Prisma:

bash
npx prisma init
Edit prisma/schema.prisma:

prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
Run migration:

bash
npx prisma migrate dev --name init
Helper: lib/prisma.ts

ts
import { PrismaClient } from "@prisma/client";
const globalForPrisma = global as unknown as { prisma: PrismaClient };
export const prisma = globalForPrisma.prisma || new PrismaClient();
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
Routing & Layouts
Directory structure:

Code
app/
  layout.tsx
  page.tsx
  users/
    page.tsx
    new/page.tsx
    [id]/page.tsx
  api/
    users/route.ts
    users/[id]/route.ts
lib/
  prisma.ts
prisma/
  schema.prisma
schemas/
  user.ts
Validation
schemas/user.ts:

ts
import { z } from "zod";

export const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

export const updateUserSchema = z.object({
  id: z.number().positive(),
  name: z.string().min(1).max(100).optional(),
  email: z.string().email().optional(),
});
API Routes
app/api/users/route.ts:

ts
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { createUserSchema } from "@/schemas/user";

export async function GET() {
  const users = await prisma.user.findMany();
  return NextResponse.json(users);
}

export async function POST(req: Request) {
  const data = await req.json();
  const parsed = createUserSchema.safeParse(data);
  if (!parsed.success) return NextResponse.json({ error: parsed.error }, { status: 400 });
  const user = await prisma.user.create({ data: parsed.data });
  return NextResponse.json(user, { status: 201 });
}
app/api/users/[id]/route.ts:

ts
export async function PUT(req: Request, { params }: { params: { id: string } }) {
  const id = Number(params.id);
  const body = await req.json();
  const updated = await prisma.user.update({ where: { id }, data: body });
  return NextResponse.json(updated);
}

export async function DELETE(_: Request, { params }: { params: { id: string } }) {
  const id = Number(params.id);
  await prisma.user.delete({ where: { id } });
  return NextResponse.json({ ok: true });
}
Frontend Pages
app/users/page.tsx:

tsx
import { prisma } from "@/lib/prisma";

export default async function UsersPage() {
  const users = await prisma.user.findMany();
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name} - {u.email}</li>)}
    </ul>
  );
}
Forms & Client Components
app/users/new/page.tsx:

tsx
"use client";
import { useState } from "react";

export default function NewUserPage() {
  const [name, setName] = useState(""); const [email, setEmail] = useState("");
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    await fetch("/api/users", {
      method: "POST", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name, email }),
    });
  }
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <button type="submit">Create</button>
    </form>
  );
}
Server Actions (Advanced)
app/users/actions.ts:

tsx
"use server";
import { prisma } from "@/lib/prisma";

export async function createUserAction(formData: FormData) {
  const name = String(formData.get("name"));
  const email = String(formData.get("email"));
  return prisma.user.create({ data: { name, email } });
}
Testing & Debugging
Use Prisma Studio: npx prisma studio

Validate inputs with Zod

Log queries with Prisma’s log option

Deployment
Push repo to GitHub.

Deploy on Vercel.

Add DATABASE_URL in Vercel environment.

Run migrations against hosted DB.

Learning Path
Step 1: CRUD with route handlers

Step 2: Add Server Actions

Step 3: Pagination & search

Step 4: Relationships (Users → Posts)

Step 5: Authentication with NextAuth

Runbook
bash
npm run dev              # Start dev server
npx prisma migrate dev   # Apply migrations
npx prisma studio        # Inspect DB
References
Next.js  Docs (nextjs.org in Bing)

Prisma Docs

Zod Docs