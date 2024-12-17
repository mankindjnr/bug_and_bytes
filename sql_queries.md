## CREATE a database
```sql
CREATE DATABASE database_name;
```

## CREATE a table
```sql
CREATE TABLE user_data (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age DECIMAL NOT NULL
);

-- Create an index on user_id for faster queries
CREATE INDEX idx_user_id ON user_data (user_id);

```

## ALTER a Table
```sql
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
```

## DROP a table
```sql
DROP TABLE users
```

# Data Manipulation Language (DML)
Commands used to manipulate data in the database

## Insert Data
```sql
INSERT INTO user_data (name, email, age)
VALUES
    ('Alice Johnson', 'alice.johnson@example.com', 25),
    ('Bob Smith', 'bob.smith@example.com', 30),
    ('Catherine Brown', 'catherine.brown@example.com', 27),
```

`NOTE`
```sql
1. Enable the pgcrypto Extension
If the pgcrypto extension is not enabled, you need to enable it first. You can do this by running the following command as a database superuser:

sql
Copy code
CREATE EXTENSION IF NOT EXISTS pgcrypto;
2. Using gen_random_uuid()
Once the pgcrypto extension is enabled, the gen_random_uuid() function will be available, and you can use it directly in your table definition or queries.

For example:

sql
Copy code
INSERT INTO user_data (user_id, name, email, age)
VALUES (gen_random_uuid(), 'John Doe', 'john.doe@example.com', 30);
```
## Update Data
```sql
UPDATE users SET email = 'john.updated@example.com' WHERE id = 1;
```

## Delete Data
```sql
DELETE FROM users WHERE id = 1;
```

# Data Query Language (DQL)
Commands used to query data

## Select Data
```sql
SELECT * FROM users;
```

## Select with conditions
```sql
SELECT name, email FROM users WHERE is_active = TRUE;
```

## Aggregate functions
```sql
SELECT COUNT(*) AS total_users FROM users;
```

# Data Control Language (DCL)
Commands to control access to the database

## Grant Privileges
```sql
GRANT SELECT, INSERT ON users TO user_name;
```

## Revoke Privileges
```sql
REVOKE SELECT ON users FROM user_name;
```

# INDEXES

## Create an Index
```sql
CREATE INDEX idx_users_email ON users(email);
```

## Drop an Index
```sql
DROP INDEX idx_users_email;
```