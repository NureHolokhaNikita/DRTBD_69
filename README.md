# 🧪 Liquibase Lab — Database State Management

A hands-on lab project where you'll learn how to manage database schema changes using **Liquibase** — the same tool used by enterprise teams to version-control their databases.

> **A team of 4 students** collaborates through GitHub to design and implement a database from scratch. Each student designs their own domain model (ER diagram) divided into microservices (e.g., User Service, Product Service, Order Service, etc.) — no pre-made database template is used. Students create tables, add indexes, and insert data — just like real developers collaborating on a production database.

---

## 📥 Getting Started — Download the Project

**Option A: Clone with Git (recommended)**
```bash
git clone https://github.com/YOUR-REPO/liquibase-lab.git
cd liquibase-lab
```

**Option B: Download as ZIP**
1. Go to the GitHub repo page
2. Click the green **Code** button → **Download ZIP**
3. Unzip and open in IntelliJ IDEA (see below)

---

## 🛠 Prerequisites — What You Need Before Starting

### 1. IntelliJ IDEA
Download and install [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) (Community Edition is free and sufficient). After downloading the project, open it in IntelliJ:
- **File → Open** → select the `liquibase-lab` folder
- IntelliJ will auto-detect `pom.xml` and import the Maven project
- Wait for indexing and dependency download to complete

### 2. JDK 17+ and Maven 3.8+
IntelliJ usually bundles Maven, but verify JDK is configured:
- **File → Project Structure → Project SDK** → must be JDK 17 or higher
- Check terminal:
```bash
java -version    # should show 17 or higher
mvn -version     # should show 3.8 or higher
```

### 3. MS SQL Server (local)
You need a running MS SQL Server instance on your machine:
- **Windows**: Install [SQL Server Express](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) (free)
- **Docker alternative**: `docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourPassword123!" -p 1433:1433 -d mcr.microsoft.com/mssql/server:2022-latest`

### 4. Configure your connection
Open **both** properties files and update `username` and `password` to match your local MS SQL credentials:

- `src/main/resources/liquibase-init-db.properties` — connects to `master` (for database creation)
- `src/main/resources/liquibase.properties` — connects to `liquibase_lab` (for table migrations)

### 5. Create the database via Liquibase
**Don't create the database manually!** This project uses Liquibase to provision the database — the same way enterprise teams do it:

```bash
mvn -Pinit-db liquibase:update
```

This connects to the `master` database and runs `db-init-changelog.xml`, which:
1. Creates the `liquibase_lab` database (if it doesn't exist)
2. Creates a `lab_user` login
3. Assigns proper database roles (`db_datareader`, `db_datawriter`, `db_ddladmin`)

### 6. Git + GitHub account
You'll be collaborating through GitHub:
```bash
git --version
```

---

## 🚀 Key Commands — Your Main Toolkit

| Command | What It Does |
|---------|-------------|
| `mvn -Pinit-db liquibase:update` | **First-time setup**: creates the database (runs against `master`) |
| `mvn liquibase:status` | Shows which changesets haven't been applied yet |
| `mvn liquibase:update` | Applies all pending changesets to `liquibase_lab` database |
| `mvn liquibase:validate` | Validates all changelogs without applying them |
| `mvn liquibase:rollback -Dliquibase.rollbackCount=1` | Undoes the last applied changeset |

**Try it now!** After creating the database with `mvn -Pinit-db liquibase:update`, run:
```bash
mvn liquibase:status
```
You should see that 1 changeset has not been applied (the `tables` table). Then:
```bash
mvn liquibase:update
```
The `tables` meta-table should now exist in your database. 🎉

> **IntelliJ Tip**: You can run Maven commands from the **Maven panel** (right sidebar) → **Plugins** → **liquibase** → double-click `liquibase:update`. For profiles, right-click the project → **Maven** → **Select Maven Profiles** → check `init-db`.

---

## 📁 Project Structure — Where Everything Lives

```
liquibase-lab/
├── pom.xml                                  ← Maven config with 2 profiles
├── src/main/resources/
│   ├── liquibase-init-db.properties         ← Connection to 'master' (DB creation)
│   ├── liquibase.properties                 ← Connection to 'liquibase_lab' (tables)
│   ├── clear-database.sql                   ← Wipe all objects (reset to empty DB)
│   ├── drop-database.sql                    ← Delete the entire database
│   └── db/changelogs/
│       ├── db-init-changelog.xml            ← Creates the database + user + roles
│       ├── master-changelog.xml             ← THE LIST — controls table migration order
│       ├── init/                            ← CREATE TABLE files go here
│       │   └── tables-init.xml              ← Template: creates 'tables' meta-table
│       ├── changes/                         ← ALTER TABLE, ADD COLUMN, etc.
│       └── data/                            ← INSERT seed data
```

### Two-Phase Architecture (like the enterprise project)

This project has **two distinct phases**, each using a different Maven profile:

| Phase | Profile | Connects To | Changelog | Purpose |
|-------|---------|------------|-----------|---------|
| 1. DB Init | `init-db` | `master` | `db-init-changelog.xml` | Create database, logins, roles |
| 2. Tables | default | `liquibase_lab` | `master-changelog.xml` | Create/modify tables, insert data |

This mirrors the enterprise pattern where DB provisioning and table management are separate concerns.

### The golden rule of `master-changelog.xml`

This file is the **single source of truth** for what runs and in what order. Every changelog file must be `<include>`-d here. The section order matters:

```
1. init/     ← Tables must exist first
2. changes/  ← Modifications to existing tables
3. data/     ← Data inserted after tables and columns are ready
```

---

## ✍️ How to Write a Changeset

A **changeset** is a single unit of change. It has a unique `id` + `author` pair. Here's the anatomy:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-latest.xsd">

    <!-- author must be the student's last name -->
    <changeSet id="001-create-students-table" author="ivanov">
        <createTable tableName="students">
            <column name="id" type="BIGINT" autoIncrement="true">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="first_name" type="VARCHAR(100)">
                <constraints nullable="false"/>
            </column>
            <column name="last_name" type="VARCHAR(100)">
                <constraints nullable="false"/>
            </column>
            <column name="email" type="VARCHAR(255)">
                <constraints unique="true"/>
            </column>
            <column name="created_at" type="DATETIME" defaultValueComputed="GETDATE()"/>
        </createTable>

        <!-- ============================================================== -->
        <!-- SYSTEM REGISTRATION — DO NOT MODIFY                            -->
        <!-- Copy this block as-is for every new table you create.          -->
        <!-- Replace 'students' with your table name and update description. -->
        <!-- ============================================================== -->
        <sql>
            INSERT INTO dbo.tables (db_, name, type, author, version, service, description, column_count, created_at, last_update_at)
            VALUES ('liquibase_lab', 'students', 'DATA_TABLE', 'ivanov', 1, 'user_service',
                    'Table for storing student records.',
                    (SELECT COUNT(*) FROM sys.columns WHERE object_id = OBJECT_ID('students')),
                    SYSUTCDATETIME(), NULL);
        </sql>

        <rollback>
            <sql>DELETE FROM dbo.tables WHERE name = 'students';</sql>
            <dropTable tableName="students"/>
        </rollback>
    </changeSet>

</databaseChangeLog>
```

### Common Liquibase Tags Reference

| Tag | Purpose | Example |
|-----|---------|---------|
| `<createTable>` | Create a new table | See above |
| `<addColumn>` | Add column to existing table | See schema changes example below |
| `<dropColumn>` | Remove a column | `<dropColumn tableName="students" columnName="phone"/>` |
| `<renameColumn>` | Rename a column | `<renameColumn tableName="students" oldColumnName="name" newColumnName="full_name" columnDataType="VARCHAR(100)"/>` |
| `<addForeignKeyConstraint>` | Add FK relationship | See examples below |
| `<createIndex>` | Add an index | `<createIndex tableName="students" indexName="idx_email"><column name="email"/></createIndex>` |
| `<addNotNullConstraint>` | Make column NOT NULL | `<addNotNullConstraint tableName="students" columnName="email" columnDataType="VARCHAR(255)"/>` |
| `<insert>` | Insert a row of data | See data section below |
| `<rollback>` | Define undo operation | Always include this! |

### Schema Changes Example (changes/ folder)

When modifying an existing table (adding columns, renaming, etc.), place the file in `changes/`:

```xml
<changeSet id="001-add-phone-to-students" author="petrov">
    <addColumn tableName="students">
        <column name="phone" type="VARCHAR(20)"/>
    </addColumn>

    <!-- ============================================================== -->
    <!-- SYSTEM REGISTRATION UPDATE — DO NOT MODIFY                     -->
    <!-- Copy this block as-is when modifying columns of a table.       -->
    <!-- Replace 'students' with the table you are changing.            -->
    <!-- ============================================================== -->
    <sql>
        UPDATE dbo.tables
        SET column_count = (SELECT COUNT(*) FROM sys.columns WHERE object_id = OBJECT_ID('students')),
            last_update_at = SYSUTCDATETIME()
        WHERE name = 'students';
    </sql>

    <rollback>
        <dropColumn tableName="students" columnName="phone"/>
        <sql>
            UPDATE dbo.tables
            SET column_count = (SELECT COUNT(*) FROM sys.columns WHERE object_id = OBJECT_ID('students')),
                last_update_at = SYSUTCDATETIME()
            WHERE name = 'students';
        </sql>
    </rollback>
</changeSet>
```

### Foreign Key Example

```xml
<addForeignKeyConstraint
        baseTableName="grades"
        baseColumnNames="student_id"
        constraintName="fk_grades_students"
        referencedTableName="students"
        referencedColumnNames="id"/>
```

### Insert Data Example

```xml
<changeSet id="001-insert-sample-students" author="ivanov">
    <insert tableName="students">
        <column name="first_name" value="John"/>
        <column name="last_name" value="Doe"/>
        <column name="email" value="john.doe@example.com"/>
    </insert>
    <insert tableName="students">
        <column name="first_name" value="Jane"/>
        <column name="last_name" value="Smith"/>
        <column name="email" value="jane.smith@example.com"/>
    </insert>

    <rollback>
        <delete tableName="students">
            <where>email IN ('john.doe@example.com', 'jane.smith@example.com')</where>
        </delete>
    </rollback>
</changeSet>
```

---

## 🔄 Lab Workflow — Step-by-Step

### Step 0: All students — Initial setup

1. Clone the repository
2. Open project in IntelliJ IDEA
3. Update `liquibase-init-db.properties` and `liquibase.properties` with your local MS SQL Server credentials
4. Run the initial migration to create the local database:
   ```bash
   mvn -Pinit-db liquibase:update
   ```
   This executes `db-init-changelog.xml` and provisions the `liquibase_lab` database under MS SQL Server control.
5. Apply the template migration: `mvn liquibase:update`
6. Verify: the `tables` meta-table should exist in your `liquibase_lab` database (check via SSMS)

### Round 1 (Task Type 1): Database Skeleton

Each of the 4 students creates the table structure for their assigned microservice.

1. Create a feature branch: `git checkout -b MCR-<service_name>-feature` (e.g., `MCR-user_service-feature`)
2. Write changelog files in the `init/` folder:
   - Create **2–3 tables** for your microservice (e.g., `init/users-init.xml`, `init/roles-init.xml`)
   - Create corresponding **indexes** (e.g., `init/users-indexes-init.xml`)
3. Add `<include>` lines to `master-changelog.xml` in the **INIT** section
4. Test the migration locally:
   ```bash
   mvn liquibase:update
   ```
5. Verify via SSMS: check that your tables and indexes exist in the `liquibase_lab` database
6. Commit, push, and create a **Pull Request** to merge your branch into `main`

### Round 2 (Task Type 2): Populating Data

After all students have merged their Round 1 branches, everyone populates data.

1. Pull the latest merged changes: `git pull origin main`
2. Apply the combined migrations: `mvn liquibase:update` — this creates all tables from all 4 students in your local DB
3. Create a data branch: `git checkout -b MCR-<service_name>-data` (e.g., `MCR-user_service-data`)
4. Add data files to the `data/` folder:
   - Insert records into your tables (e.g., `data/users-data-V001.xml`, `data/roles-data-V001.xml`)
5. Add `<include>` lines to `master-changelog.xml` in the **DATA** section
6. Test locally:
   ```bash
   mvn liquibase:update
   ```
7. Verify via SSMS: check that your data is inserted correctly
8. Commit, push, and create a **Pull Request** to merge into `main`

---

## 🔀 Handling Merge Conflicts on `master-changelog.xml`

When multiple students edit `master-changelog.xml`, you'll see a conflict like this:

```
<<<<<<< HEAD
    <include file="init/users-init.xml" relativeToChangelogFile="true"/>
=======
    <include file="init/products-init.xml" relativeToChangelogFile="true"/>
>>>>>>> MCR-product_service-feature
```

**How to resolve**: Keep **both** lines! The order matters — put them in a logical sequence:

```xml
    <include file="init/users-init.xml" relativeToChangelogFile="true"/>
    <include file="init/products-init.xml" relativeToChangelogFile="true"/>
```

Then: `git add master-changelog.xml && git commit`

---

## 🗂 Understanding DATABASECHANGELOG

When you run `liquibase:update`, Liquibase creates a special tracking table called `DATABASECHANGELOG`. Run this query to see it:

```sql
SELECT ID, AUTHOR, FILENAME, DATEEXECUTED, MD5SUM, DESCRIPTION
FROM DATABASECHANGELOG
ORDER BY ORDEREXECUTED;
```

| Column | What It Tells You |
|--------|-------------------|
| `ID` | The changeset id (e.g., "001-create-students-table") |
| `AUTHOR` | Who wrote it |
| `FILENAME` | Which changelog file it came from |
| `DATEEXECUTED` | When it was applied |
| `MD5SUM` | A hash of the changeset content |
| `ORDEREXECUTED` | The sequence number |

### ⚠️ The #1 Rule: NEVER edit an applied changeset!

Once a changeset is applied, its `MD5SUM` is recorded. If you modify the XML, the hash changes, and Liquibase will **throw an error**:
```
Validation Failed: 1 changesets check sum
```

**If you need to change something, write a NEW changeset** (e.g., add a column, rename something). That's the whole point — every change is tracked as history.

---

## 🔄 Rolling Back Changes

Rollback is how you undo changesets. It only works if you have `<rollback>` blocks in your changesets.

```bash
# Undo the last changeset
mvn liquibase:rollback -Dliquibase.rollbackCount=1

# Undo the last 3 changesets
mvn liquibase:rollback -Dliquibase.rollbackCount=3
```

After rollback, the changeset is removed from `DATABASECHANGELOG` and it becomes "pending" again.

---

## 🧹 Starting Fresh (Nuclear Options)

### Clear all objects but keep the database

Open `src/main/resources/clear-database.sql` in SSMS (or IntelliJ's Database tool) and execute it. This drops all tables (including `DATABASECHANGELOG`), views, procedures, and functions. Then re-run:
```bash
mvn liquibase:update
```
All changesets will be applied from scratch.

### Drop the entire database

Open `src/main/resources/drop-database.sql` in SSMS and execute it. Then recreate it via Liquibase:
```bash
mvn -Pinit-db liquibase:update
```

---

## 📋 Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Init files | `tablename-init.xml` | `students-init.xml` |
| Change files | `tablename-change-V001.xml` | `students-change-V001.xml` |
| Data files | `tablename-data-V001.xml` | `courses-data-V001.xml` |
| ChangeSet ID | `NNN-description` | `001-create-students-table` |
| Author | Student's last name | `ivanov` |
| Table names | lowercase, underscores | `student_grades` |
| Index names | `idx_table_column` | `idx_students_email` |
| FK names | `fk_child_parent` | `fk_grades_students` |

---

## ❓ Troubleshooting

| Problem | Solution |
|---------|----------|
| "Connection refused" | Is MS SQL Server running? Check port 1433 |
| "Login failed" | Check username/password in your `.properties` files |
| "Checksum validation failed" | You edited an applied changeset! See DATABASECHANGELOG section above |
| "Table already exists" | Use `<preConditions onFail="MARK_RAN">` or clear the database |
| "Unknown changelog file" | Did you add `<include>` to `master-changelog.xml`? |
| Maven not found | Install Maven and add to PATH, or use IntelliJ's bundled Maven |
| IntelliJ doesn't recognize Maven | Right-click `pom.xml` → **Add as Maven Project** |

---

Good luck! 🚀
