---
layout: post
title: "When the Schema Breaks"
date: 2014-06-13 11:20:29 -0700
comments: true
categories: database, modeling
---
Like a lot of organizations, we make trade-offs between 'schema' oriented data
modeling and 'schema-free' models.

There are a ton of ways to do this: no-schema document stores
like MongoDB, key/value stores like Cassandra, key/value stores that also
provide 'table' abstractions (Berkeley DB), etc.

At the moment, we're using another popular and venerable technique: adding a property/value table to a traditional schema-oriented RDBMS.

But once you have one of these, you face another question: how do you decide to add a dynamic property, rather than a column?

One answer directly relates to database normalization theory:

> If two properties have to be treated together, they should be treated as fixed attributes (columns), not dynamic ones.

Another practical answer comes from the behavior of ORMs (particularly ActiveRecord) and how they handle side effects in the model:

> If the value of a property causes side-effects on the domain model for the entity, that property should be a fixed attribute (column).

For a complete discussion of these trade-offs, there's always [http://martinfowler.com/apsupp/properties.pdf]. But let's take
a quick example through to completion.

## An example

Let's say we have an entity Person, and a Person can have one Phone. For the sake of simplicity, we'll ignore the fact that people might be contacted by more than one phone number, and that a phone number might be shared by multiple people.

Given that simplification, let's just put the `phone_number` on the Person model.

Now, let's say we want to distinguish whether that phone number represents a mobile, work, or home number, primarily for presentational purposes.
Since it's just about presentation (and profile) information and rarely written, we can quickly pop that into the code using a dynamic property (say, property `phone_type`).

But what happens next?  Someone looks at the application and says "well, hey -- if they change the number, you have default the `phone_type` to `mobile` again". Now we're treating two properties together: the behavior of one changes the value of the other.

It'd be a lot nicer now if we just called that a "new phone", with appropriate default values for the two properties (in fact it'd be a *ton* easier if we hadn't left the phone number in the Person model, but let's ignore that for now). An ORM like Active Record handles default values very smoothly, and it's not too difficult to make a default property value dependent on a change in another property.

In principle, you can construct that behavior from the combination of the fixed attribute `phone_number` and the dynamic `phone_type`.  For example, you can put a save or validation hook on the ```Person``` model that looks up and changes the ```phone_type``` property. In Mongoid you could do it more directly, but it still requires a more convoluted syntax than just treating the two properties as being in the same subdocument. And in both cases you have to be careful about what the controller layer is trying to do: what if the transaction is updating both things? Your save callback now has to know the whole transaction.

From normalization theory, we have the old maxim: *all* the attributes should be functionally dependent on the *key properties* of the entity. ("The key, the whole key, and nothing but the key"). Make both properties columns. You still have to be careful about updates (since you're handling rules about a 'sub-entity' of Person), but at least you have direct visibility as to what is changing.

## And with side-effects?

Of course, there's always another story or iteration.  The next story might be "put a check box in for whether we can send them SMS messages at this number".

Again, there's the temptation to just attach a boolean ```phone_send_sms``` dynamic property to the model. No migration necessary, just add code.

But, of course, the follow-up happens:

* If we add a phone number that happens to be mobile *and* sms-able, we should send them a text "welcome" message
* If we change a phone number, and it is now mobile and sms-able, we should send them a similar message

Now we either end up with a complicated "controller" action (using the Rails definition of Controller), or we use model callbacks to implement the side-effect. Again, using fixed attributes massively simplifies the coding for the side-effects.

## In the end, adding columns is often better than using dynamic properties.

And when we finally get to that iteration that allows phone numbers to get their own table, it will simplify the migration as well -- and we'll get to rip out all the special case code that had to be written to deal with embedding the ```Phone``` entity in the ```Person``` model.

So help me Codd.
