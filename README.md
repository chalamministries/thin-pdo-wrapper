# Thin PDO Wrapper

A simple database client utilizing PHP PDO.

## Advantages
- Maintains at most one connection to slave DB and one to master DB per request.
- Automatically uses slave connection for data retrieval, master for writes.
- Randomized slave connection, with automatic fallback to master if no slaves exist.
- Enforces data sanitization (using PDO prepared statements and bind parameters).
- Catches all errors, and writes them to error log if configured to do so.
- Automatic timestamps for creates and updates
- Handles multiple inserts with a single query.
- Fallback to custom queries if one needs to do a join or second order query.
- Deals exclusively with associative arrays for input/output

## Initializing And Setup
```
   $pdo = PDOWrapper::instance();
```

If you want to enable logs, set the LOG_ERRORS value to true.
```
   $pdo->LOG_ERRORS = true;
```

To use a custom log file, set LOG_FILE to the absolute path of your custom log file.
```
   $pdo->LOG_FILE = '/var/www/logs/sql_log.txt';
```

You will then want to configure your database details. Master connections are used for write operations.
While Slave connections are only used for read only connections such as SELECT and QUERY unless you sepcifically specify to use the Master Database when calling those methods.

### Master
```
  $pdo->configMaster(
    'myhostname.example',
    'my_database_name',
    'my_database_user',
    'my_database_password',
    'my_database_port' // optional
  );
```

### Slaves (optional)
```
  $pdo->configSlave(
    'myhostname1.example',
    'my_database_name',
    'my_database_user',
    'my_database_password',
    'my_database_port' // optional
  );
```
You can set more than one Slave connection by simply calling the configSlave method again.

> In the use case of replicated databases, you can configure both Master and Slave and this class will use the Master to write changes and Slaves to read only.
>
> If you only have 1 connection, this class will automatically configure a slave connection for you.

## Examples

### Selecting
Call the **select** method and pass the following parameters.
Returns an associate array representing the fetched table row or false on failure.


| table | the name of the db table we are retreiving the rows from |
| params | associative array representing the WHERE clause filters |
| limit (optional) | the amount of rows to return |
| start (optional) | the row to start on, indexed by zero |
| order_by (optional) | an array with order by clause |
| use_master (optional) | use the master db for this read |
 
```
  $results = $pdo->select('post', array('thread_id' => $thread_id));
  $results = $pdo->select('thread', array('open' => 0), $limit + 1, $start, array('favs' => 'DESC'));
```

### Inserting
Call the **insert** method to add a row to the specified table.
Returns new primary key of inserted table or false on failure.

| table | the name of the db table we are adding row to |
| params | associative array representing the columns and their respective values |

```
  $invite_id = $pdo->insert('invite', array(
    'user_id' => $user_id,
    'text' => $text,
    'invite_key' => $invite_key
  ));
```

### Updating
Call the **update** method to update a row to the specified table.
Returns the amount of rows updated or false on failure.

| table | the name of the db table we are updating |
| params | associative array representing the columns and their respective values to update |
| wheres (Optional) | associative array representing the where clause of the query |


### Complex Queries with bind parameters
Call **query** to get an array response or **queryFirst** to get just the first row returned.

```
  $post = $pdo->queryFirst('
    SELECT invite.*, user.name FROM invite
    LEFT JOIN user ON user.id=invite.user_id
    WHERE invite_key = :invite_key', 
    array(':invite_key' => $invite_key));
```


