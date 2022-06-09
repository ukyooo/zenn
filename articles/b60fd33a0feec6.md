---
title: "MySQL JSON カラム内の配列に含まれる要素を検索する方法"
emoji: "🎉"
type: "tech"
topics: ["JSON", "MySQL"]
published: true
---
例えば、

```sql
CREATE TABLE IF NOT EXISTS `user` (
    `id`    BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `data`  JSON,
    PRIMARY KEY (`id`)
) ;
```

上記のようなスキーマのテーブルに、

```sql
INSERT INTO user SET data = '{ "skill": { "tags": ["Python", "JavaScript", "Java"] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": [          "JavaScript", "Java"] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": ["Python",               "Java"] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": ["Python", "JavaScript"        ] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": ["Python"                      ] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": [          "JavaScript"        ] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": [                        "Java"] } }' ;
INSERT INTO user SET data = '{ "skill": { "tags": [                              ] } }' ;
```

という具合に情報を追加し、

```sql
SELECT * FROM user ;

+----+-------------------------------------------------------+
| id | data                                                  |
+----+-------------------------------------------------------+
|  1 | {"skill": {"tags": ["Python", "JavaScript", "Java"]}} |
|  2 | {"skill": {"tags": ["JavaScript", "Java"]}}           |
|  3 | {"skill": {"tags": ["Python", "Java"]}}               |
|  4 | {"skill": {"tags": ["Python", "JavaScript"]}}         |
|  5 | {"skill": {"tags": ["Python"]}}                       |
|  6 | {"skill": {"tags": ["JavaScript"]}}                   |
|  7 | {"skill": {"tags": ["Java"]}}                         |
|  8 | {"skill": {"tags": []}}                               |
+----+-------------------------------------------------------+
```

`user` の `skill.tags` に `Python` を含むレコードを抽出したい場合は、

```sql
SELECT * FROM user WHERE JSON_CONTAINS(data, '"Python"', '$.skill.tags') ;

+----+-------------------------------------------------------+
| id | data                                                  |
+----+-------------------------------------------------------+
|  1 | {"skill": {"tags": ["Python", "JavaScript", "Java"]}} |
|  3 | {"skill": {"tags": ["Python", "Java"]}}               |
|  4 | {"skill": {"tags": ["Python", "JavaScript"]}}         |
|  5 | {"skill": {"tags": ["Python"]}}                       |
+----+-------------------------------------------------------+
```

上記のようなイメージで `JSON_CONTAINS()` を使用することで抽出ができる。

配列の形で指定しても、

```sql
SELECT * FROM user WHERE JSON_CONTAINS(data, '["Python"]', '$.skill.tags') ;

+----+-------------------------------------------------------+
| id | data                                                  |
+----+-------------------------------------------------------+
|  1 | {"skill": {"tags": ["Python", "JavaScript", "Java"]}} |
|  3 | {"skill": {"tags": ["Python", "Java"]}}               |
|  4 | {"skill": {"tags": ["Python", "JavaScript"]}}         |
|  5 | {"skill": {"tags": ["Python"]}}                       |
+----+-------------------------------------------------------+
```

上記のように同じ結果となり、また、指定した配列の要素を複数にした場合、

```sql
SELECT * FROM user WHERE JSON_CONTAINS(data, '["Python", "JavaScript"]', '$.skill.tags') ;

+----+-------------------------------------------------------+
| id | data                                                  |
+----+-------------------------------------------------------+
|  1 | {"skill": {"tags": ["Python", "JavaScript", "Java"]}} |
|  4 | {"skill": {"tags": ["Python", "JavaScript"]}}         |
+----+-------------------------------------------------------+
```

```sql
SELECT * FROM user WHERE JSON_CONTAINS(data, '["Python", "Java"]', '$.skill.tags') ;

+----+-------------------------------------------------------+
| id | data                                                  |
+----+-------------------------------------------------------+
|  1 | {"skill": {"tags": ["Python", "JavaScript", "Java"]}} |
|  3 | {"skill": {"tags": ["Python", "Java"]}}               |
+----+-------------------------------------------------------+
```

`AND` 検索となる。

もちろん、下記のイメージで `OR` 検索も可能。

```sql
SELECT * FROM user
  WHERE JSON_CONTAINS(data, '"Python"',     '$.skill.tags')
    OR  JSON_CONTAINS(data, '"JavaScript"', '$.skill.tags') ;

+----+-------------------------------------------------------+
| id | data                                                  |
+----+-------------------------------------------------------+
|  1 | {"skill": {"tags": ["Python", "JavaScript", "Java"]}} |
|  2 | {"skill": {"tags": ["JavaScript", "Java"]}}           |
|  3 | {"skill": {"tags": ["Python", "Java"]}}               |
|  4 | {"skill": {"tags": ["Python", "JavaScript"]}}         |
|  5 | {"skill": {"tags": ["Python"]}}                       |
|  6 | {"skill": {"tags": ["JavaScript"]}}                   |
+----+-------------------------------------------------------+
```
