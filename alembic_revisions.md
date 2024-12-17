# updates with alembic

## Renaming a Column Without Losing Any Data

To rename a column in SQLAlchemy using Alembic, you need to use the op.alter_column function with the new_column_name parameter.

Example
Suppose we want to rename the column old_column_name to new_column_name in the example_table.

First, create a new migration file:

```bash
alembic revision -m "Rename column old_column_name to new_column_name"
```

Then, edit the migration file (in the versions directory) as follows:

```python
from alembic import op
import sqlalchemy as sa

# revision identifiers, used by Alembic.

revision = 'xxxxxx'
down_revision = 'yyyyyy'
branch_labels = None
depends_on = None

def upgrade():
op.alter_column('example_table', 'old_column_name', new_column_name='new_column_name')

def downgrade():
op.alter_column('example_table', 'new_column_name', new_column_name='old_column_name')
```

Explanation:

op.alter_column('example_table', 'old_column_name', new_column_name='new_column_name') renames the column.
downgrade() does the reverse operation to rename the column back in case of a downgrade. 2. Renaming a Table Without Deleting or Losing Any Data

## To rename a table, use the op.rename_table function.

Example
Suppose we want to rename old_table_name to new_table_name.

First, create a new migration file:

```bash
alembic revision -m "Rename table old_table_name to new_table_name"
```

Then, edit the migration file (in the versions directory) as follows:

```python
from alembic import op
import sqlalchemy as sa

# revision identifiers, used by Alembic.

revision = 'xxxxxx'
down_revision = 'yyyyyy'
branch_labels = None
depends_on = None

def upgrade():
op.rename_table('old_table_name', 'new_table_name')

def downgrade():
op.rename_table('new_table_name', 'old_table_name')
```

Explanation:

op.rename_table('old_table_name', 'new_table_name') renames the table.
downgrade() does the reverse operation to rename the table back in case of a downgrade. 3. Adding a Non-Nullable Column to a Table with Existing Data

## To add a non-nullable column to a table that already has data, you need to:

Add the column with a default value.
Alter the column to be non-nullable.
Example
Suppose we want to add a non-nullable column new_column of type String to the example_table.

First, create a new migration file:

```bash
alembic revision -m "Add non-nullable column new_column to example_table"
```

Then, edit the migration file (in the versions directory) as follows:

```python
from alembic import op
import sqlalchemy as sa

# revision identifiers, used by Alembic.
revision = 'xxxxxx'
down_revision = 'yyyyyy'
branch_labels = None
depends_on = None

def upgrade():
    # Step 1: Add the new column with a default value.
    op.add_column('example_table', sa.Column('new_column', sa.String(), nullable=True, server_default='default_value'))

    # Step 2: Remove the server default and set the column as non-nullable.
    op.alter_column('example_table', 'new_column', nullable=False, server_default=None)

def downgrade():
    # Downgrade by removing the column
    op.drop_column('example_table', 'new_column')
```

Explanation:

- op.add_column('example_table', sa.Column('new_column', sa.String(), nullable=True, server_default='default_value')) adds the new column with a default value, ensuring no existing rows violate the non-null constraint.

- op.alter_column('example_table', 'new_column', nullable=False, server_default=None) makes the column non-nullable and removes the default value.
  downgrade() removes the column in case of a rollback.

These steps ensure the data integrity and smooth transition of schema changes.
