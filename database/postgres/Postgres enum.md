There is an enum type in Postgres:
```create type myenum as enum('foo', 'bar');```

There are pros and cons related to using it vs a varchar or an integer field. Mostly pros imho.

In terms of size, it's stored as an `oid`, so `int32` type. This makes it smaller than a varchar populated with typical values (e.g. `'draft'`, `'published'`, `'pending'`, `'completed'`, whatever your enum is about), and the same size as an `int` type. If you've very few values, a `smallint` / `int16` will be admittedly be smaller. Some of your performance change will come from there (smaller vs larger field, i.e. mostly negligible).

Validation is possible in each case, be it through the built-in catalog lookup for the `enum`, or a check constraint or a foreign key for a `varchar` or an `int`. Some of your performance change will come from there, and it'll probably not be worth your time either.

Another benefit of the enum type, is that it is ordered. In the above example, `'foo'::myenum < 'bar'::myenum'`, making it possible to `order by enumcol`. To achieve the same using a `varchar` or an `int`, you'll need a separate table with a `sortidx` column or something... In this case, the enum can yield an enormous benefit if you ever want to order by your enum's values. This brings us to (imho) the only gotcha, which is related to how the enum type is stored in the catalog...

Internally, each enum's value carries an `oid`, and the latter are stored as is within the table. So it's technically an int32. When you create the enum type, its values are stored in the correct order within the catalog. In the above example, `'foo'` would have an `oid` lower than `'bar'`. This makes it very efficient for Postgres to order by an enum's value, since it amounts to sorting `int32` values.

When you `ALTER` your enum, however, you may end up in a situation where you change that order. For instance, imagine you alter the above enum in such a way that `myenum` is now (`'foo', 'baz', 'bar'`). For reasons tied to efficiency, Postgres does not assign a new `oid` for existing values and rewrite the tables that use them, let alone invalidate cached query plans that use them. What it does instead, is populate a separate field in the the `pg_catalog`, so as to make it yield the correct sort order. From that point forward, ordering by the enum field requires an extra lookup, which de facto amounts to joining the table with a separate values table that carries a `sortidx` field -- much like you would do with a `varchar` or an `int` if you ever wanted to sort them.

This is usually fine and perfectly acceptable. Occasionally, it's not. When not there is a solution: alter the tables with the enum type, and change their values to varchar. Also locate and adjust functions and triggers that make use of it as you do. Then drop the type entirely, and then recreate it to get fresh oid values. And finally alter the tables back to where they were, and readjust the functions and triggers. Not trivial, but certainly feasible.

https://stackoverflow.com/questions/16392737/in-postgres-is-it-performance-critical-to-define-low-cardinality-column-as-int