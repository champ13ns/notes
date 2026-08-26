# . ACID -> 

Atomicity,
Consistency,
Isolation ,
Durability
   
A transaction is a logical unit of database work.
Atomicity means all or nothing.
Consistency means database rules remain valid.
Isolation prevents improper interference between concurrent transactions.
Durability means committed changes survive failures.
COMMIT makes a transaction permanent.
ROLLBACK undoes uncommitted changes.


# Scheduling -> it is the process of executing multiple transactions in a database. 

## . Serial Schedules
### . Non-Serial Schedules => b.1 -> Serializable Schedules    b.2 => Non-Serializable Schedules

#### b.1 => Serializable Schedules => b.1.1 => Conflict Serializable.   b.1.2 => View Serializable.
##### b.2 => Non-Serializable Schedules => b.2.1 => Recoverable Schedules (Cascading schedules , cascadeless schedules , strict schedules) , b.2.2 => Non recoverable schedules.


`Serial Schedule` => Multiple transactions happen after completions of first transaction.

T1

R1 (a)
W1 (a)
R2 (a)

            T2
            R1(a)
            W1(a)
            R2(a)

No conflicts will happen, as transactions are working serially.


`Non Serial Schedule` -> Transactions execute in an interleaving manner. They are of two types `Serializable Schedules` and `Non Serializable Schedules`
Example of a non serial schedule.

R1(x)
W1(x)
R2(x)
W2(x)



Serializablity -> Concurrency of a non-serial schedule + Correctness of a serial schedule.


`Non-Serial Schdule -> Serializable Schedule` -> A non serial schedule that behaves like a serial schedule. Benefits ofc are concurrency, avoid anomalies , imporves performance by having concurrency.

A non serial schedule that behaves like a serial schedule. It behaves as so through the transactions had executed serially.



Suppose 
T1   R1(x)
    W1(x)
    R1(y)
    W1(y)

T2   R2(x)
    W2(x)

Schedule : 

R1(X)
W1(X)
R2(X)
R1(Y)
W1(Y)
W2(X), here t1 works on x and t2 works on y. so T1 -> T2 is same as T2 -> T1.


Another example where both transactions have same variables.
x = 100;

T1
R1(X)
X = X + 50
W1(X)
T2
R2(X)
X = X * 2
W2(X)

Consider serial execution:

T1 → T2

T1:
100 + 50 = 150
Then T2:
150 × 2 = 300
Final:
X = 300

T2 -> T1.
T2 :
x = 100; R2(x)
W2 (x) => x = 200.

T1 :
R1(x) = 200.
W1(x) = 250.

x = 250. (T2 -> T1)
x = 300 (T1 -> T2)

Now consider this schedule:

R1(X)   → T1 reads 100
R2(X)   → T2 reads 100
W1(X)   → T1 writes 150
W2(X)   → T2 writes 200

Final:
X = 200
But neither serial order produces 200. Therefore this schedule is non-serial + non-serializable.

Conflicting operations -> Two schedules are confilicting if 3 condititions are met.
a. They belong to different transactions.
b. They acts on a single shared variable.
c. Atleast one of the operations in write.

Confliect equivalent schedules -> iff the 

Conflict serializable schedule -> We use precedence graph/ serialization graph to find weather a schedule has a cycle 
present in it.

for example a scheule is like this  -> 

R1(X)
W1(X)
R2(X)
W2(X)

here R1(x) before W2(x) so T1 -> T2
and W1(x) before R2(x) so again T1 -> T2.
and W1(x) before @2(x) so again T1 -> T2.

therefore there is no cycle, meaning it is equivalent to T1 -> T2 , serial scheule. Therfore this scheule is 
### Conflict serializable.

If a scheule contains cycle that means , `Not Conflict serializable`


`View Serializable` -> A schedule is view serializable if it is view equivalent to some serial schedule.

For view equivalence, three things must remain the same:

1. Who reads the initial value.
2. Who reads values written by whom.
3. Who performs the final write.
Suppose:

W1(X)
R2(X)

Here T2 reads X produced by T1.
In a view-equivalent schedule, T2 must still read the value produced by T1.

And if:
W2(X)

is the final write on X, then T2 must remain the final writer.


1. No conflict-graph cycle
   → Conflict Serializable
   → definitely View Serializable too
   → good

2. Conflict-graph cycle
   → NOT Conflict Serializable
   → but MAY still be View Serializable
   → can still be equivalent to serial execution

3. Conflict-graph cycle
   → NOT Conflict Serializable
   → NOT View Serializable either
   → genuinely non-serializable