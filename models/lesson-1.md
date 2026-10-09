# Lesson 3 — Prisma Schema & Models
 
## Learning Objectives
 
After this lesson you will be able to:
 
- Explain what `schema.prisma` is and what its three main blocks do
- Write a Prisma model that describes a PostgreSQL table
- Choose field types and use `@id`, `@default`, `@unique`
- Explain the difference between required and optional fields
- Rename tables and columns with `@@map` and `@map`


## Why Are We Learning This?
 
The schema is the **single source of truth** for your database design in a Prisma project. Prisma reads it to create tables, to generate Prisma Client, and to give you auto-complete in your editor. Good backend work starts with a good schema.
 
## Concept Explanation
 
**Prisma schema** — a file (`prisma/schema.prisma`) written in Prisma's own simple language. It describes your data.
 
It has three kinds of blocks:
 
| Block | Purpose |
|---|---|
| `datasource` | Which database we use (PostgreSQL) |
| `generator` | What Prisma should generate (Prisma Client) and where |
| `model` | One table in the database |
 
**Model** — describes one table. A model named `Student` represents the students table.
 
**Field** — one line inside a model. It represents one column. A field has a **name**, a **type**, and optional **attributes** (the parts that start with `@`).
 
**Required vs optional** — `String` means the column is `NOT NULL`. `String?` (with a question mark) means the column allows `NULL`.
 
## Real-World Analogy
 
A model is a **blank registration form**.
 
- The model name is the title ("Student Form").
- Each field is a line on the form ("Name", "Age", "Email").
- Attributes are rules printed on the form ("must be unique", "filled automatically").
The form is not the filled-in data. It only says what a valid record must look like.
 
## Visual Explanation
 
```
 Prisma Model                  PostgreSQL Table
 ─────────────                 ──────────────────
 model Student { ... }   →     students
   id    Int                     id    SERIAL PRIMARY KEY
   name  String                  name  TEXT NOT NULL
   age   Int                     age   INTEGER NOT NULL
```
 
Field types and their PostgreSQL types:
 
| Prisma type | PostgreSQL type (default) |
|---|---|
| `Int` | `INTEGER` |
| `String` | `TEXT` |
| `Boolean` | `BOOLEAN` |
| `DateTime` | `TIMESTAMP(3)` |
| `Float` | `DOUBLE PRECISION` |
| `Decimal` | `DECIMAL(65,30)` |
 
## SQL vs Prisma
 
**PostgreSQL:**
 
```sql
CREATE TABLE students (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  age INTEGER NOT NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```
 
**Prisma:**
 
```prisma
model Student {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
 
  @@map("students")
}
```
 
| SQL idea | Prisma way |
|---|---|
| `PRIMARY KEY` | `@id` |
| `SERIAL` (auto number) | `@default(autoincrement())` |
| `NOT NULL` | field without `?` |
| allows `NULL` | field with `?` |
| `UNIQUE` | `@unique` |
| `DEFAULT value` | `@default(value)` |
| `NOW()` | `@default(now())` |
| table name `students` | `@@map("students")` |
| column name `is_active` | `@map("is_active")` |
 
Prisma can also express `CHECK` rules only in limited ways, so in this course we keep validation rules in our application code or in SQL.
 
## Syntax
 
```prisma
datasource db {
  provider = "postgresql"
}
 
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}
 
model ModelName {
  fieldName  Type   @attribute
}
```
 
Naming convention: models are **singular and start with a capital letter** (`Student`), fields use **camelCase** (`createdAt`).
 
## Example
 
Add this to `prisma/schema.prisma`:
 
```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}
 
datasource db {
  provider = "postgresql"
}
 
model Student {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
 
  @@map("students")
}
```
 
Check your schema for mistakes:
 
```bash
npx prisma format
npx prisma validate
```
 
## Explain the Example
 
- **`generator client`** — tells Prisma to generate Prisma Client using the `prisma-client` generator and put the code in `generated/prisma`.
- **`datasource db`** — says we use PostgreSQL. The connection string itself comes from `.env` through the config file.
- **`model Student`** — one model = one table.
- **`id Int @id @default(autoincrement())`** — integer primary key that increases automatically (`SERIAL`).
- **`name String`** — required text, like `TEXT NOT NULL`.
- **`email String @unique`** — no two students can share an email.
- **`age Int`** — required whole number.
- **`isActive Boolean @default(true)`** — if we do not give a value, it becomes `true`.
- **`createdAt DateTime @default(now())`** — filled automatically with the current time.
- **`@map("is_active")`** — in JavaScript we write `isActive`, but the real PostgreSQL column is `is_active`.
- **`@@map("students")`** — the real table is called `students`, so your `psql` queries look the same as before. Without it, Prisma would name the table `Student`.
- **`prisma format`** tidies the file. **`prisma validate`** checks that the schema is correct.


## Common Beginner Mistakes
 
1. **Forgetting `@id`.** Every model needs a unique identifier. Add `@id` to one field.
2. **Writing SQL types.** Use `Int`, `String`, `Boolean`, not `INTEGER` or `TEXT`.
3. **Forgetting `?` for optional fields.** If a column should accept `NULL`, write `String?`.
4. **Lowercase model names.** Write `model Student`, not `model student`.
5. **Thinking the schema creates the table immediately.** Writing a model only *describes* a table. The table is created in Lesson 4 with a migration.
6. **Expecting the table to be called `students` automatically.** Add `@@map("students")` if you want that name.

 
## Assignment
 
Design the **Teacher** model for the school database.
 
### Assignment Requirements
 
1. Add a `Teacher` model with: `id`, `name`, `email` (unique), `subject` (optional text), `isActive` (default `true`), `createdAt` (default now).
2. Map the table to `teachers` and use snake_case column names for multi-word fields.
3. Run `prisma format` and `prisma validate` without errors.
4. Write the equivalent `CREATE TABLE teachers (...)` SQL on paper or in a text file.
5. Add comments (`//`) above each field explaining the matching SQL idea.
### Restrictions
 
- Do **not** run any migration yet.
- Do **not** add relations to other models.
- Use only Lessons 1–3.
### Expected Result
 
A valid schema with `Student` and `Teacher` models, and a text file that shows the Prisma model next to its SQL version.
 
## Concepts Used
 
Lessons 1–2: ORM, Prisma, PostgreSQL, SQL, project setup
Lesson 3: schema, datasource, generator, model, field, field types, `@id`, `@default`, `@unique`, optional fields, `@map`, `@@map`, `prisma format`, `prisma validate`
 
---