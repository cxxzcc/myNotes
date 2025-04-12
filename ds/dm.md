


1a.
for table scans  it will be more efficient，
only one seek operation, then can read all data sequentially

1b.
yes,  logical order does not match the physical order
need a seek operation to locate each block that is out of order.y

2a
index(B,C) will have more values to be indexed than  index B
the height of the tree for (B,C) >= b

2b
b have more same values
hash(b,c) have more distinct entries than hash(b)

3a
nested-loop or block nested-loop join
it cannot use index on C

3b
merge-join
data are sorted on C in both tables
fastest algorithm

4a
number of rows B=5 is large 
not use non-clustered index, prefer table scan 

4b
number of rows B>=6 is low or no rows B>=6


5a
uncommitted values, non-repeatable reads, or phantom reads
Read Uncommitted, Read Committed, Repeatable Read
not Serializable 

5b
No, T only have read operations


6a
T1 S-lock on the table
T2 IX-lock on the table before X-lock on the record 

6b
yes
T1 read, and T2 write on the same object (Q), 
if T2 before T1 
T1 rolled back.


7a
 Yes. 
If all dirty pages have already been written to disk by the checkpoint, then transaction commit  between checkpoint and crash
Because it committed, it is marked for redo.

7b
redo phase redo CLR
undo phase  undo next LSN

8a
 Queries that (A,)B or (A,)C 
avoid get data that they do not use 

8b
 create a materialized view with the precomputed results of the inner 
query
in order to avoid having to repeatedly compute those results multiple times 


9a
query can be answered straight from the index without going to the actual table data


9b














