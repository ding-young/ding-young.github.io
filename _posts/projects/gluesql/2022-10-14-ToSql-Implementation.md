---
title: Implement ToSql trait
author: ding-young
description: null
tags: []
featuredImage: null
img: null
categories: [projects, gluesql]
date: '2022-10-14'
---

## For what

AST is barely readable. Rust provides Debug trait that displays internal structure of structs, but it is quite complicated to print certain AST.

`example`

Below is an example of “simple” CREATE TABLE AS statement.

```sql
CREATE TABLE IF NOT EXISTS Foo AS VALUES(True)
```

It looks simple in SQL.. but when it comes to Gluesql AST..

```rust
"CREATE TABLE IF NOT EXISTS Foo AS VALUES(True)",
Statement::CreateTable {
    if_not_exists: true,
    name: "Foo".into(),
    columns: vec![],
    source: Some(Box::new(Query {
        body: SetExpr::Values(Values(vec![vec![Expr::Literal(AstLiteral::Boolean(
            true
        ))]])),
        order_by: vec![],
        limit: None,
        offset: None
    }))
}
```

## Implement ToSql for Statements

```rust
pub trait ToSql {
    fn to_sql(&self) -> String;
}
```

Trait `ToSql` requires implementation of `to_sql` function that returns string.

Almost all of the gluesql AST implements `ToSql` trait, so we can just call to_sql fn for struct field and then format strings.

`example`

```rust
Statement::CreateTable {
                if_not_exists,
                name,
                columns,
                source,
            } => match source {
                Some(query) => match if_not_exists {
                    true => format!("CREATE TABLE IF NOT EXISTS {name} AS {}", query.to_sql()),
                    false => format!("CREATE TABLE {name} AS {}", query.to_sql()),
                },
                None => {
                    let columns = columns
                        .iter()
                        .map(ToSql::to_sql)
                        .collect::<Vec<_>>()
                        .join(", ");
                    match if_not_exists {
                        true => format!("CREATE TABLE IF NOT EXISTS {name} ({columns})"),
                        false => format!("CREATE TABLE {name} ({columns})"),
                    }
                }
            },
```

## What I learned

After implementing to_sql fn for Statements, SQL syntax became much familiar to me.

Could also play with rust vector. Iterate over vector, apply to_sql fn with map method, and then collect.

## Related PR

### start

[https://github.com/gluesql/gluesql/pull/554](https://github.com/gluesql/gluesql/pull/554)

### my works

[https://github.com/gluesql/gluesql/pull/807](https://github.com/gluesql/gluesql/pull/807)

[https://github.com/gluesql/gluesql/pull/821](https://github.com/gluesql/gluesql/pull/821)

[https://github.com/gluesql/gluesql/pull/824](https://github.com/gluesql/gluesql/pull/824)

[https://github.com/gluesql/gluesql/pull/931](https://github.com/gluesql/gluesql/pull/931)
