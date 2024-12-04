# DBMS-Transaction-Project
 import psycopg2

# Connect to the PostgreSQL database
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="DBMS_Project",  # Replace with your database name
    user="postgres",         # Replace with your username
    password="asdfghjkl"     # Replace with your password
)

# Your transaction code here
# Set isolation level for SERIALIZABLE isolation
conn.set_isolation_level(3)
# Disable autocommit to manage transactions manually
conn.autocommit = False

try:
    cur = conn.cursor()

    # Transaction 1: Add product (P100, cd, 5) in Product
    cur.execute("""
        INSERT INTO product (prod_id, pname, price) 
        VALUES ('P100', 'cd', 5)
        ON CONFLICT (prod_id) DO NOTHING
    """)

    # Transaction 2: Add product (P1, Tape, 5) in Product (make sure 'P1' exists)
    cur.execute("""
        INSERT INTO product (prod_id, pname, price) 
        VALUES ('P1', 'Tape', 5)
        ON CONFLICT (prod_id) DO NOTHING
    """)

    # Transaction 3: Add depot (D2, New York, 1000) in Depot
    # Ensure the depot 'D2' exists before inserting into stock
    cur.execute("""
        INSERT INTO depot (dept_id, addr, volume) 
        VALUES ('D2', 'New York', 1000)
        ON CONFLICT (dept_id) DO NOTHING
    """)

    # Transaction 4: Add (P100, D2, 50) in Stock (now D2 exists)
    cur.execute("""
        INSERT INTO stock (prod_id, dept_id, quantity) 
        VALUES ('P100', 'D2', 50)
        ON CONFLICT (prod_id, dept_id) DO NOTHING
    """)

    # Transaction 5: Add depot (D100, Chicago, 100) in Depot
    cur.execute("""
        INSERT INTO depot (dept_id, addr, volume) 
        VALUES ('D100', 'Chicago', 100)
        ON CONFLICT (dept_id) DO NOTHING
    """)

    # Transaction 6: Add (P1, D100, 100) in Stock (ensure P1 exists)
    cur.execute("""
        INSERT INTO stock (prod_id, dept_id, quantity) 
        VALUES ('P1', 'D100', 100)
        ON CONFLICT (prod_id, dept_id) DO NOTHING
    """)

    # Transaction 7: Delete dependent rows from stock table that reference product P1
    cur.execute("""
        DELETE FROM stock WHERE prod_id = 'P1'
    """)

    # Transaction 8: Change product P1 name to PP1 in Product table
    cur.execute("UPDATE product SET prod_id = 'PP1' WHERE prod_id = 'P1'")

    # Transaction 9: Change depot D1 name to DD1 in Depot table
    cur.execute("UPDATE depot SET dept_id = 'DD1' WHERE dept_id = 'D1'")

    # Transaction 10: Delete product PP1 from Product
    cur.execute("""
        DELETE FROM product 
        WHERE prod_id = 'PP1'
    """)

    # Transaction 11: Delete depot DD1 from Depot and Stock (cascade delete handles Stock)
    cur.execute("""
        DELETE FROM depot 
        WHERE dept_id = 'DD1'
    """)

except (Exception, psycopg2.DatabaseError) as error:
    print("Error occurred:", error)
    print("Rolling back the transaction...")
    conn.rollback()  # Rollback the transaction in case of error

else:
    # Commit the transaction if no errors occurred
    conn.commit()
    print("Transaction committed successfully!")

finally:
    # Close the cursor and the connection
    cur.close()
    conn.close()
    print("PostgreSQL connection is now closed.")
