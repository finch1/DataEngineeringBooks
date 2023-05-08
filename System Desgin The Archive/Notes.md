# ACID
## Atomicity
> Commit to the entirety of the transaction occuring. Ensuring that any commit finishes the entire operation successfully. 
> Have no transaction at all. In case of a lost connection, partially finalized transaction, unkown state,  the transaction is rolled back to the state prior to the commit being initialized
> Atomicity ensures that a transaction is either commited or not no matter the interruptions
## Consistency
> Maintaining data integrity constraints
## Isolation
> Transactions are executed in a distinct order after each other and considered to be *serializable*. Any occuring read/writes won't be affected by other read/writes. A global order queing transactions. This doesn't mean operations cannot happen at the same time, but that transactions won't effect each other in the same occurance. Tradeoff between wait cycles and data security
## Durability
> Ensures changes are stored permanently even in event of failure. 


# Database Isolation levels
Isolation allows transactions to execute as if there are no other concurrently running transactions.        

> **Serializable**: This is the highest isolation level. Concurrent transactions are guaranteed to execute in sequence.under SERIALIZABLE reads the second select is guaranteed to see exactly the same rows as the first. No row can change, nor deleted, nor new rows could be inserted by a concurrent transaction.      

> **Repeatable Read**: Data read during the transaction stays the same as the transaction starts. Is a higher isolation level, that in addition to the guarantees of the read committed level, it also guarantees that any data read cannot change, if the transaction reads the same data again, it will find the previously read data in place, unchanged, and available to read. under REPEATABLE READ the second SELECT is guaranteed to display at least the rows that were returned from the first SELECT unchanged. New rows may be added by a concurrent transaction, but the existing rows cannot be deleted nor changed.      

> **Read Committed**: Data modification can only be read after the transaction is committed. It guarantees that any data read was committed at the moment is read. It simply restricts the reader from seeing any intermediate, uncommitted, 'dirty' read. It makes no promise whatsoever that if the transaction re-issues the read, will find the Same data, data is free to change after it was read.

> **Read Uncommitted**: The data modification can be read by other transactions before the transaction is committed.        

> **Non Repeatable Read**: Occurs when during a course of a transaction, a row is retrieved twice and the values between rows differ between reads      

> **Phantom Read**:  Occurs when in the course of a transaction, two identical queries are executed and the collection of rows returned by the second is different from the first       

> **Dirty read**: Occurs when one transaction reads the data written by another uncommitted transaction. The danger with dirty reads is that transaction might never commit, leaving the original read with *dirty data*.       

> **Read (Shared Lock)**: locks table for read only access. If one transaction tries to update or write, it will be prevented until the lock is released.      
> **Write (Exclusive Lock)**: transactions are queued in FIFO. Insert, Update Delete. Lock released after COMMIT or ROLLBACK.       
> **Deadlocks**: happen when two transactions wait for each other/a lock. One way to avoid this is by Timeout, if lock is not given within the period, transaction is canceled. 

> **Optimistic Concurrency**: The DB engine assumes concurrency conflicts are rare. Doesn't lock. It reports any failures and possibly retries operation. 
One method is to select the row with its version number, then update it only if the version number is still the same:
```sql
UPDATE people SET person_name = @p0 WHERE personID = @p1 and [Version] = @p2
```
If a concurrent update occurs, code returns *zero rows effected*, throws an exception which must be handled appropriately. Same exception can happen during *delete*. *Inserts* can only throw *constraint exceptions*. 

Another method is to query the data with *Repeatable Read* isolation level, to prevent other transactions from altering. **Note**: this is ideal when only the row is locked, NOT the entire table. Also, a user input operation implies long wait time = lengthy lock. 

> **Pessimistic Concurrency**: Locks data upfront and then proceeds to changing.        

> **Application managed concurrency**: Can provide finer grained control such as the concurrency token: [Version number, GUID, Timestamp, Hash of certain columns]. Some light DBs do not have conc-control.        

```C#
using var context = new PersonContext();
// Fetch a person from database and change phone number
var person = context.People.Single(p => p.PersonId == 1);
person.PhoneNumber = "555-555-5555";

// Change the person's name in the database to simulate a concurrency conflict
context.Database.ExecuteSqlRaw(
    "UPDATE dbo.People SET FirstName = 'Jane' WHERE PersonId = 1");

var saved = false;
while (!saved)
{
    try
    {
        // Attempt to save changes to the database
        context.SaveChanges();
        saved = true;
    }
    catch (DbUpdateConcurrencyException ex)
    {
        foreach (var entry in ex.Entries)
        {
            if (entry.Entity is Person)
            {
                var proposedValues = entry.CurrentValues;
                var databaseValues = entry.GetDatabaseValues();

                foreach (var property in proposedValues.Properties)
                {
                    var proposedValue = proposedValues[property];
                    var databaseValue = databaseValues[property];

                    // TODO: decide which value should be written to database
                    // proposedValues[property] = <value to be saved>;
                }

                // Refresh original values to bypass next concurrency check
                entry.OriginalValues.SetValues(databaseValues);
            }
            else
            {
                throw new NotSupportedException(
                    "Don't know how to handle concurrency conflicts for "
                    + entry.Metadata.Name);
            }
        }
    }
}
```

# Example Transaction Code Design in .NET
**DBTransaction** class is available for writing vendor independent code requiring transactions. The scope of the transaction is limited ot the connection.         
**Transactions** are best run on SQL server side, embedded in stored procs. Otherwise, in code, the transaction is executed in the try block and rolled back if the transaction or connection fails
```C#
using (SqlConnection connection = new SqlConnection(connectionString))
{
    connection.Open();

    // Start a local transaction.
    SqlTransaction sqlTran = connection.BeginTransaction();

    // Enlist a command in the current transaction.
    SqlCommand command = connection.CreateCommand();
    command.Transaction = sqlTran;

    try
    {
        // Execute two separate commands.
        command.CommandText =
          "INSERT INTO Production.ScrapReason(Name) VALUES('Wrong size')";
        command.ExecuteNonQuery();
        command.CommandText =
          "INSERT INTO Production.ScrapReason(Name) VALUES('Wrong color')";
        command.ExecuteNonQuery();

        // Commit the transaction.
        sqlTran.Commit();
        Console.WriteLine("Both records were written to database.");
    }
    catch (Exception ex)
    {
        // Handle the exception if the transaction fails to commit.
        Console.WriteLine(ex.Message);

        try
        {
            // Attempt to roll back the transaction.
            sqlTran.Rollback();
        }
        catch (Exception exRollback)
        {
            // Throws an InvalidOperationException if the connection
            // is closed or the transaction has already been rolled
            // back on the server.
            Console.WriteLine(exRollback.Message);
        }
    }
}
```
# Clustered Index Scan
The Clustered Index Scan operator scans the clustered index specified in the Argument column of the query execution plan. When an optional WHERE:() predicate is present, only those rows that satisfy the predicate are returned. If the Argument column contains the ORDERED clause, the query processor has requested that the output of the rows be returned in the order in which the clustered index has sorted it. If the ORDERED clause is not present, the storage engine scans the index in the optimal way, without necessarily sorting the output. Clustered Index Scan is a logical and physical operator.
# Clustered Index Seek
The Clustered Index Seek operator uses the seeking ability of indexes to retrieve rows from a clustered index. The Argument column contains the name of the clustered index being used and the SEEK:() predicate. The storage engine uses the index to process only those rows that satisfy this SEEK:() predicate. It can also include a WHERE:() predicate where the storage engine evaluates against all rows that satisfy the SEEK:() predicate, but this is optional and does not use indexes to complete this process.

If the Argument column contains the ORDERED clause, the query processor has determined that the rows must be returned in the order in which the clustered index has sorted them. If the ORDERED clause is not present, the storage engine searches the index in the optimal way, without necessarily sorting the output. Allowing the output to retain its ordering can be less efficient than producing nonsorted output. When the keyword LOOKUP appears, then a bookmark lookup is being performed. In SQL Server 2008 (10.0.x) and later versions, the Key Lookup operator provides bookmark lookup functionality. Clustered Index Seek is a logical and physical operator.
# NonClustered Index Scan
A NonClustered Index Scan is effectively the same as a Clustered Index Scan. The difference is that the leaf level of a non-clustered index contains index pages rather than data pages, which means this is generally less I/O than a clustered index scan and is not analogous with a table scan.
# NonClustered Index Seek
# Index Scan
	The Index Scan operator retrieves all rows from the nonclustered index specified in the Argument column. If an optional WHERE:() predicate appears in the Argument column, only those rows that satisfy the predicate are returned. Index Scan is a logical and physical operator.
# Index Seek
The Index Seek operator uses the seeking ability of indexes to retrieve rows from a nonclustered index. The Argument column contains the name of the nonclustered index being used. It also contains the SEEK:() predicate. The storage engine uses the index to process only those rows that satisfy the SEEK:() predicate. It optionally may include a WHERE:() predicate, which the storage engine will evaluate against all rows that satisfy the SEEK:() predicate (it does not use the indexes to do this). If the Argument column contains the ORDERED clause, the query processor has determined that the rows must be returned in the order in which the nonclustered index has sorted them. If the ORDERED clause is not present, the storage engine searches the index in the optimal way (which does not guarantee that the output will be sorted). Allowing the output to retain its ordering may be less efficient than producing nonsorted output. Index Seek is a logical and physical operator.

# Include vs Composite
Include columns can only be used to supply columns to the SELECT portion of the query. They cannot be used as part of the index for filtering. https://stackoverflow.com/questions/3883984/composite-index-vs-include-covering-index-in-sql-server