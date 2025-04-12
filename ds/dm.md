


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




