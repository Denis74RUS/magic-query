What is Magic-query?
====================

Magic-query is a PHP library that helps you work with complex queries that require
a variable number of parameters.

How does it work?
-----------------

Easy! You write the query with all possible parameters.

```php
$sql = "SELECT * FROM users WHERE name LIKE :name AND country LIKE :country";

// Get a MagicQuery object.
$magicQuery = new MagicQuery();

// Let's pass only the "name" parameter
$result = $magicQuery->build($sql, [ "name" => "%John%" ]);
// $result = SELECT * FROM users WHERE name LIKE '%John'

// Let's pass no parameter at all!
$result2 = $magicQuery->build($sql, []);
// $result2 = SELECT * FROM users
```

Why should I care?
------------------

Because it is **the most efficient way to deal with queries that can have a variable number of parameters**!
Think about a typical datagrid with a bunch of filter (for instance a list of products filtered by name, company, price, ...).
If you have the very common idea to generate the SQL query using no PHP library, your code will look like this:

<div class="alert"><strong>You should not do this!</strong></div>

```php
// People usually write queries like this:
$sql = "SELECT * FROM products p JOIN companies c ON p.company_id = c.id WHERE 1=1 ";
// They keep testing for parameters, and concatenating strings....
if (isset($params['name'])) {
	$sql .= "AND p.name LIKE '".addslashes($params['name'])."%'";
}
if (isset($params['company'])) {
	$sql .= "AND c.name LIKE '".addslashes($params['company'])."%'";
}
if (isset($params['country'])) {
	$sql .= "AND c.country LIKE '".addslashes($params['country'])."%'";
}
// And so on... for each parameter, we have a "if" statement
```

Concatenating SQL queries is **dangerous** (especially if you forget to protect parameters).
You can always use parameterized SQL queries, but you will still have to concatenate the filters.

To avoid concatenating strings, frameworks and libraries have used different strategies. Building a full ORM (like
Doctrine or Propel) is a good idea, but it makes writing complex queries even more complex. Other frameworks like
Zend are building queries using function calls. These are valid strategies, but you are no more typing SQL queries
directly, and let's face it, it is always useful to use a query directly.

This is where Magic-query becomes helpful.


How does it work under the hood?
--------------------------------

A lot happens to your SQL query. It is actually parsed (thanks to a modified
version of the php-sql-parser library) and then changed into a tree.
The magic happens on the tree where the node containing unused parameters
are simply discarded. When it's done, the tree is changed back to SQL and
"shazam!", your SQL query is purged of useless parameters!
