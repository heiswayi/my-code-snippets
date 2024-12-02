This is a utility script to work with PostgreSQL database.

```python
import psycopg2
from psycopg2 import sql
from psycopg2.extras import RealDictCursor
import os

# Database configuration
DB_CONFIG = {
    "dbname": os.getenv("DB_NAME"),
    "user": os.getenv("DB_USER"),
    "password": os.getenv("DB_PASSWORD"),
    "host": os.getenv("DB_HOST"),
    "port": os.getenv("DB_PORT", 5432),  # Default PostgreSQL port is 5432
}


class DatabaseManager:
    def __init__(self):
        self.connection = None

    def connect(self):
        """Establish a connection to the PostgreSQL database."""
        try:
            self.connection = psycopg2.connect(**DB_CONFIG)
            self.connection.autocommit = True
            print("Database connection established.")
        except Exception as e:
            print(f"Error connecting to the database: {e}")
            raise

    def close(self):
        """Close the database connection."""
        if self.connection:
            self.connection.close()
            print("Database connection closed.")

    def execute_query(self, query, params=None):
        """
        Execute a SQL query.
        :param query: The SQL query to execute.
        :param params: Optional parameters for the query.
        :return: Query result or None for non-select queries.
        """
        try:
            with self.connection.cursor(cursor_factory=RealDictCursor) as cursor:
                cursor.execute(query, params)
                if query.strip().lower().startswith("select"):
                    return cursor.fetchall()
                return None
        except Exception as e:
            print(f"Error executing query: {e}")
            raise

    def insert(self, table, data):
        """
        Insert data into a table.
        :param table: The name of the table.
        :param data: A dictionary of column-value pairs to insert.
        """
        columns = data.keys()
        values = data.values()

        insert_query = sql.SQL(
            "INSERT INTO {table} ({fields}) VALUES ({values})"
        ).format(
            table=sql.Identifier(table),
            fields=sql.SQL(", ").join(map(sql.Identifier, columns)),
            values=sql.SQL(", ").join(sql.Placeholder() * len(columns)),
        )

        try:
            with self.connection.cursor() as cursor:
                cursor.execute(insert_query, list(values))
            print(f"Data inserted into table {table}: {data}")
        except Exception as e:
            print(f"Error inserting data into table {table}: {e}")
            raise

    def update(self, table, data, condition):
        """
        Update data in a table.
        :param table: The name of the table.
        :param data: A dictionary of column-value pairs to update.
        :param condition: A dictionary defining the condition for the update.
        """
        columns = data.keys()
        condition_column, condition_value = list(condition.items())[0]

        update_query = sql.SQL(
            "UPDATE {table} SET {fields} WHERE {condition_column} = {condition_value}"
        ).format(
            table=sql.Identifier(table),
            fields=sql.SQL(", ").join(
                sql.SQL("{} = {}").format(sql.Identifier(col), sql.Placeholder())
                for col in columns
            ),
            condition_column=sql.Identifier(condition_column),
            condition_value=sql.Placeholder(),
        )

        try:
            with self.connection.cursor() as cursor:
                cursor.execute(update_query, list(data.values()) + [condition_value])
            print(f"Data updated in table {table}: {data} where {condition}")
        except Exception as e:
            print(f"Error updating data in table {table}: {e}")
            raise

    def delete(self, table, condition):
        """
        Delete data from a table.
        :param table: The name of the table.
        :param condition: A dictionary defining the condition for the delete.
        """
        condition_column, condition_value = list(condition.items())[0]

        delete_query = sql.SQL(
            "DELETE FROM {table} WHERE {condition_column} = {condition_value}"
        ).format(
            table=sql.Identifier(table),
            condition_column=sql.Identifier(condition_column),
            condition_value=sql.Placeholder(),
        )

        try:
            with self.connection.cursor() as cursor:
                cursor.execute(delete_query, [condition_value])
            print(f"Data deleted from table {table} where {condition}")
        except Exception as e:
            print(f"Error deleting data from table {table}: {e}")
            raise


# if __name__ == "__main__":
#     # Example usage
#     db = DatabaseManager()
#     try:
#         db.connect()

#         # Example: Insert data
#         data_to_insert = {"id": 2, "name": "Jane Doe", "email": "jane.doe@example.com"}
#         db.insert("users", data_to_insert)

#         # Example: Update data
#         data_to_update = {"name": "Jane Smith", "email": "jane.smith@example.com"}
#         condition = {"id": 2}
#         db.update("users", data_to_update, condition)

#         # Example: Select query
#         result = db.execute_query("SELECT * FROM users WHERE id = %s", (2,))
#         print(result)

#         # Example: Delete data
#         condition = {"id": 2}
#         db.delete("users", condition)
#     finally:
#         db.close()
```
