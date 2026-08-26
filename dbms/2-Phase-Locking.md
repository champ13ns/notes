2 Phase Locking -> 2 phase is rule for when a transaction may acquire and release locks.
It has two phases -> Growing phase and shrinking phase.

1. Growing phase -> in growing phase, a transaction may 
                     1. acquire new locks.
                     2. upgrade locks.
                     3. cannot release locks.
   
2. Shrinking phase -> a transaction may
                     1. can release locks
                     2. cannot acquire new locks.

The golden 2PL rule
The easiest way to remember it:
After a transaction releases its first lock, it can never acquire another lock.


Lock - X1(X)
Lock - X2(Y)
W1(X)
W1(Y)
Unlock (x1)
Unlock (x2)



Example : 
W1(X)
W2(X)
W2(Y)
W1(Y)



| Feature            | Star ⭐           | Snowflake ❄️      |
| ------------------ | ---------------- | ---------------- |
| Fact table         | Usually one      | Usually one      |
| Dimensions         | Denormalized     | Normalized       |
| Redundancy         | Higher           | Lower            |
| Number of tables   | Fewer            | More             |
| Joins              | Fewer            | More             |
| Query simplicity   | Simpler          | More complex     |
| Query speed        | Generally faster | Generally slower |
| Storage efficiency | Lower            | Better           |
