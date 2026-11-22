import sqlite3


class Database:
    """A tiny database wrapper around SQLite."""

    def __init__(self, path=":memory:"):
        self.conn = sqlite3.connect(path)
        self.conn.row_factory = sqlite3.Row

    def create_table(self, name, columns):
        cols = ", ".join(f"{k} {v}" for k, v in columns.items())
        self.conn.execute(f"CREATE TABLE IF NOT EXISTS {name} (id INTEGER PRIMARY KEY, {cols})")

    def insert(self, table, **data):
        keys = ", ".join(data)
        marks = ", ".join("?" for _ in data)
        cur = self.conn.execute(f"INSERT INTO {table} ({keys}) VALUES ({marks})", tuple(data.values()))
        self.conn.commit()
        return cur.lastrowid

    def get(self, table, **where):
        sql = f"SELECT * FROM {table}"
        if where:
            sql += " WHERE " + " AND ".join(f"{k} = ?" for k in where)
        return [dict(r) for r in self.conn.execute(sql, tuple(where.values()))]

    def update(self, table, row_id, **data):
        sets = ", ".join(f"{k} = ?" for k in data)
        self.conn.execute(f"UPDATE {table} SET {sets} WHERE id = ?", (*data.values(), row_id))
        self.conn.commit()

    def delete(self, table, row_id):
        self.conn.execute(f"DELETE FROM {table} WHERE id = ?", (row_id,))
        self.conn.commit()


if __name__ == "__main__":
    db = Database()
    db.create_table("users", {"name": "TEXT", "age": "INTEGER"})

    uid = db.insert("users", name="Alexx", age=30)
    db.insert("users", name="Sam", age=25)

    print(db.get("users"))            # all rows
    print(db.get("users", name="Sam"))  # filtered

    db.update("users", uid, age=31)
    db.delete("users", 2)
    print(db.get("users"))
