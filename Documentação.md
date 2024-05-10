# 5.7 River crossing problem

This is from a problem set written by Anthony Joseph at U.C. Berkeley, but
I don’t know if he is the original author. It is similar to the H2O problem in
the sense that it is a peculiar sort of barrier that only allows threads to pass in
certain combinations.
Somewhere near Redmond, Washington there is a rowboat that is used by
both Linux hackers and Microsoft employees (serfs) to cross a river. The ferry
can hold exactly four people; it won’t leave the shore with more or fewer. To
guarantee the safety of the passengers, it is not permissible to put one hacker
in the boat with three serfs, or to put one serf with three hackers. Any other
combination is safe.
As each thread boards the boat it should invoke a function called board. You
must guarantee that all four threads from each boatload invoke board before
any of the threads from the next boatload do.
After all four threads have invoked board, exactly one of them should call
a function named rowBoat, indicating that that thread will take the oars. It
doesn’t matter which thread calls the function, as long as one does.
Don’t worry about the direction of travel. Assume we are only interested in
traffic going in one of the directions.

## 5.7.1 River crossing hint
Here are the variables I used in my solution
```python
barrier = Barrier (4)
mutex = Semaphore (1)
hackers = 0
serfs = 0
hackerQueue = Semaphore (0)
serfQueue = Semaphore (0)
local isCaptain = False
```
hackers and serfs count the number of hackers and serfs waiting to board.
Since they are both protected by mutex, we can check the condition of both
variables without worrying about an untimely update. This is another example
of a scoreboard.
hackerQueue and serfQueue allow us to control the number of hackers and
serfs that pass. The barrier makes sure that all four threads have invoked board
before the captain invokes rowBoat.
isCaptain is a local variable that indicates which thread should invoke row.

## 5.7.2 River crossing solution
The basic idea of this solution is that each arrival updates one of the counters
and then checks whether it makes a full complement, either by being the fourth
of its kind or by completing a mixed pair of pairs.
I’ll present the code for hackers; the serf code is symmetric (except, of course,
that it is 1000 times bigger, full of bugs, and it contains an embedded web
browser):

### Oxygen Code
```python
mutex . wait ()
hackers += 1
if hackers == 4:
hackerQueue . signal (4)
hackers = 0
isCaptain = True
elif hackers == 2 and serfs >= 2:
hackerQueue . signal (2)
serfQueue . signal (2)
serfs -= 2
hackers = 0
isCaptain = True
else :
mutex . signal () # captain keeps the mutex

hackerQueue . wait ()

board ()
barrier . wait ()

if isCaptain :
rowBoat ()
mutex . signal ()
```
As each thread files through the mutual exclusion section, it checks whether
a complete crew is ready to board. If so, it signals the appropriate threads,
declares itself captain, and holds the mutex in order to bar additional threads
until the boat has sailed.
The barrier keeps track of how many threads have boarded. When the last
thread arrives, all threads proceed. The captain invoked row and then (finally)
releases the mutex.
