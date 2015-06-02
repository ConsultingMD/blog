
# Polymorphic associations can drive you insane

With apologies to [Joseph Stein](https://youtu.be/7V2lxFWBqfI):

```
    [DHH]

    Convention, Convention! Convention!
    Convention, Convention! Convention!

    [DHH AND PAPAS]

    What, day and night, must scramble my data,
    Make my tables all odd and crazy?
    And who has the right, as master of the app,
    To have the final written word?

    The Model, Active Model! Convention.
    The Model, Active Model! Convention.

    [DPCLARK AND MAMAS]

    Who must know the way to make a proper route,
    A secure, an authenticated route?
    Who must catch exceptions and find the views
    So the Model's free to scramble its words?

    Controller, Controller! Convention!
    Controller, Controller! Convention!

    [GEMS]
    At 0.0.3 I was made a gem; at 0.10 I'm on rubygems.org!
    I hear they've picked some apps for me! I hope they're kewl.

    The Gem, the Gem! Convention!
    The Gem, the Gem! Convention!

    [VIEWS]
    And what does the user see and fret with and hold?
    And what shows the wrong number in every field known?
    Preparing to store in the Model's old hold?

    The view, and the template! Convention!
    The view, and the template! Convention!

    [THE CRAPMAKER]

    Crapmaker crapmaker, make me some crap!
    Find me a number, when I wanted a map!
    Crapmaker crapmaker, look through your rules
    And make me some perfect crap!
```

OK, so maybe I'm over-reacting, but I'm coming to agree with [Bill
Karwin](http://stackoverflow.com/users/20860/bill-karwin): Rails polymorphic
associations are an anti-pattern.

Bill's point is valid: SQL DBMSs have a firm separation between meta-data and
data, and any attempt to finesse this separation will cause you to lose some of
the significant features of SQL (e.g., foreign key constraints).

But I've never been a SQL lover (I'm a Quel guy at heart), and my current
environment is all-Rails-all-the-time, so issues around declaring foreign keys
and SQL constraints are less applicable; I can rely on Rails features instead
of DBMS features. So let me tell you why they're an anti-pattern in our case.

Imagine that you have a model that uses a non-integer primary key:
```
class MyCaseGyration < ActiveRecord::Migration
  create_table "cases", :id => false do |t|
    t.string   "identifier",
    t.string   "manufacturer"
    t.integer  "user_id"
  end
end
```
```
class Case < ActiveRecord::Base
  self.primary_key = 'identifier'
end
```
(Oh, how I miss DataMapper, where I could know what my model's properties were from a single file....)

Imagine you have a few other models that use normal auto-incremented ``id`` fields. For the sake of this
example, let's use a silly User table:
```
class MyUserGyration < ActiveRecordMigration
  create_table "users" do |t|
    t.string   "name"
  end
end
```
```
class User < ActiveRecord::Base
end
```

Now, since you're a good open-source Rails coder who doesn't repeat oneself --
nor anybody else you can find on Rubygems or via StackOverflow -- imagine you'd
like to use the [audited](https://github.com/collectiveidea/audited) gem to
audit changes to these tables. We will have two problems:

1. We'll have to patch the gem.
2. And then the generated SQL will only work for very slow definitions of "work"

### We need to patch the gem

* The gem itself will want to create a model like this (taking some liberties with the [source](https://github.com/collectiveidea/audited/blob/master/lib/generators/audited/templates/install.rb)):
```
class AuditMigration < ActiveRecord::Migration
  def self.up
    create_table :audits, :force => true do |t|
      t.column :auditable_id, :integer
      t.column :auditable_type, :string
      t.column :version, :integer
      ...
    end
    add_index :audits, [:auditable_id, :auditable_type], :name => 'auditable_index'
    ...
  end
  ...
end
```

* and DSL-generated associations like:
```
   class User
     has_many :audits, :as => :auditable
   end

   class Case
     has_many :audits, :as => :auditable
   end
```

which will immediately fail when we try to save a Case, since the identifier of
the Case model is not an integer.

So we modify our generated AuditMigration to make the ``auditable_id`` a String
instead. So far so good.

Through the magic of **Convention**, this will mostly work: audited records
will be written whenever we change a Case or a User.

And any given audit record will be able to get its ``auditable`` object, since
it has a perfectly good table name and id.

### But...

But ActiveRecord is _really_ conventional.  And in particular, it doesn't
*care* what the declared type of a database field is, or even what comes along
with a column definition, unless it really has to.  So let's say you want to
find the latest version of a Case:

```
```

You'll get what you want, since the case id is actually a string in this case:

```
  2.1.5 :016 > kase = Case.first
  2.1.5 :017 > kase.audits
    Audited::Adapters::ActiveRecord::Audit Load (0.5ms)  SELECT `audits`.* FROM `audits` WHERE `audits`.`auditable_id` = 'ycyh5qwb4ubw' AND `audits`.`auditable_type` = 'Case' ORDER BY version
  ...
```

and MySQL will use the ``auditable_index`` since it can see that it has a string value for equality comparison on the ``auditable_id``.

But now you do the same thing from User.  Audits hasn't changed its declaration or anything, but

```
  2.1.5 :021 >   user = User.first
    User Load (0.3ms)  SELECT `users`.* FROM `users` LIMIT 1
  => #<User id: 1, name: "Rock'n Roller">
  2.1.5 :022 > user.audits
    Audited::Adapters::ActiveRecord::Audit Load (0.7ms)  SELECT `audits`.* FROM `audits` WHERE `audits`.`auditable_id` = 1 AND `audits`.`auditable_type` = 'User' ORDER BY version
     ...
```

Looks good, no?

No.  Look right after that first equal sign in the SELECT. MySQL (tested with 5.5 and 5.6) apparently sees that equality comparison to 1 as roughly:

```
... WHERE CONVERT(`auditable_id`, SIGNED INTEGER) = 1 ...
```
That is, it has no idea that you meant `'1'`, not `1`. And since we're
effectively transforming every value in the table in order to make the
comparison, it _wonn't use an index on it_.

You can see that by comparing the EXPLAIN PLAN output for both of those
queries:
```
    mysql> explain SELECT `audits`.* FROM `audits` WHERE `audits`.`auditable_id` = 'ycyh5qwb4ubw' AND `audits`.`auditable_type` = 'Case' ORDER BY version
        -> ;
    +----+-------------+--------+------+-----------------+-----------------+---------+-------------+------+-------------+
    | id | select_type | table  | type | possible_keys   | key             | key_len | ref         | rows | Extra       |
    +----+-------------+--------+------+-----------------+-----------------+---------+-------------+------+-------------+
    |  1 | SIMPLE      | audits | ref  | auditable_index | auditable_index | 1536    | const,const |    1 | Using where |
    +----+-------------+--------+------+-----------------+-----------------+---------+-------------+------+-------------+
    1 row in set (0.00 sec)

    mysql> explain SELECT `audits`.* FROM `audits` WHERE `audits`.`auditable_id` = 1 AND `audits`.`auditable_type` = 'User' ORDER BY version
        -> ;
    +----+-------------+--------+------+-----------------+------+---------+------+------+-----------------------------+
    | id | select_type | table  | type | possible_keys   | key  | key_len | ref  | rows | Extra                       |
    +----+-------------+--------+------+-----------------+------+---------+------+------+-----------------------------+
    |  1 | SIMPLE      | audits | ALL  | auditable_index | NULL | NULL    | NULL |    2 | Using where; Using filesort |
    +----+-------------+--------+------+-----------------+------+---------+------+------+-----------------------------+
    1 row in set (0.00 sec)
```

In our situation, where we have millions of audit entries, that's a disaster.

I could argue that this is MySQL's fault: it should just raise an
exception since you used an integer constant where a string comparison
is required.  But I also think this is yet another case where Rails' conventional
thinking breaks down as soon as you do anything out of the ordinary.

## What are the alternatives?

### Be relational, not object-oriented

Mr. [Karwin](http://stackoverflow.com/users/20860/bill-karwin) suggests that the
whole approach of the gem is wrong, since it uses SQL in a non-relational
manner.  But finding an alternative is complicated: this isn't the only
polymorphic assocation on the Audited model. There are actually __three__:

* The ``auditable`` association to the object modified
* The ``associated`` association to an object that the auditable object was associated with (to make it easier to find all changes to an object that has many associations, e.g., all order changes for a customer).
* The ``user`` association to the party responsible for the change.

Since SQL itself never mixes meta-data with data, all of these are out-of-band
from the database's perspective: the associations only make sense to Rails
code, not to general SQL users, visualization systems, etc., and they impose
Rails conventions on any other user of the DBMS.  In our case, we don't care,
but that only makes sense in a pure Rails shop.

To see if we can use some SQL alternatives, we'll make the Case associated with a
User (the user may own many cases, the case is owned by its user). So both our models are ``audited``, and
the User model ``has_associated_audits``. See the example source on Github for more details.

Karwin suggests three alternatives:

* **Exclusive Arcs**: Add a column for each type, and get rid of the type columns.
That'd work if MySQL could add columns responsibly (i.e., without locking the
table and modifying every record), but MySQL (InnoDB) is just getting around to having
implemented that 1988-vintage Oracle feature (in 5.6 we can avoid the lock but not the
modifying every record). It might also require the
``audited`` gem to get a little bit nosier about the declaration of
associations in its audited models.  This still relies on non-SQL constraint
checking, since for the columns implementing each of the three associations,
there can be only one non-NULL value. But foreign keys can at least be
declared, and we still only have one audits table.
* **Reverse the Relationship**: Create a
table for each combination of types. This has the advantage of allowing
proper SQL constraint checking, since each foreign key can be declared at the
database level.
This is the least likely to be surprising of the alternatives, since it also avoids the use
of [NULL](http://www09.sigmod.org/sigmod/record/issues/0809/p23.grant.pdf).
On the other hand, if we want to maintain an order of
operation (`version`) across all the audits for a specific table, we have
to use a multi-table sequence of some sort (no relying on ``AUTO_INCREMENT``).
* **Concrete Supertable**: If I understand this proposal correctly,
the idea would be to create an 'auditables' table. Any creation of an auditable
object (say `cases`) would also create one tuple in `auditables`.  Then to get
the audits for my Case, I join through its `auditable` entry to get its audits.
I.e., ``:case belongs_to :auditable``, and ``:auditable has_many :audits``. But how
would those audits reference the other objects?

### Avoid the ORM

For the audits of integer-keyed models, replace any uses of the 'audits'
accessor with a method that gives Active record the ``id`` as a string.
Unfortunately, I couldn't find a way to patch ``audited`` to do this
(overriding the generated association accessor wasn't reliable: it'd work in
development and production, but not with rspec, for example).

This is the solution we used, though: we just used a different method name to
implement the access we needed.

### Fix the ORM

This turns out the be the best quick answer: In Rails 4.2, Arel generates
the correct query for the type of the column:

```
    rec@rec-x230: (master) ~/polymorphic_assoc$ rails c
    Loading development environment (Rails 4.2.1)
    2.1.5 :001 > kase = Case.first
      Case Load (0.7ms)  SELECT  `cases`.* FROM `cases`  ORDER BY `cases`.`identifier` ASC LIMIT 1
     => #<Case identifier: "ycyh5qwb4ubw", manufacturer: "anvil", user_id: 1>
    2.1.5 :002 > kase.audits
      Audited::Adapters::ActiveRecord::Audit Load (0.6ms)  SELECT `audits`.* FROM `audits` WHERE `audits`.`auditable_id` = 'ycyh5qwb4ubw' AND `audits`.`auditable_type` = 'Case'  ORDER BY `audits`.`version` ASC
     => #<ActiveRecord::Associations::CollectionProxy [#<Audited::Adapters::ActiveRecord::Audit id: 1, auditable_id: "ycyh5qwb4ubw", auditable_type: "Case", associated_id: 1, associated_type: "User", user_id: nil, user_type: nil, username: nil, action: "create", audited_changes: {"identifier"=>"ycyh5qwb4ubw", "manufacturer"=>"anvil", "user_id"=>1}, version: 1, comment: nil, remote_address: nil, created_at: "2015-06-02 01:50:34">]>
    2.1.5 :005 >   user = User.first
      User Load (0.4ms)  SELECT  `users`.* FROM `users`  ORDER BY `users`.`id` ASC LIMIT 1
     => #<User id: 1, name: "Rock'n Roller">
    2.1.5 :006 > user.audits
      Audited::Adapters::ActiveRecord::Audit Load (0.7ms)  SELECT `audits`.* FROM `audits` WHERE `audits`.`auditable_id` = '1' AND `audits`.`auditable_type` = 'User'  ORDER BY `audits`.`version` ASC
```

Unfortunately, we can't easily move the application we have problems with from Rails 3.2 to Rails 4.2.

### Don't treat audits as relational data at all

I've been completely avoiding a major issue with the ``audited`` gem: what
exactly does it record about a change in the Case model above?

Well, it records changes to its rows. For any update, it records a JSON
string representing a hash (dictionary), where for each field changed,
there is a key for that field, and the value is either a scalar
(for a change from NULL, including creation), or a two-entry array
identifying the previous and new values.  How "relational" is that? By
SQL's conventions, it's not relational at all. It's yet another melding
of meta-data with data.

So why use a relational DB to record it? One good reason: to avoid [two-phase
commit](http://en.wikipedia.org/wiki/Two-phase_commit_protocol) while still
maintaining consistency. That is, you want the every change audited, and you
want to be certain that every audit represents a real change.  As soon as you
use a second resource manager (message queue, document store, whatever), you
have to deal with making both durable in a single atomic action.

Other than that, I can't think of any good reason. But, uh, I'm in the privacy
business. Audits are important, so consistency is reason enough.

On the other hand, we can certainly treat the audits as a "big blob" of
append-only data, ETL it somewhere, and stop querying it in MySQL at all....

**Convention!**
