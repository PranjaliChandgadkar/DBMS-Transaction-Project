# DBMS-Transaction-Project
import psycopg2
from tabulate import tabulate
 
con = psycopg2.connect(
    host="localhost",
    database="DBMS_Project",
    user="postgres",
    password="asdfghjkl")
 
 
#For isolation: SERIALIZABLE
con.set_isolation_level(3)
#For atomicity
con.autocommit = False
 
try:
    cur = con.cursor()
 
    cur.execute("ALTER TABLE Stock ADD CONSTRAINT fk_depot FOREIGN KEY(dep_id) REFERENCES depot(dep_id) ON UPDATE CASCADE ON DELETE CASCADE");
    cur.execute("ALTER TABLE Stock ADD CONSTRAINT fk_product FOREIGN KEY(prod_id) REFERENCES product(prod_id) ON UPDATE CASCADE ON DELETE CASCADE");
    #5 We add a product (p100, cd, 5) in Product and (p100, d2, 50) in Stock.
    cur.execute("insert into product values ('p100','cd',5)")
    cur.execute("insert into stock values ('p100','d2',50)")
    #6 We add a depot (d100, Chicago, 100) in Depot and (p1, d100, 100) in Stock.
    cur.execute("insert into depot values ('d100','Chicago',100)")
    cur.execute("insert into stock values ('p1','d100',100)")
 
    # #3 The product p1 changes its name to pp1 in Product and Stock.
    cur.execute("update product set prod_id = 'pp1' where prod_id ='p1'")
    # # Since we have added the on update and on delete cascade, we do not need to execute below query to update stock. Chnages in product will auto reflect in stock
    # #cur.execute("update stock set prod_id = 'pp1' where prod_id ='p1'")
 
    # #4 The depot d1 changes its name to dd1 in Depot and Stock.
    cur.execute("update depot set dep_id = 'dd1' where dep_id ='d1'")
    # # Since we have added the on update and on delete cascade, we do not need to execute below query to update stock. Changes in depot will auto reflect in stock
    # #cur.execute("update stock set dep_id = 'dd1' where dep_id ='d1'")
    # #1 The product p1 is deleted from Product and Stock.
    cur.execute("delete from product where prod_id='pp1'")
    # # Since we have added the on update and on delete cascade, we do not need to execute below query to update stock. Changes in product will auto reflect in stock
    # #cur.execute("delete from stock where prod_id='pp1'")
 
    # #2 The depot d1 is deleted from Depot and Stock.
    cur.execute("delete from depot where dep_id='dd1'")
    # # Since we have added the on update and on delete cascade, we do not need to execute below query to update stock. Changes in depot will auto reflect in stock
    # #cur.execute("delete from stock where dep_id='dd1'")
    cur.execute("select * from stock")
    rows = cur.fetchall()
    for row in rows:
        print(row)
 
except (Exception, psycopg2.DatabaseError) as err:
    print(err)
    print("Transactions could not be completed so database will be rolled back before start of transactions")
    con.rollback()
finally:
    if con:
        con.commit()
        cur.close
        con.close
        print("PostgreSQL connection is now closed")
