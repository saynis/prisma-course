# Lesson 1 — What is Prisma ORM?
 
## Learning Objectives
 
After this lesson you will be able to:
 
- Explain what an ORM is in simple words
- Explain the difference between PostgreSQL, Prisma and Prisma Client
- Describe how a Node.js application reaches the database through Prisma
- Compare a SQL query with its Prisma Client version
## Why Are We Learning This?
 
A real backend application needs to read and write data. A user registers, a student is added, a grade is updated. These actions happen from code, not from a terminal.
 
We need a clean, safe way to send database operations from JavaScript/TypeScript. Prisma gives us that.
 
## Concept Explanation
 
**Database** — the place where data is stored. We use **PostgreSQL**.
 
**SQL** — the language PostgreSQL understands. You already know it.
 
**ORM (Object-Relational Mapper)** — a tool that connects *objects* in your code to *tables* in your database. Instead of writing SQL strings inside your program, you call functions, and the ORM turns them into SQL for you.
 
**Prisma ORM** — a popular ORM for Node.js and TypeScript. It has a few parts:
 
| Part | What it does |
|---|---|
| **Prisma Schema** | A file where you describe your tables (as *models*) |
| **Prisma Migrate** | Turns your schema into real tables in PostgreSQL |
| **Prisma Client** | Code your app calls to create, read, update and delete data |
| **Prisma Studio** | A visual window to look at your data in the browser |
 
**Important:** Prisma is **not** a database. The data still lives in PostgreSQL. Prisma is a *developer tool* that helps your code talk to it.
 
## Real-World Analogy
 
Imagine a big **warehouse** (PostgreSQL). Workers inside only understand warehouse forms (SQL).
 
You are a shop owner who writes requests in everyday language (JavaScript). Prisma is your **assistant**: you hand over a simple request, and the assistant fills in the correct warehouse form and brings back the goods.
 
The warehouse did not change. You just found an easier way to make requests.
 
## Visual Explanation
 
```
 Node.js Application (your code)
          ↓
     Prisma Client      ← the functions you call
          ↓
      Prisma ORM        ← turns calls into SQL
          ↓
 PostgreSQL Driver      ← sends SQL over the connection
          ↓
   PostgreSQL Database  ← stores the tables and rows
```
 
```
Two ways to reach the SAME data:
 
  psql  ──── SQL ─────────────────→ ┐
                                     ├→  PostgreSQL
  Node.js ── Prisma Client ────────→ ┘
```
 
## SQL vs Prisma
 
**SQL:**
 
```sql
SELECT * FROM students;
```
 
**Prisma:**
 
```ts
const students = await prisma.student.findMany();
```
 
| | SQL | Prisma |
|---|---|---|
| Who writes it | You, in `psql` | Your JavaScript/TypeScript app |
| Talks to | PostgreSQL directly | PostgreSQL, through Prisma Client |
| Result | Rows printed in the terminal | A JavaScript array of objects |
 
Both ask PostgreSQL for all rows in the students table. Prisma writes the SQL for you behind the scenes.
 
## Syntax
 
The basic shape of a Prisma query:
 
```ts
await prisma.<model>.<operation>({ ...options });
```
 
- `<model>` — which table (written in lowercase-first style, e.g. `student`)
- `<operation>` — what to do (`findMany`, `create`, `update`, `delete`)
- `{ ...options }` — details such as `where` or `data`
## Example
 
Here is how the same two tasks look in both worlds:
 
**Task 1 — get all students**
 
```sql
SELECT * FROM students;
```
```ts
await prisma.student.findMany();
```
 
**Task 2 — add a student**
 
```sql
INSERT INTO students (name, age) VALUES ('Ali', 20);
```
```ts
await prisma.student.create({
  data: { name: "Ali", age: 20 },
});
```
 
## Explain the Example
 
- `prisma.student` points to the students table.
- `findMany()` means "find many records". With no options it returns all of them, like `SELECT *`.
- `create({ data: {...} })` means "create one record". `data` holds the column values, like the `VALUES` part of `INSERT`.
- Both return JavaScript values we can use in our program.
You do not need to run this yet. We set everything up in Lesson 2.
 
## Common Beginner Mistakes
 
1. **Thinking Prisma replaces PostgreSQL.** Prisma needs PostgreSQL to store the data. Without the database, Prisma has nothing to talk to.
2. **Thinking Prisma replaces SQL knowledge.** Prisma writes SQL for us. When something goes wrong, understanding SQL helps us debug it.
3. **Mixing the names up.** *Prisma* is the tool. *Prisma Client* is the code you call. *PostgreSQL* is the database.
## Mini Practice
 
Match each item to its role: **PostgreSQL**, **SQL**, **Prisma Client**, **Node.js**
 
- (a) Where the backend application runs
- (b) The language the database understands
- (c) Where the data is stored
- (d) The code the app calls to work with the database
## Assignment
 
Create a short document (or notes) called **"My First Prisma Map"**.
 
### Assignment Requirements
 
1. Draw the architecture diagram: Node.js → Prisma Client → Prisma ORM → PostgreSQL Driver → PostgreSQL.
2. Write one sentence explaining each box in your own words.
3. Write the Prisma version of these SQL statements:
   - `SELECT * FROM students;`
   - `INSERT INTO students (name, age) VALUES ('Sara', 22);`
4. Answer: "Is Prisma a database? Why or why not?"
### Restrictions
 
- This lesson has no setup yet. Do **not** install or run anything.
- Use only the ideas from Lesson 1.
### Expected Result
 
A one-page document with the diagram, the explanations, the two Prisma translations and a correct answer to the question (No — Prisma is a tool; PostgreSQL stores the data).
 
## Concepts Used
 
Database, PostgreSQL, SQL, ORM, Prisma ORM, Prisma Client, `findMany`, `create`
 
---
