---
title: Querying JSON data in Postgres
post_excerpt: Combining the flexibility of JSON with the robustness of a relational database in Postgres.
taxonomy:
  category:
    - knowledge
    - notes
 post_tag:
    - postgres
    - json
    - database
---

## The best of both worlds: Postgres and JSON

A very common use case in my own work of data engineering is to store semi-structured data in perhaps an intermediary table of a relational database. Since [Sectors](https://sectors.app) is built with Postgres, I have the flexibility to store JSON data in a column. This is particularly useful when you have data that doesn't fit neatly into a tabular structure, such as nested objects or arrays.

PostgreSQL, I should note, is not a document store like MongoDB or Couchbase. It is a relational database management system (RDBMS) with excellent support for JSON data types, allowing you to store and query semi-structured data even within a relational database.

This article will demonstrate that along with some specific examples of querying JSON data in Postgres.

## Querying JSON columns in Postgres

Supposed we have a table called `idx_company_report` cotaining a column `major_shareholders` that stores JSON data. Here's an example of what the data might look like:

```json
{
  "symbol": "TLKM",
  "company_name": "PT Telekomunikasi Indonesia Tbk",
  "major_shareholders": [
    {
      "name": "Republic of Indonesia",
      "ownership_percentage": 52.09
    },
    {
      "name": "Public",
      "ownership_percentage": 47.91
    }
  ]
}
```

To select rows from the idx_company_report view where the major_shareholders array contains an object with the value 'Republic of Indonesia', you can use the following SQL query:

```sql
SELECT
  symbol, company_name, major_shareholders
FROM
  idx_company_report
WHERE
  major_shareholders @> '[{"name": "Republic of Indonesia"}]';
```

This query uses the `@>` operator to check if the `major_shareholders` array contains an object with the specified name. You can adjust the query based on your specific requirements and the structure of your JSON data.

### Incorporating `OR` and `ILIKE` in your json queries

Sectors is the only place, as far as I know, where you can find a list of companies on the Indonesian Stock Exchange (IDX) with state ownership whether directly or indirectly, i.e. through another entity with state ownership. The state-owned companies in Indonesia are often referred to as '(persero)' which is short for "Badan Usaha Milik Negara yang berbentuk Perusahaan Perseroan". Based on [BUMN and PERSERO on IDX by Sectors](https://sectors.app/indonesia/bumn) there are 44 such companies that are either directly or indirectly owned by the Republic of Indonesia.

To gather this list of companies, we will have to modify the query slightly so that, in addition to 'Republic of Indonesia' any 'name' containing the string '(persero)' regardless of its position in the string should also get a match.

Here is our new query:

```sql
SELECT
  symbol, company_name, major_shareholders
FROM
  idx_company_report
WHERE
  major_shareholders @> '[{"name": "Republic of Indonesia"}]'
  OR major_shareholders::jsonb @> '[{"name": {"@>": "(persero)"}}]';
```

In this query, we use the `@>` operator to check if the `major_shareholders` array contains an object with the specified name. We also use the `::jsonb` cast to convert the column to a JSONB data type, which allows us to use the `@>` operator for pattern matching.

The result of this query converted to JSON would look something like this:

```sql
[
  ...
  {
    "symbol": "BBNI.JK",
    "company_name": "PT Bank Negara Indonesia (Persero) Tbk",
    "major_shareholders": [
      {
        "name": "Republic of Indonesia",
        "share_value": 105849774057500,
        "share_amount": 22378387750,
        "share_percentage": "0.6"
      },
      {
        "name": "Public",
        "share_value": 70419087578070,
        "share_amount": 14887756359,
        "share_percentage": "0.39917"
      },
      {
        "name": "Putrama Wahju Setyawan",
        "share_value": 18350157980,
        "share_amount": 3879526,
        "share_percentage": "0.0001"
      },
      {
        "name": "Royke Tumilaar",
        "share_value": 17297330930,
        "share_amount": 3656941,
        "share_percentage": "0.0001"
      },
     ...
    ]
  },
  {
    "symbol": "BBRI.JK",
    "company_name": "PT Bank Rakyat Indonesia (Persero) Tbk",
    "major_shareholders": [
      {
        "name": "Republic of Indonesia",
        "share_value": 336147773572920,
        "share_amount": 80610976876,
        "share_percentage": "0.53188"
      },
      {
        "name": "Public",
        "share_value": 291831312468900,
        "share_amount": 69983528170,
        "share_percentage": "0.46177"
      },
      {
        "name": "Treasury Stock",
        "share_value": 3794192511000,
        "share_amount": 909878300,
        "share_percentage": "0.006"
      },
      {
        "name": "Handayani",
        "share_value": 23943723000,
        "share_amount": 5741900,
        "share_percentage": "0.00004"
      },
    ...
    ]
  },
    ...
]
```

Sweet 🎉! Now for the most part, these boilerplate code above should serve the need for most of your JSON querying needs in Postgres.

You can mix and match the `@>` operator with other operators like `ILIKE` to perform more complex queries on your JSON data. The possibilities are endless, but anything beyond this might require a more tailored approach that -- lacking insight into your specific use case -- I can't provide here.

### Incorporating `JSONB_ARRAY_ELEMENTS` in your json queries

In cases where the `@>` operator does not suffice for your needs, you can use the `jsonb_array_elements` function along with a `WHERE` clause to filter based on the condition. Here’s how you can do it:

```sql
SELECT
  symbol, company_name, major_shareholders
FROM
  idx_company_report
WHERE
  EXISTS (
    SELECT
      1
    FROM
      JSONB_ARRAY_ELEMENTS(major_shareholders) AS shareholder
    WHERE
      shareholder ->> 'name' = 'Republic of Indonesia'
      OR shareholder ->> 'name' ILIKE '%pemerintah ri%'
  )
OR company_name ILIKE '%persero%';
```

This query checks if there exists any element in the `major_shareholders` array (aliased as `shareholder`) that matches either condition of:

1. The `name` field is equal to 'Republic of Indonesia'.
2. The `name` field contains the string 'pemerintah ri' (case-insensitive), regardless of its position in the string. "ri" is short for "Republic Indonesia".

In cases where there is a match, the `EXISTS` clause returns `TRUE`, and the row is included in the result set. Here, we also use `JSONB_ARRAY_ELEMENTS` to [expand the `major_shareholders` array into a set of JSON objects](https://www.postgresql.org/docs/9.5/functions-json.html), which allows us to filter based on the `name` field. The `ILIKE` operator is used for case-insensitive pattern matching.

![Query Execution BigQuery](/_images/bumn_persero_idx.png)

## Conclusion

With just a simple query, you can combine the robustness and predictability of a relational database with the flexibility and expressiveness of JSON data.

In the case of shareholders, a JSON data type feels natural because the number of shareholders vary from company to company, and the shareholding structure can be rather complex and nested. When we opt for a JSON data type, we can store the data in a way that reflects its natural structure, making it easier to work with and query.

I hope this article has given you a good starting point for working with JSON data in Postgres. I write about database optimizations and engineering here on Supertype, so if you'd like to see more content, here are some other articles you might find interesting:

- [Optimizing queries in Postgres and BigQuery](https://supertype.ai/notes/optimizing-postgres-bigquery/)
- [Optimizing full text search in Postgres and BigQuery](https://supertype.ai/notes/optimizing-postgres-bigquery/)
