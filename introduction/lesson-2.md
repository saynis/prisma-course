# Lesson 2 — Installing and Setting Up Prisma
 
## Learning Objectives
 
After this lesson you will be able to:
 
- Create a Node.js + TypeScript project for Prisma
- Install the Prisma packages and explain what each one does
- Create a PostgreSQL database for the project
- Initialise Prisma and connect it to the database
- Explain the purpose of the important files and folders
## Why Are We Learning This?
 
Before we can use Prisma in a real application, the project needs the right packages and a working database connection. Every backend project starts with this setup, so it is worth understanding each step.
 
## Concept Explanation
 
Setup has four small jobs:
 
1. Create a project folder and install packages.
2. Create an empty PostgreSQL database.
3. Run `prisma init` so Prisma creates its starting files.
4. Put the database address into a `.env` file.
**Connection string** — a single line that tells Prisma where the database is:
 
```
postgresql://USER:PASSWORD@HOST:PORT/DATABASE
```
 
**Environment variable** — a value stored outside your code (in `.env`) so secrets such as passwords stay out of your source files.

 
## Real-World Analogy
 
Setting up Prisma is like moving into a new office.
 
- The **project folder** is the empty office.
- **Packages** are the furniture you bring in.
- The **database** is the filing room.
- The **connection string** is the address and key for that filing room.
## Visual Explanation
 
Final project structure:
 
```
school-prisma/
├── prisma/
│   └── schema.prisma        ← describes our tables
├── generated/
│   └── prisma/              ← Prisma Client code (created for us)
├── src/
│   ├── lib/
│   │   └── prisma.ts        ← creates the Prisma Client (Lesson 5)
├── .env                     ← database connection string (secret)

 
| File / folder | Responsibility |
|---|---|
| `prisma/schema.prisma` | Describes the database tables as models |
| `prisma/migrations/` | History of database changes (appears in Lesson 4) |
| `generated/prisma/` | The generated Prisma Client code |
| `.env` | Stores `DATABASE_URL` |
| `src/` | Our own application code |
 

 
## SQL vs Prisma
 
| What you did in PostgreSQL | What we do now |
|---|---|
| `CREATE DATABASE school_db;` in `psql` | Same! Prisma uses a database that already exists |
| Connected with `psql -U postgres -d school_db` | Connect with a **connection string** in `.env` |
 
Prisma does not create the PostgreSQL *database* for you here. You create the empty database with SQL, and Prisma creates the *tables* inside it (Lesson 4).
 
## Syntax
 
Key commands (all run in the terminal):
 
```bash
npm init -y
npm install prisma@7  --save-dev
npm install @prisma/client@7
npx prisma init --datasource-provider postgresql --output ../generated/prisma
```
 
## Example
 
**Step 1 — Create the project**
 
```bash
mkdir school-prisma
cd school-prisma
npm init -y
```
 
**Step 2 — Install packages**
 
```bash
npm install prisma@7
npm install @prisma/client@7 
```
 
Check the Prisma version:
 
```bash
npx prisma --version
```
 
It should show a `7.x.x` version.

 
**Step 3 — Create the database in `psql`**
 
```sql
CREATE DATABASE school_db;
```
 
**Step 5 — Initialise Prisma**
 
```bash
npx prisma init --datasource-provider postgresql --output ../generated/prisma
```
 
**Step 6 — Set the connection string in `.env`**
 
```
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/school_db?schema=public"
```
 
**Step 7 — Protect secrets.** Make sure `.gitignore` contains:
 
```
node_modules
.env
generated
```
 
## Explain the Example
 
- **Step 2:** `prisma` (dev dependency) is the command-line tool. `@prisma/client` is the Prisma Client library. 
- **Step 3:** Prisma 7's generated client uses modern ES modules, so we turn them on.
- **Step 4:** We create the empty database with SQL, exactly like in your PostgreSQL course.
- **Step 5:** `--datasource-provider postgresql` says our database is PostgreSQL. `--output ../generated/prisma` says where to put the generated client code.
- **Step 6:** Replace `YOUR_PASSWORD` with your own PostgreSQL password.
After `prisma init`, `schema.prisma` looks like this:
 
```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}
 
datasource db {
  provider = "postgresql"
}
```
 
We explain these two blocks in Lesson 3.
 
## Common Beginner Mistakes
 
1. **Forgetting to create the database.** Prisma cannot connect to `school_db` if it does not exist. Run `CREATE DATABASE school_db;` first.
2. **Wrong password or port in `DATABASE_URL`.** Test the same details in `psql` first. If special characters are in the password (like `@` or `#`), they must be URL-encoded.
3. **Installing Prisma without a version.** Without `@7`, you may get a newer major version that works differently from this course.
4. **Committing `.env` to Git.** It contains your password. Always add it to `.gitignore`.

 
1. Run `npx prisma --version`. Write down the Prisma version.
2. Open `schema.prisma`. Find the line that says which database provider you use.
## Assignment
 
Create a new project called **`library-prisma`** using the same steps.
 
### Assignment Requirements
 
1. Create the project folder and install the packages (with version 7).
2. Create a PostgreSQL database called `library_db` in `psql`.
3. Run `prisma init` with PostgreSQL and the output path `../generated/prisma`.
4. Set `DATABASE_URL` in `.env` to point to `library_db`.
5. Add `.env` and `generated` to `.gitignore`.
6. Draw the final folder structure and write one line next to each important file.
### Restrictions
 
- Do **not** write any models yet.
- Do **not** run migrations yet.
- Use only Lessons 1–2.
### Expected Result
 
A project folder with the Prisma files created, a working `DATABASE_URL`, an empty `library_db` database, and a written explanation of the folder structure.
 
## Concepts Used
 
Lesson 1: ORM, Prisma, PostgreSQL, SQL
Lesson 2: Connection string, environment variable, `.env`, driver adapter, `prisma init`, `schema.prisma`, `CREATE DATABASE`
 
---
