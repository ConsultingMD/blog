---
layout: post
title: "Guarding an Open Door: Authorizing Your GraphQL Fields"
date: 2017-10-09 18:05:51 -0700
comments: true
author: "Etienne Tripier <etienne.tripier@grandrounds.com>"
categories: graphql authorization security
---
Can you imagine what it must be like to work security at the U.S. Mint?

On one hand, you're tasked with safeguarding one of the most juicy targets in the country. On the other, you have a third-grade class rubbing their hands all over the bullion chained up in the lobby.

Everybody walks in through the front door - thieves and students alike.

I thought about that problem recently in the context of our GraphQL schema. Here we have another open door, this time the controller endpoint that receives and processes the query string. The authentication happens early; the user is admitted without yet knowing what they're after.

Let's say our user makes a query that looks like this:

```
query {
  delayedJobs {
    createdAt
    failedAt
    handler
  }
}
```

When our server processes the query (using the [`graphql` gem](https://github.com/rmosolgo/graphql-ruby)), it executes [three steps](https://dev-blog.apollodata.com/graphql-explained-5844742f195e) in the following order:

**1. Parse**

The server constructs an abstract syntax tree (AST) representing the query and validates the query syntax.

**2. Validate**

The server validates the query path, ensuring that the hierarchy of requested fields is correct and that any required arguments are provided, among other checks. A [query analyzer](http://graphql-ruby.org/queries/analysis.html) visits each field *without* executing the field's resolver and collects metadata before returning a final result.

**3. Execute**

The server resolves each field, beginning with the query root and working its way down the AST until it hits a leaf or an `ExecutionError`.

In the example above, assuming our query syntax is properly formed, the server should quickly iterate through all three steps and return an array of `DelayedJob` objects, each with the `createdAt`, `failedAt`, and `handler` properties.

Of course, we've got a problem. It's perfectly reasonable for an administrator to request our `delayedJobs` - however, we don't want to share that information with just *anybody*. In fact, we'd like to prevent the majority of our users from even being able to access this field.

## Ahead-of-Time Authorization
Incidentally, access-level authorization lines up nicely with the second step in the query processing flow. We only need to know enough about the field to make a quick decision as to whether or not a user should be able to request it - we don't actually care about what the field returns (in fact, we don't even want the resolver to execute).

Let's see if we can use a query analyzer to prevent the field from surfacing:

```ruby
TestSchema = GraphQL::Schema.define do
  query_analyzer AuthorizationAnalyzer.new
end
```

We'll define our new `AuthorizationAnalyzer` using the template provided in the `graphql` gem documentation:

```ruby
class AuthorizationAnalyzer
  # Called before the visit.
  # Returns the initial value for `memo`
  def initial_value(query)
  end

  # This is like the `reduce` callback.
  # The return value is passed to the next call as `memo`
  def call(memo, visit_type, irep_node)
  end

  # Called when we're done with the whole visit.
  # The return value may be a GraphQL::AnalysisError (or an array of them).
  # Or, you can use this hook to write to a log, etc
  def final_value(memo)
  end
end
```

We're interested in two methods: `call`, which iterates over each node of the AST, and `final_value`, which we'll use to return an `AnalysisError` if the user requests an unauthorized field.

In our `call` method, we'll want to take a good look at the `irep_node` given as the third argument:

```ruby
def call(memo, visit_type, irep_node)
  irep_node.definition # nil
end
```

The first time we hit this method, our `irep_node` will be the `query` node (one of three [root types](http://graphql.org/learn/execution/#root-fields-resolvers) in our schema: `query`, `mutation`, and `subscription`). The second time, we'll get the `delayedJobs` node:

```ruby
def call(memo, visit_type, irep_node)
  irep_node.definition.name # "delayedJobs"
end
```

Now that we know which field the user is trying to get access to, we just need to surface the `current_user` object so we can check for the relevant permissions. That's as easy as attaching the `current_user` to the query context:

```ruby
# Called from the controller endpoint handling GraphQL requests
Graph::Schema.execute(
  query: "your_query_string_here",
  context: { current_user: current_user },
  operation_name: "your_operation_name_here",
  variables: { ... }
)
```

Then, we can call the `current_user` in our query analyzer:

```ruby
def call(memo, visit_type, irep_node)
  irep_node.query.context[:current_user]
end
```

Checking for the right permissions goes something like this:

```ruby
def call(memo, visit_type, irep_node)
  current_user = irep_node.query.context[:current_user]
  requested_node_name = irep_node.definition.name.to_sym
  ability = Ability.new(current_user) # Using CanCan
  memo[:unauthorized_nodes] ||= []
  memo[:unauthorized_nodes] << irep_node if ability.cannot?(:access, requested_node_name)
  memo
end
```

Finally, we can return an `AnalysisError` once query analysis has completed:

```ruby
def final_value(memo)
  unauthorized_node_names = memo[:unauthorized_nodes].map { |node| node.definition.name }
  GraphQL::AnalysisError.new("You do not have permission to access #{unauthorized_node_names.join(', ')}")
end
```

If we want to get fancy, we can build some [instrumentation](http://graphql-ruby.org/queries/instrumentation.html) to attach metadata to each field indicating which action and subject we'd like to authorize against.

**Key Takeaways:**

- Ahead-of-time authorization is used when the conditions for access do not depend on the values returned by each field

- Authorization happens after the query has been parsed, but before the query is executed. A query analyzer makes a good hook for authorization checks

- Ahead-of-time authorization usually takes two parameters: The query context and the metadata for each field

## Runtime Authorization
Let's go back to the third-grade class in the lobby. We're not worried if they're playing with the gold bar on display; in fact, it's encouraged (what else is there to do at the Mint?). We'd be a little more concerned if we found a third-grader in the vaults, though.

In that case, we can't forbid everyone from handling our gold. But we can be very clear about *which* ingots can be touched.

Let's take a look at a different query:

```
query {
  user(id: 123) {
    email
    name
  }
}
```

We can't apply our binary field-access authorization policy here; at least, not if we want to support certain features (like allowing a member to see their user information in a profile page, for example). We need a more fine-grained authorization strategy that takes into account the `user` object being requested.

It's clear that we won't be able to perform this authorization in the first two steps of the query process. We'll have to wait until the third step - execution - when we have access to the return values of the field resolvers.

The obvious solution, then, is to perform authorization in the resolver itself. Let's pretend the resolver in our `user` field looks like this:

```ruby
resolve ->(obj, args, ctx) {
  User.find(args[:id])
}
```

We can take the `User` instance and authorize against it before returning anything; if authorization fails, we can simply return `nil` instead.

```ruby
resolve ->(obj, args, ctx) {
  ability = Ability.new(ctx[:current_user]) # Using CanCan
  user = User.find(args[:id])
  return user if ability.can?(:read, user)
  nil
}
```

[By definition](http://graphql.org/learn/queries/#mutations), queries are free of side effects, so waiting until the end of the resolver to perform authorization should be safe (that is, no unintended changes are made to our server's state).

If we're authorizing a mutation, however, we need a different approach. Let's take a look at this mutation:

```
mutation {
  updateUser(id: 123, input: { name: "<script>alert('You got hacked!')</script>" }) {
    id
  }
}
```

If our resolver looks like this:

```ruby
resolve ->(obj, args, ctx) {
  ability = Ability.new(ctx[:current_user]) # Using CanCan
  user = User.find(args[:id])
  user.update_attributes(args[:input]) # "You got hacked!"
  return user if ability.can?(:update, user) # false
  nil
}
```

... someone is going to be very annoyed. Clearly, we need to authorize before we perform the update:

```ruby
resolve ->(obj, args, ctx) {
  ability = Ability.new(ctx[:current_user]) # Using CanCan
  user = User.find(args[:id])
  if ability.can?(:update, user) # false
    user.update_attributes(args[:input])
    user
  else
    nil
  end
}
```

With just a bit of work, we can parameterize this solution for reuse:

```ruby
# Performs runtime authorization when resolving GraphQL mutation fields.
class MutationAuthorization
  # @return [NilClass] if unauthorized
  # @return [Result] if authorized
  def call(obj, args, ctx)
    ability = ctx[:current_ability]
    subject = @subject.call(obj, args, ctx)
    return nil if ability.cannot?(@action, subject) # Early return if authorization fails
    @resolver.call(obj, args, ctx, subject)
  end

  # @param action [Symbol] the action to authorize on
  # @param subject [Proc, Symbol] the subject to call cannot? on
  # @param resolver [Proc] the resolver to be called
  def initialize(action, subject, resolver)
    @action = action
    @subject = subject
    @resolver = resolver
  end
end
```

Using our new class, our mutation resolver looks like this:

```ruby
resolve MutationAuthorization.new(
  :update,
  ->(obj, args, ctx) { User.find(args[:id]) },
  ->(obj, args, ctx, subject) {
    subject.update_attributes(args[:input]) if subject.present?
  }
)
```

Now we're covered for resource-level mutations. We may not need this authorization strategy for *all* mutations, however. For example, consider the following mutation:

```ruby
mutation {
  createUser(input: { role: "admin" }) {
    id
  }
}
```

In this case, we might simply perform an ahead-of-time authorization check to see if the user has permission to create `admin`-level users.

**Key Takeaways:**

- Runtime authorization is used when the resolvers in each field must be executed in order to find the subjects to be authorized on

- Runtime authorization is performed anywhere in the resolver body

- Authorization *must* be performed before stateful changes are made

## Persisted Queries
Why do we even need to watch those third-graders so carefully? Couldn't we just tell them exactly where to go, and trust the teacher to enforce that?

There is a similar concept in GraphQL: [persisted queries](http://graphql-ruby.org/operation_store/overview#what-are-persisted-queries). Persisted queries are immutable, and their fields cannot be modified; clients reference these queries by ID.

Instead of sending this:

```javascript
post({
  query: "query { user(id: 123) { name } }",
});
```

... clients referencing persisted queries would send this:

```javascript
post({
  operationId: "yourQueryIdHere",
});
```

At runtime, the server uses the `operationId` to lookup the query string in the relevant table. Once the query is found, it is given to the query processor and the result is returned to the client.

There are some key benefits to this strategy:

- It is easier to authorize the entire query than it is to authorize individual fields. For example, consider the following query:

```
query {
  user(id: 123) {
    hashedId
    name
  }
}
```

We may have different, less restrictive permissions for the `hashedId` field. `name` is confidential information, but `hashedId` doesn't reveal anything personal about the user (at least on its own).

What if we had multiple fields that we wanted to restrict more thoroughly? What if we decided to reveal them only to `admin`-level users? We could store all personal fields in a persisted query:

```
# operationId "1234abcdef"
query {
  user(id: $id) {
    address
    email
    name
    ...otherPersonalFields
  }
}
```

We can refuse to perform the query (in a query analyzer, say) if we find that the user does not have an `admin`-level role. If we allowed arbitrary queries, we would have to authorize each field individually, greatly increasing the potential area for error (making multiple authorization checks versus a single check).

- Persisted queries help prevent complicated queries from exhausting your resources. An attacker may be able to bring your server to a crawl with a deeply-nested query

- Referencing a query by ID saves a bit of bandwith versus sending the entire query string in a request

That's it! Persisted queries can be an invaluable tool for creating a tightly-controlled API. [Facebook](https://dev-blog.apollodata.com/5-benefits-of-static-graphql-queries-b7fa90b0b69a), for example, references all of its queries by ID, guaranteeing a high degree of security and supervision for its various APIs.

## Conclusion
Authorization always requires a good deal of forethought and careful planning. This is especially true when creating a GraphQL API - because the request vector is harder to predict, each field's permissions must be considered across multiple contexts. Good understanding of the various authorization strategies is critical.

If nothing else, please remember the analogy of the third-grade class at the Mint; if you always expect honesty from a third-grader, you may be encouraging a budding kleptomaniac.
