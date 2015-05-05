---
layout: post
title: "Dealing with big MySQL migrations on Rails"
date: 2015-04-15 03:31:48 -0700
comments: true
categories: 
---

Migrating tables in MySQL can be a very painful thing to do if you need to alter the schema of a big table.
Altering the schema of a table in MySQL will lock the table which means you will only be able to read from that table during the period of time that the table is being migrated. This means trouble for most applications out there, this means trouble if you write to this table very often. 

##### Why would altering the schema lock a MySQL table ?
The reason is because the way ``ALTER TABLE`` will copy over the table to a new one and incorporate the changes in that copy.
If there are insertions or updates being done to the table during that copy time then there's possibility of losing some of those changes.

During and ``ALTER TABLE`` MySQL will create a copy of the table with the new schema, migrate all the data from the old table into the new one, drop the original table and then rename the new table. During this time you can read from the table but all the writes to the table are stalled until the new table is ready. After the modification of the table is done all the queries that were stalled get redirected to the new table. 

##### How do we migrate our table? 

Now that we know how MySQL perform this operations we can look for ways to optimize this process and achieve the same result without preventing our application from writing to our table during the migration. Let's first identify what are our needs for regarding this table.

Let's identify what are our main concerns regarding this important migration.

- We want to be able to read from the table the entire time
- We want to be able to write to the table the entire time
- We don't want to lose the integrity of the data 

Great! We already have one of the bullet points covered, locking the table doesn't prevent us from reading from the table, so let's move to the next one.

Since we want to be able to write to the table the entire time we won't be able to migrate our table without getting some tooling in place and handle the transaction differently. Probably the easiest way to migrate our table and not lock our application when it tries to write to our table is to have our model write to a different table.

How do we get our model to use a new table? Well we have two options for this. 

- We can tell our model the name of it's new table, like so.
```
class OurAwesomeModel < ActiveRecord::Base
    self.table_name = "our_awesome_table_name"
end
```
- We can create a new table with the same name of the old one. For this we need to rename our original table to something else.
```
class AddIndexOnCouponsCreatedAt < ActiveRecord::Migration
  def change
    rename_table :our_awesome_table_name, :our_deprecated_awesome_table_name
    execute "CREATE TABLE our_awesome_table LIKE our_deprecated_awesome_table_name"
  end
```

We don't want to change the behavior of our models so we want to do it on a migration just like our second alternative.

Now we are reade to execute our slow migration without losing the ability to write to our table, the only problem is that by swaping out our original table we lost the ability to access the information on the table, we can read to ur new table but all our data is on the old one.

How do we fix this? well, an option is to first copy over all the data from the original table to the new one before renaming it.

```
class AddIndexOnCouponsCreatedAt < ActiveRecord::Migration
  def change
    execute "CREATE TABLE our_awesome_new_ table_name LIKE our_deprecated_awesome_table_name"
    execute "INSERT FROM SELECT * FROM our_awesome_table_name" 
    rename_table :our_awesome_table_name, :our_deprecated_awesome_table_name
    rename_table :our_awesome_new_table_name, :our_awesome_table_name
  end
```

Perfect, now we have a duplicate table that we can read and write to while we migrate the other one and sawp them again. 

```
class AddIndexOnCouponsCreatedAt < ActiveRecord::Migration
  def change
    execute "CREATE TABLE our_awesome_new_ table_name LIKE our_deprecated_awesome_table_name"
    execute "INSERT FROM SELECT * FROM our_awesome_table_name" 
    rename_table :our_awesome_table_name, :our_deprecated_awesome_table_name
    rename_table :our_awesome_new_table_name, :our_awesome_table_name

    # Now we can finally execute our super slow alter table
    execute "alter table our_deprecated_awesome_table_name"
    
    # After the alter table is done we can swap the 2 tables again
    rename_table :our_awesome_table_name, :our_new_awesome_table_name
    rename_table :our_awesome_deprecated_table_name, :our_awesome_table_name
  end
```

Awesome, we are almost there. Our only problem now is precisely our last bullet point. 

##### Data integrity. 

There're a couple places in our migration in which our 2 tables could've gotten out of sync. 

First when we were copying the data over to our secundary table some other session could've written to the table. That specific case is not a problem since we're switching over to that table again at the end, but it means that during the time of the alter table we won't be able to access that data. This also brings to light another problem, if we have any ``auto_increment`` field in the table this new data that got entered in the original table during the data copy will cause conflicts since it won't be reflected in the ''auto_increment'' of our new table.

The other place is while we were altering our original table and our model was writing to our temporary table.

Lucky for us there's an easy solution for these two problems. First let's table the easiest one. We'll set the ``auto_increment`` in our temporary table equal to ``max(original_table_auto_increment_column_name) + some_error_margin``,
the error margin number should be a number higher than the number of records that might get created during the time of the copy. To avoid collisions you should pick a number that gives you room for error.

Our other problem we'll with a simple solution. We'll capture the time when we start copying data from the original table and also when we finish, that way we'll now that we have to copy over any record with an ``updated_at`` timestamp later than our captured time and that way we'll be able to keep our data integrity.

```
class AddIndexOnCouponsCreatedAt < ActiveRecord::Migration
  def change
    execute "CREATE TABLE our_awesome_new_ table_name LIKE our_deprecated_awesome_table_name"
    execute "ALTER TABLE our_awesome_new_table_name AUTO_INCREMENT=#{max(original_table_auto_increment_column_name) + 1000}"
    initial_copy_time = Time.now
    execute "INSERT INTO our_awesome_new_table_Name SELECT * FROM our_awesome_table_name"
    rename_table :our_awesome_table_name, :our_deprecated_awesome_table_name
    rename_table :our_awesome_new_table_name, :our_awesome_table_name
    
    initial_time_of_new_table = Time.now
    # Now we copy over any data that might have been inserted during the copy time
    execute "INSERT INTO our_awesome_table_name SELECT * FROM our_deprecated_awesome_table_name WHERE updated_at > #{initial_copy_time}"
    
    # Now we can finally execute our super slow alter table
    execute "alter table our_deprecated_awesome_table_name"
    
    # After the alter table is done we can swap the 2 tables again
    rename_table :our_awesome_table_name, :our_new_awesome_table_name
    rename_table :our_awesome_deprecated_table_name, :our_awesome_table_name
    
    # Now let's copy any data that was inserted into the temporary table
    execute "INSERT INTO our_awesome_table_name SELECT * FROM our_new_awesome_table_name WHERE updated_at > #{initial_time_of_new_table}"
  end
```

Now our table should now have the new schema, all the data, and no application downtime.
