# Lesson 5 — Relationships & Foreign Keys
 
## Learning Objectives
 
After this lesson you will be able to:
 
- Explain how a PostgreSQL foreign key becomes a Prisma relation
- Write a one-to-many relation with `@relation`
- Explain the difference between the foreign key field and the relation field
- Create a migration for a relation
- Create records that are linked to each other
## Why Are We Learning This?
 
Real data is connected. A student belongs to a section. An order belongs to a user. A post belongs to an author.
 
You already learned relationships and foreign keys in PostgreSQL. Now we describe the same connections in Prisma so our application can use them.
 
## Concept Explanation
 
From your PostgreSQL course:
 
- **Foreign key** — a column that points to the primary key of another table.
- **One-to-many** — one section has many students; each student has one section.
In Prisma, a relation needs **two pieces on the "many" side** and **one piece on the "one" side**:
 
| Piece | Example | Becomes a real column? |
|---|---|---|
| **Foreign key field** (scalar) | `sectionId Int?` on `Student` | **Yes**, `section_id` |
| **Relation field** (on the "many" side) | `section Section?` on `Student` | No, it is only for Prisma |
| **Relation field** (on the "one" side) | `students Student[]` on `Section` | No, it is only for Prisma |
 
**Relation field** — a field whose type is another model. It does not create a column. It tells Prisma how two models are connected, so Prisma Client can query across them.
 
**`@relation(fields: [...], references: [...])`** — says which field is the foreign key (`fields`) and which field it points to in the other model (`references`).
 
**`Student[]`** — a list. It means "many students".
 
## Real-World Analogy
 
A **school hallway with classrooms**.
 
- Each **student** carries a card with a classroom number (the foreign key).
- Each **classroom door** has a list of enrolled students (the relation list).
The number on the card is stored with the student. The list on the door is not stored anywhere; it is simply found by checking which student cards have that number.
 
## Visual Explanation
 
```
 sections                      students
 ┌────┬───────────┐           ┌────┬───────┬────────────┐
 │ id │ name      │           │ id │ name  │ section_id │
 ├────┼───────────┤           ├────┼───────┼────────────┤
 │ 1  │ Section A │◄──────────│ 1  │ Ali   │ 1          │
 │ 2  │ Section B │◄───┐      │ 2  │ Sara  │ 1          │
 └────┴───────────┘    └──────│ 3  │ Omar  │ 2          │
                              └────┴───────┴────────────┘
   One section  ────────→  many students
```
 
```
 Student.section   → "which section does this student belong to?"  (one)
 Section.students  → "which students are in this section?"         (many)
```
 
## SQL vs Prisma
 
**PostgreSQL:**
 
```sql
CREATE TABLE sections (
  id SERIAL PRIMARY KEY,
  name TEXT UNIQUE NOT NULL
);
 
ALTER TABLE students ADD COLUMN section_id INTEGER;
 
ALTER TABLE students
  ADD CONSTRAINT students_section_id_fkey
  FOREIGN KEY (section_id) REFERENCES sections(id);
```
 
**Prisma:**
 
```prisma
model Section {
  id       Int       @id @default(autoincrement())
  name     String    @unique
  students Student[]
 
  @@map("sections")
}
 
model Student {
  // ...existing fields...
  sectionId Int?     @map("section_id")
  section   Section? @relation(fields: [sectionId], references: [id])
 
  @@map("students")
}
```
 
| SQL | Prisma |
|---|---|
| `section_id INTEGER` column | `sectionId Int?` |
| `FOREIGN KEY (section_id) REFERENCES sections(id)` | `@relation(fields: [sectionId], references: [id])` |
| Column may be `NULL` | `Int?` and `Section?` |
| "One section has many students" | `students Student[]` |
 
## Syntax
 
```prisma
model Parent {
  id       Int     @id @default(autoincrement())
  children Child[]                                   // the "one" side
}
 
model Child {
  id       Int     @id @default(autoincrement())
  parentId Int?                                      // foreign key field
  parent   Parent? @relation(fields: [parentId], references: [id])   // relation field
}
```
 
## Example
 
**Step 1 — Update the schema**
 
```prisma
model Section {
  id       Int       @id @default(autoincrement())
  name     String    @unique
  students Student[]
 
  @@map("sections")
}
 
model Student {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int
  phone     String?
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
  sectionId Int?     @map("section_id")
  section   Section? @relation(fields: [sectionId], references: [id])
 
  @@map("students")
}
```
 
**Step 2 — Migrate and generate**
 
```bash
npx prisma format
npx prisma migrate dev --name add_sections_and_relation
npx prisma generate
```
 
The migration SQL looks like this:
 
```sql
-- AlterTable
ALTER TABLE "students" ADD COLUMN "section_id" INTEGER;
 
-- CreateTable
CREATE TABLE "sections" (
    "id" SERIAL NOT NULL,
    "name" TEXT NOT NULL,
 
    CONSTRAINT "sections_pkey" PRIMARY KEY ("id")
);
 
-- CreateIndex
CREATE UNIQUE INDEX "sections_name_key" ON "sections"("name");
 
-- AddForeignKey
ALTER TABLE "students" ADD CONSTRAINT "students_section_id_fkey"
FOREIGN KEY ("section_id") REFERENCES "sections"("id")
ON DELETE SET NULL ON UPDATE CASCADE;
```
 
 
**Step 3 — Check with the SQL you already know**
 
```sql
SELECT s.name AS student, sec.name AS section
FROM students s
LEFT JOIN sections sec ON s.section_id = sec.id;
```
 
## Explain the Example
 
- **Why `Int?` and `Section?`?** A student may exist before being placed in a section, so the foreign key may be `NULL`. If every student *must* belong to a section, we would write `sectionId Int` and `section Section` (no question marks).
- **`students Student[]` on `Section`** creates no column in `sections`. It is Prisma's view of the relationship. Prisma needs it so you can query from the section side later.
- **`fields: [sectionId]`** is the foreign key in this model. **`references: [id]`** is the primary key in the other model.
- In the script, we connect records by saving the section's `id` into `sectionId`. This is the same as inserting a value in a foreign key column.
- `section` and `students` are *not* filled in when you read students yet. In Lesson 9 we use `include` to load them.
- Existing student rows are safe because `sectionId` is optional.
**Other relationship types** (a short look ahead, no practice yet):
 
- **One-to-one:** like one-to-many, but the foreign key is also `@unique`, and the other side is a single optional record instead of a list.
- **Many-to-many:** both models have a list field (for example `students Student[]` and `courses Course[]`). Prisma creates the join table for you. You will meet this when a project needs it.

## Common Beginner Mistakes
 
1. **Writing only one side of the relation.** `prisma validate` reports an error. Run `npx prisma format`; it adds the missing opposite field for you in most cases.
2. **Thinking `section` is a column.** The column is `section_id` (`sectionId`). The relation field `section` has no column.
3. **Forgetting the foreign key field.** `@relation(fields: [sectionId], ...)` needs `sectionId` to exist in the model.
4. **Mismatched types.** `sectionId` must have the same type as `Section.id` (`Int`).
5. **Making the foreign key optional but the relation required.** Keep both consistent: `Int?` with `Section?`, or `Int` with `Section`.
6. **Linking to an id that does not exist.** `sectionId: 99` fails with a foreign key error, like in PostgreSQL.
7. **Forgetting to migrate.** Changing the schema alone does not create the foreign key.



## Assignment
 
Add a **Teacher → Section** relationship: one teacher can teach many sections.
 
### Assignment Requirements
 
1. Update the schema: add `teacherId Int? @map("teacher_id")` and a `teacher Teacher?` relation field to `Section`. Add `sections Section[]` to `Teacher`.
2. Run `prisma format`, create a migration named `add_teacher_to_section`, and run `prisma generate`.
3. Create 2 teachers (if you do not already have them) and 3 sections. Assign sections to teachers using `teacherId`.
4. Assign at least 4 students to sections using `sectionId`.
5. Draw the relationship diagram for `Teacher`, `Section` and `Student`.


### Restrictions
 
- Do **not** use `include` (Lesson 9).
- Do **not** create many-to-many relations.
- Do **not** use nested writes (creating related records inside another `create`).


### Expected Result
 
A schema with three connected models, a migration that adds the foreign key, linked data visible in `psql`, and a diagram showing Teacher → Section → Student.

 
---