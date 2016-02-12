# Correção da sequência da numeração do ids de todas as tabelas

Quando fazemos um dump e depois o restore do banco de dados Postgresql, a numeração fica desordenada. Para corrigir isso, temos o seguinte script:

```
SELECT
  'SELECT SETVAL(' ||quote_literal(S.relname)||
  ', MAX(' ||quote_ident(C.attname)||
  ') ) FROM ' ||quote_ident(T.relname)|| ';'
FROM
  pg_class AS S
  ,pg_depend AS D
  ,pg_class AS T
  ,pg_attribute AS C
WHERE
  S.relkind = 'S'
  AND S.oid = D.objid
  AND D.refobjid = T.oid
  AND D.refobjid = C.attrelid
  AND D.refobjsubid = C.attnum
ORDER BY S.relname;
```

Para fazer isso numa tabela individual, vai um exemplo:

```
ALTER SEQUENCE product_id_seq RESTART WITH 100;
```

Só copiar, colar e rodar!
