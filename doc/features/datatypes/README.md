# CQL data types to JavaScript types

When retrieving the value of a column from a `Row` object, the value is typed according to the following table.

CQL data type|JavaScript type
---|---
ascii|String
bigint|[Long](numerical)
blob|Buffer
boolean|Boolean
counter|[Long](numerical)
date|[LocalDate](datetime)
decimal|[BigDecimal](numerical)
double|[Number](numerical)
float|[Number](numerical)
inet|InetAddress
int|[Number](numerical)
list|[Array](collections)
map|[Object / ECMAScript 6 Map](collections)
set|[Array / ECMAScript 6 Set](collections)
smallint|[Number](numerical)
text|String
time|[LocalTime](datetime)
timestamp|[Date](datetime)
timeuuid|[TimeUuid](uuids)
tinyint|[Number](numerical)
tuple|[Tuple](tuples)
uuid|[Uuid](uuids)
varchar|String
varint|[Integer](numerical)

Note: There are few data types defined in the ECMAScript standard for JavaScript. This usually represents a problem when
you are trying to deal with data types that come from other systems.

## Encoding data 

When encoding data, on a normal execute with parameters, the driver tries to guess the target type based on the input
type. Values of type Number will be encoded as double (because Number is double or IEEE 754 value).

Consider the following example:

```javascript
var key = 1000;
client.execute('SELECT * FROM table1 where key = ?', [key], callback);
```

If the key column is of type int, the execution fails. There are two possible ways to avoid this type of problem:

### Prepare the data (recommended)

Using prepared statements provides multiple benefits. Prepared statements are parsed and prepared on the Cassandra nodes
and are ready for future execution. Also, the driver retrieves information about the parameter types which allows an
**accurate mapping between a JavaScript type and a Cassandra type**.

Using the previous example, setting the prepare flag in the queryOptions will fix it:

```javascript
// prepare the query before execution 
client.execute('SELECT * FROM table1 where key = ?', [key], { prepare : true }, callback);
```

When using prepared statements, the driver prepares the statement once on each host to execute multiple times.

### Hinting the target data type

Providing hints in the query options is another way around it.

```javascript
// Hint that the first parameter is an integer 
client.execute('SELECT * FROM table1 where key = ?', [key], { hints : ['int'] }, callback);
```