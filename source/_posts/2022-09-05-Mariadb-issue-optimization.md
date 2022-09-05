---
title: Mariadb issue optimization
date: 2022-09-05 14:45:19
tags:
  - mariadb
  - issue
  - devops
categories:
  - work
---

## Background of the issue

In the project,need a complete migration of data from their Mariadb(10.5.7) to our AWS RDS Mariadb(10.6.7).

I did the following：

1. Use the mysqldump command to export sql file from their database
2. Import the sql file into our database
<!--more-->
## Issue Description

The time to run the following sql on our database(14.3s) is significantly longer than on theirs(0.002s).

```
SELECT ...
FROM `tbl_key_visual` `t`  
  LEFT OUTER JOIN `tbl_storage` `storage` ON (`t`.`storage_id`=`storage`.`storage_id`) 
  LEFT OUTER JOIN `tbl_storage` `relatedStorages` ON (`relatedStorages`.`parent_id`=`storage`.`storage_id`) 
  LEFT OUTER JOIN `tbl_image` `image` ON (`t`.`image_id`=`image`.`image_id`) 
  LEFT OUTER JOIN `tbl_image` `relatedImages` ON (`relatedImages`.`parent_image_id`=`image`.`image_id`)  
WHERE (kv_status='published') 
ORDER BY kv_order ASC;
```



explain  sql  on our new db:

![](https://raw.githubusercontent.com/sven0219/sven0219.github.io/static_files/blog/images/4cac1200-2532-4948-bcb1-f1229dd0b9fa.png)

explain  sql  on their old db:

![](https://raw.githubusercontent.com/sven0219/sven0219.github.io/static_files/blog/images/c453f8a0-11d0-4317-92a8-5c271ecc8c08.png)

When adding force index, you will get a similar query time

```
SELECT  ...
FROM `tbl_key_visual` `t`  
  LEFT OUTER JOIN `tbl_storage` `storage` ON (`t`.`storage_id`=`storage`.`storage_id`) 
  LEFT OUTER JOIN `tbl_storage` `relatedStorages` force index(index_parent_id) ON (`relatedStorages`.`parent_id`=`storage`.`storage_id`) 
  LEFT OUTER JOIN `tbl_image` `image` ON (`t`.`image_id`=`image`.`image_id`) 
  LEFT OUTER JOIN `tbl_image` `relatedImages` force index(parent_image_id) ON (`relatedImages`.`parent_image_id`=`image`.`image_id`) 
WHERE (kv_status='published') 
ORDER BY kv_order ASC;
```

## Cause of this issue

When the result obtained by explain shows Using join buffer  in extra means the join is unable to use an index, and it's doing the join the hard way.

## How to slove 

> When rows are added or deleted to an InnoDB [fulltext index](https://mariadb.com/kb/en/full-text-indexes/), the index is not immediately re-organized, as this can be an expensive operation. Change statistics are stored in a separate location . The fulltext index is only fully re-organized when an OPTIMIZE TABLE statement is run.

Run 

```
optimize table tablename
```

https://mariadb.com/kb/en/optimize-table/ 
