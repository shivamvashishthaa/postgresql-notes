# 📌 PART 1: PostgreSQL Basics, CLI & User Permissions

## 1. Introduction & Terminal Commands (psql)

PostgreSQL (often called Postgres) is a powerful, open-source object-relational database system. The `psql` is the interactive terminal program that allows you to interact with the database.

### 🖥️ Essential Terminal Commands

- `\l` or `\list`: Shows the list of all databases.
- `\c db_name`: Switches (connects) to a specific database.
- `\dt`: Shows the list of all tables in the current database.
- `\d table_name`: Shows the structure/schema of a specific table (columns, data types, constraints).
- `\du`: Shows all database users (roles) and their privileges.
- `\q`: Quits the psql terminal.
- `\?`: Shows help for all psql backslash commands.
- `\h`: Shows help for SQL commands (e.g., `\h CREATE TABLE`).

### 🔌 Connecting to Database via Terminal

- `psql -U postgres`: Connects to the database using the `postgres` user.
- `psql -U postgres -d db_name`: Opens psql directly connected to a specific database (`db_name`).
- `psql -h hostname -U username -d db_name`: Connects to a remote database.
  - _Note: If you get a "Peer authentication failed" error, you might need to use `sudo -u postgres psql` or check your `pg_hba.conf` file._

---

## 2. User Management

In PostgreSQL, users and groups are collectively known as **Roles**. A role with the `LOGIN` privilege is a User.

### 👤 Creating a User

```sql
CREATE USER username WITH PASSWORD 'your_password';
-- OR
CREATE ROLE username WITH LOGIN PASSWORD 'your_password';
```

### 🔑 Altering a User

```sql
-- Change Password:
ALTER USER username PASSWORD 'new_password';

-- Granting Special Roles:
ALTER USER username CREATEDB; -- Give permission to create databases
ALTER USER username SUPERUSER; -- Give superuser privileges (Use with caution)

-- Dropping a User
DROP USER username;
```

---

## 3. PostgreSQL Permissions (User Privileges)

Permissions in PostgreSQL are granted at various levels. To give a user access to data, you must grant permissions from the top down (Database -> Schema -> Table/Object).

### 🏢 Level 1: Database Level

This controls who can connect to the database and what they can create inside it.

| #   | Permission              | Meaning                       |
| --- | ------------------ | --------------------------- |
| 01  | CONNECT             | User can log in to the database. |
| 02  | CREATE                | User can create new schemas and objects within the database.|
| 03  | TEMP                | User can create temporary tables.                    |



```sql
-- Syntax
GRANT CONNECT, CREATE, TEMP ON DATABASE db_name TO user_name;
```


### 📂 Level 2: Schema Level

Schemas are like folders inside a database. By default, PostgreSQL uses the `public` schema.

| #   | Permission              | Meaning                       |
| --- | ------------------ | --------------------------- |
| 01  | USAGE             | User can access the schema (see the objects inside). |
| 02  | CREATE                | User can create objects (tables, views) within the schema.|

```sql
-- Syntax:
GRANT USAGE ON SCHEMA public TO user_name;
GRANT CREATE ON SCHEMA public TO user_name;
```

### 📊 Level 3: Object Level (Tables, Views, Sequences)

This controls what a user can do with specific tables or data inside the schema.

| #   | Permission              | Meaning                       |
| --- | ------------------ | --------------------------- |
| 01  | SELECT             | Read data from the table. |
| 02  | INSERT                | Add new data (rows) to the table.|
| 03  | UPDATE             | Modify existing data in the table. |
| 04  | DELETE             | Delete data from the table. |
| 05  | TRUNCATE             | Delete all data from the table quickly. |
| 06  | REFERENCES             | Create foreign keys referencing this table. |
| 07  | TRIGGER             | Create triggers on this table.|



```sql
-- Syntax (Granting on a specific table):
GRANT SELECT, INSERT, UPDATE, DELETE ON table_name TO user_name;
```

```sql
-- Syntax (Granting on all tables in a schema):
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO user_name;
```

### 🛡️ Level 4: Special Role Permissions

These are powerful system-level privileges.

 #   | Permission              | Meaning                       |
| --- | ------------------ | --------------------------- |
| 01  | SUPERUSER             | Can do anything, bypass all permission checks. |
| 02  | CREATEDB                | Can create new databases.|
| 03  | CREATEROLE             | Can create new users/roles.|
| 04  | LOGIN             | Can log in to the database.|
| 05  | REPLICATION             | Can initiate streaming replication (for backups). |

```sql
-- syntax:
ALTER USER user_name CREATEDB;
```

### 🔢 Level 5: Sequence Permissions

Sequences are used for auto-incrementing IDs. If a user needs to insert data into a table with a `SERIAL` column, they need permission on the sequence.

```sql
-- syntax:
GRANT USAGE, SELECT ON SEQUENCE seq_name TO user_name;
```
---

## 4.💡 Real-World Example (Putting it all together)

Let's say you have a new developer, `app_user`, who needs to read and write data in a specific database `mydb`.

```sql
-- 1. Create the user
CREATE USER app_user WITH PASSWORD 'secure_password';

-- 2. Allow them to connect to the database
GRANT CONNECT ON DATABASE mydb TO app_user;

-- 3. Allow them to access the public schema
GRANT USAGE ON SCHEMA public TO app_user;

-- 4. Allow them to read/write all existing tables in that schema
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;

-- 5. Allow them to use sequences (for auto-increment IDs)
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO app_user;
```

---

## 5. ➕ Bonus: Client Authentication (`pg_hba.conf`)

In PostgreSQL, the `pg_hba.conf` (Host-Based Authentication) file controls who can connect, from which IP address, and how they must authenticate (e.g., password, trust, peer).

- Location: Usually in the data directory (e.g., /`etc/postgresql/14/main/pg_hba.conf`).

- Common Methods:

    - `trust`: Anyone can connect without a password (Not recommended for production).

    - `md5 / scram-sha-256`: Requires a hashed password.

    - `peer`: Uses the OS username (default for local connections on Linux).

    - `reject`: Refuses connection.

Note: After editing `pg_hba.conf`, you must reload PostgreSQL (`sudo systemctl reload postgresql`) for changes to take effect.