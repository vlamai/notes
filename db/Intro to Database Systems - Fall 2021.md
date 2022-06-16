# [[Lecture 02 - Intermidiate SQL]] notes

Database Management Systems (**DBMS**) are software systems used to store, retrieve, and run queries on data.
A DBMS serves as an interface between an end-user and a database, allowing users to create, read, update, and delete data in the database.

Data manipulation lang (DML)
Data definition lang (DDL)
Data control lang (DCL)

Also includes: 
- View definition
- Integrity and referential control 
- Transactions

SQL based on bags (**duplicates**) not on sets

**HAVING** filter results based on aggregate computation.

# [[Lecture 03 - Database Storage (Part I)]] notes

## Storage hierarchy
### volatile  
Faster, smaller, expensive. Random access, byte addressable.
- CPU registers 
- CPU cashes
- DRAM
### non-volatile
Slower, larger, cheaper. Sequential access, block addressable.
- SSD
- HDD
- Network storage


-- no sound =(

# [[Lecture 04 - Database Storage (Part II)]] notes

### Slotted pages 
![[Pasted image 20220615214052.png]]


The slot array maps **slots** to the tuples starting position 
The **header** tracks of number of used slots and offset of the starting location of the last used slot.


### Log structured 
![[Pasted image 20220615214452.png]]

append log records to the file of how the database was modified.



[EXPLAIN — show the execution plan of a statement](https://www.postgresql.org/docs/current/sql-explain.html) #PostgreSQL #sql 

````sql
\timing [on|off]       toggle timing of commands (currently off)
````


## n-ary storage model 
good for #OLTP
Advantages : 
- Fast insert, update,delete 
- Good for queries that need the entire tuple
Disadvantages 
- Not good for scanning large portions of the table 

## decomposition storage model 
ideal for #OLAP workloads

column store 

# [[Lecture 05 - Buffer Pools]] notes




\\\\\\\
------------------------------------
#questions

1. query optimizer
2. relation algebra 
3. slotted pages
4. n ary storage model (NSM)


#db #sql