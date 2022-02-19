## Faceted Search

A faced search on web usually requires showing aggregate counts for each facet. e.g. count of each category or brand in case of product search etc.

`GROUPING SETS` makes most sense here since we have same data set but grouped in multiple ways e.g.

```
SELECT count_type,
  CASE count_type
      WHEN 'category' THEN category
      WHEN 'brand' THEN brand
      ELSE NULL
  END AS count_group, count
FROM (
  SELECT 
  COALESCE(
      CASE GROUPING (categroy) WHEN 0 THEN 'categroy' ELSE NULL END,
      CASE GROUPING (brand) WHEN 0 THEN 'brand' ELSE NULL END,
      'count_filtered'
  ) AS count_type,
  category, brand, COUNT(*) AS count
  WHERE ...
  GROUP BY GROUPING SETS ( (category), (brand), ())
) AS statistics
```

To opitmize such aggregate queries, index only scan can be achieved by creating an index on all grouping columns i.e. `index_catgeory_brand` above.
Avoid joins in such aggregates and instead prefer `UNION`

### Trigram Search

Postgres trigram allows to build autosuggest search. Create a `GIN` index for the columsn involved.

```
CREATE INDEX IF NOT EXISTS search_idx ON product USING GIN (name gin_trgm_ops, category gin_trgm_ops, brand gin_trgm_ops);

select name, categroy, brand
      greatest(
                 word_similarity('reebok', name),
                 word_similarity('reebok', category),
                 word_similarity('reebok', brand)
               ) sml
FROM product
WHERE
   (
          'reebok' <% name
       OR 'reebok' <% category
       OR 'pizza' <% brand
   )
ORDER BY sml desc limit 10;
```

