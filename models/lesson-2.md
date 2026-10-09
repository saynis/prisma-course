# Lesson 4 — Prisma Migrations
 
## Learning Objectives
 
After this lesson you will be able to:
 
- Explain what a migration is and why it is useful
- Create your first migration with `prisma migrate dev`
- Read the SQL that Prisma generated for you
- Change your schema and create a second migration
- Look at the created table in `psql` and Prisma Studio


## Why Are We Learning This?
 
Real projects change. You add a column, rename a field, create a new table. Teammates and servers must receive those changes in the same order.
 
If everyone edits their database by hand, things break. **Migrations** solve this: every database change is saved as a file in the project.
 
## Concept Explanation
 
**Migration** — a saved set of SQL commands that moves your database from one version to the next.
 
Think of `ALTER TABLE` and `CREATE TABLE` from your PostgreSQL course. A migration is those commands written into a file and tracked in your project.
 
**How Prisma does it:**
 
1. You change `schema.prisma`.
2. You run `prisma migrate dev`.
3. Prisma compares the schema to the database, writes the needed SQL into a new migration folder, and runs it.
4. Prisma remembers which migrations ran in a table called `_prisma_migrations`.
`prisma migrate dev` is for **development**. (In production we apply saved migrations with a different command; that is a later topic.)
 
**Generate** — after a schema change we also run `prisma generate` to update Prisma Client so it knows about the new tables and fields.
 
## Real-World Analogy
 
A migration is a **building renovation log**.
 
Each entry says exactly what changed ("added a window", "moved a wall") and when. Anyone can follow the log from the first entry to rebuild the same building, in the same order.
 
## Visual Explanation
 
```
 Prisma Schema (schema.prisma)
           ↓   npx prisma migrate dev
      Migration (SQL file in prisma/migrations/)
           ↓   runs the SQL
    PostgreSQL Database (tables updated)
           ↓   npx prisma generate
      Prisma Client updated
```
 
Migration folder after the first migration:
 
```
prisma/
├── schema.prisma
└── migrations/
    ├── migration_lock.toml
    └── 20261004120000_init/
        └── migration.sql
```
 
## SQL vs Prisma
 
| In PostgreSQL | In Prisma |
|---|---|
| You type `CREATE TABLE ...` in `psql` | You write a model, then run a migration |
| You type `ALTER TABLE ... ADD COLUMN ...` | You add a field, then run another migration |
| You remember what you changed | Migration files keep the full history |
 
The migration file **is** SQL. Open it and you will recognise it:
 
```sql
-- CreateTable
CREATE TABLE "students" (
    "id" SERIAL NOT NULL,
    "name" TEXT NOT NULL,
    "email" TEXT NOT NULL,
    "age" INTEGER NOT NULL,
    "is_active" BOOLEAN NOT NULL DEFAULT true,
    "created_at" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
 
    CONSTRAINT "students_pkey" PRIMARY KEY ("id")
);
 
-- CreateIndex
CREATE UNIQUE INDEX "students_email_key" ON "students"("email");
```
 
`@unique` became a unique index. `@id` became a primary key. Nothing magical, it is the SQL you already know.
 
## Syntax
 
```bash
npx prisma migrate dev --name <short_description>
npx prisma generate
```
 
## Example
 
**Part A — First migration**
 
Make sure `schema.prisma` has the `Student` model from Lesson 3, then:
 
```bash
npx prisma migrate dev --name init
npx prisma generate
```

 
**Part B — Change the schema**
 
Add an optional `phone` field to `Student`:
 
```prisma
model Student {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int
  phone     String?
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
 
  @@map("students")
}
```
 
Run:
 
```bash
npx prisma migrate dev --name add_student_phone
npx prisma generate
```
 
The new migration SQL is:
 
```sql
-- AlterTable
ALTER TABLE "students" ADD COLUMN "phone" TEXT;
```
 
**Part C — Look at the data visually**
 
```bash
npx prisma studio
```
 
Prisma Studio opens in your browser and shows the tables.
 
## Explain the Example
 
- `--name init` is just a label for the migration folder. Use clear names like `add_student_phone`.
- Prisma created the folder `prisma/migrations/<date>_init/` with `migration.sql` inside, and ran that SQL on `school_db`.
- In `psql`, `\dt` lists tables. You will see `students` and also `_prisma_migrations` (Prisma's own history table).
- In Part B we added an **optional** field. Existing rows can safely get `NULL` in the new column. This is why optional fields are the easiest first changes.
- Prisma saw only the *difference* between the schema and the database, so it wrote only `ALTER TABLE ... ADD COLUMN`.
- `prisma generate` rebuilds Prisma Client so your code can use `phone`.
- **Never edit an old migration file** after it has been applied. Make a new migration instead.


## Common Beginner Mistakes
 
1. **Forgetting to run `migrate dev` after changing the schema.** The schema alone does not touch the database.
2. **Forgetting `prisma generate`.** The database is updated but Prisma Client still uses the old shape. Run `generate` after schema changes.
3. **Connection errors.** `migrate dev` fails if `DATABASE_URL` is wrong or PostgreSQL is not running. Test the same details in `psql`.
4. **Permission error about a "shadow database".** `migrate dev` temporarily creates an extra database to check your migrations, so your PostgreSQL user needs permission to create databases. Using the `postgres` user locally works.
5. **Adding a required field to a table that already has rows.** PostgreSQL cannot add a `NOT NULL` column without a value for existing rows. Make the field optional (`?`) or give it a `@default(...)`.
6. **Deleting the `migrations` folder.** It is the project's history. Keep it and commit it to Git.
7. **Looking for a table named `Student`.** Because of `@@map("students")`, the table is `students`.



## Assignment
 
Create the **teachers** table and then improve it.
 
### Assignment Requirements
 
1. Make sure your `Teacher` model from Lesson 3 is in the schema.
2. Run a migration named `add_teachers_table` and run `prisma generate`.
3. In `psql`, show the table structure with `\d teachers` and save a screenshot or copy of the output.
4. Add an optional field `phone String?` to `Teacher` and create a second migration named `add_teacher_phone`.
5. Open both `migration.sql` files and write, in your own words, what each SQL statement does.
6. Open Prisma Studio and confirm the `teachers` table is there.


### Expected Result
 
Two new migration folders, a `teachers` table with a `phone` column visible in `psql` and Prisma Studio, and a short written explanation of the generated SQL.
 
## Concepts Used
 
Lessons 1–3: ORM, Prisma, PostgreSQL, SQL, setup, schema, models, fields, `@id`, `@default`, `@unique`, `@map`, `@@map`

Lesson 4: migration, `prisma migrate dev`, `prisma generate`, migration history, `_prisma_migrations`, Prisma Studio, `ALTER TABLE` via migrations
 
---