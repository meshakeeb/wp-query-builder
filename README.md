[![Build Status](https://travis-ci.org/meshakeeb/wp-query-builder.svg?branch=master)](https://travis-ci.org/meshakeeb/wp-query-builder)

# WordPress Query Builder

An expressive query builder for WordPress. Build for developers by developers.

## How to use in managed environments

If you wish to use this extension in a managed environment, simply install using `composer`:

```
composer require thewpleague/wp-query-builder
```

To use the Query builder

```php
include('vendor/autoload.php');

$table = new TheLeague\Database\Database::table( 'users' );
$table = wp_query_builder( 'users' );

$table->where( 'id', 2 )
  ->orderBy( 'id', 'desc' )
  ->limit( 20 )
  ->get();
```

## Data Sanitisation

The purpose of this library is to provide an **expressive** and **safe*** way
to run queries against your WordPress database (typically involving custom tables).

To this end all **values** provided are escaped, but note that **column and table**
names are not yet escaped. In any case, even if they were you should be whitelisting
any allowed columns/tables: otherwise using user-input, or other untrusted data to
determine the column/table could allow an attacker to retrieve data they shouldn't
or generate a map of your database.

## Examples

### Select statement

```php
$table = new TheLeague\Database\Database::table( 'users' )

// select * from wp_users
$table->get();

// select distinct * from wp_users
$table->distinct()->get();

// select SQL_CALC_FOUND_ROWS * from wp_users
$table->found_rows()->get();
```

#### Specify columns for Select statement

```php
// select id from wp_users
$table->select( 'id' )->get();

// select id, user_login from wp_users
$table->select( 'id, user_login' )->get();

// select id, user_login from wp_users
$table->select( array( 'id', 'user_login' ) )->get();

// select id, user_login as username from wp_users
$table->select( array( 'id', 'user_login as username' ) )->get();

// select id, user_login as username from wp_users
$table->select( array(
	'id',
	'user_login' => 'username'
) )->get();
```

#### Select statement with count, sum, avg

```php
// select count(*) from wp_users
$table->selectCount()->get();

// select count( id ) as count from wp_users
$table->selectCount( 'id', 'count' )->get();

// select sum( id ) as total from wp_users
$table->selectSum( 'id', 'total' )->get();

// select avg( id ) as average from wp_users
$table->selectAvg( 'id', 'average' )->get();
```

#### Select a single row

```php
// select * from wp_users WHERE user_email = 'admin@example.com' LIMIT 0, 1;
$table->where( 'user_email', 'admin@example.com' )->one();
```

#### To retrieve a value

```php
$email = $table->select( 'user_email' )->where( 'ID', 123 )->getVar();
```

---

### Insert statement

```php
// insert into wp_users columnA, columbB values(`value`, `value`)
$table->insert(array(
	'columnA' => 'value',
	'columnB' => 'value',
), array( '%s', '%s' ) );
```

---

### Update statement

```php
// update wp_users set foo = `bar`
$table->set( 'foo', 'bar' )->update();

// update wp_users set foo = `bar`, bar = `foo`
$table->set( 'foo', 'bar' )
	->set( 'bar', 'foo' )
	->update();

// update wp_users set foo = `bar`, bar = `foo`
$table->set(array(
	'foo' => 'bar',
	'bar' => 'foo',
))->update();

// update wp_users set foo = `bar` where id = 1 limit 0,1
$table->set( 'foo', 'bar' )
	->where( 'id', 1 )
	->limit( 1 )
	->update();
```

---

### Delete statement

```php
// delete from wp_users where id = 1 limit 0,1
$table->where( 'id', 1 )->limit( 1 )->delete();

// truncate table wp_users
$table->truncate();
```

---

### Where statement

```php
// select * from wp_users where id = 2
$table->where( 'id', 2 )->get();

// select * from wp_users where id != 42
$table->where( 'id', '!=', 42 )->get();

// select * from wp_users where id = 2 and active = 1
$table->where( 'id', 2 )->where( 'active', 1 )->get();
$table->where( 'id', 2 )->andWhere( 'active', 1 )->get();

// select * from wp_users where id = 2 or active = 1
$table->where( 'id', 2 )->orWhere( 'active', 1 )->get();

// select * from wp_users where ( a = 'b' or c = 'd' )
$table->orWhere( array(
	array( 'a', 'b' ),
	array( 'c', 'd' ),
) )->get();

// select * from wp_users where a = 1 or ( a > 10 and a < 20 )
$table->where( 'a', 1 )
	->orWhere( array(
		array( 'a', '>', 10 ),
		array( 'a', '<', 20 ),
	), 'and' )->get();

// select * from wp_users where a = 1 or ( a > 10 and a < 20 ) and c = 30
$table->where( 'a', 1 )
	->orWhere( array(
		array( 'a', '>', 10 ),
		array( 'a', '<', 20 ),
	), 'and' )
	->andWhere( 'c', 30 )->get();

// select * from wp_users where id in (23, 25, 30)
$table->whereIn( 'id', array( 23, 25, 30 ) );

// select * from wp_users where id not in (23, 25, 30)
$table->whereNotIn( 'id', array( 23, 25, 30 ) );

// select * from wp_users where skills in ('php', 'javascript', 'ruby')
$table->whereIn( 'skills', array( 'php', 'javascript', 'ruby' ) );

// select * from wp_users where id between 10 and 100
$table->whereBetween( 'id', array( 10, 100 ) );

// select * from wp_users where id not between 10 and 100
$table->whereNotBetween( 'id', array( 10, 100 ) );

// select * from wp_users where dates between '10-04-2018' and '10-09-2018'
$table->whereBetween( 'dates', array( '10-04-2018', '10-09-2018' ) );

// select * from wp_users where id is null
$table->whereNull( 'id' )->get();

// select * from wp_users where name is not null
$table->whereNotNull( 'name' )->get();
```

---

### Groupby and Having statement

```php
// select * from wp_users group by id
$table->groupBy( 'id' )->get();

// select count(id) as total, post_id from phpunit where post_id > 10 group by post_id having count(id) > 25
$table->selectCount( 'id', 'total' )
	->select( 'post_id' )
	->whereIn( 'post_id', '>', 10 )
	->groupBy( 'post_id' )
	->having( 'count(id)', '>', 25 );
```

---

### Orderby statement

```php
// select * from wp_users order by id asc
$table->orderBy( 'id' )->get();

// select * from wp_users order by id desc
$table->orderBy( 'id', 'desc' )->get();

// select * from wp_users order by firstname desc, lastname desc
$table->orderBy( 'firstname, lastname', 'desc' )->get();

// select * from wp_users order by firstname asc, lastname desc
$table->orderBy(array(
	'firstname' => 'asc',
	'lastname'  => 'desc',
) )->get();

// select * from wp_users order by firstname <> nick
$table->orderBy( 'firstname <> nick', null )->get();
```

---

### Limit statement

```php
// select * from wp_users limit 0, 1
$table->limit( 1 )->get();

// select * from wp_users limit 20, 10
$table->limit( 10, 20 )->get();

// select * from wp_users limit 20, 10
$table->page( 2, 10 )->get();
```

### TODOs

- [ ] Add joins
- [ ] Add query cache system
