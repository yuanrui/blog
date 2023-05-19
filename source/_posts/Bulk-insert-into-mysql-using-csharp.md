---
title: C#实现MySql批量导入
date: 2021-04-25 10:16:00
tags:  
  - .Net
  - Ado.Net
  - MySql
  - Sql
categories:
  - .Net
  - Ado.Net
toc: true
---
### 前言

随着数据库的存储规模的增长，需要选择合适的数据批量写入技术。在MySql中实现批量导入，是指一次性的在数据表中插入很多记录，一般用于解决数据库写入性能。本文主要介绍C#中实现批量导入的MySql数据库的几种方法。

### 环境准备

本文使用的MySql数据库版本是：5.7，数据库连接驱动采用[nuget.org](https://www.nuget.org/packages/MySql.Data/)的最新中间件MySql.Data，安装方式如下：

```powershell
Install-Package MySql.Data -Version 8.0.24
```

我们将通过详细的代码示例研究这些方法，首先我们定义用于测试的数据表结构：

```mysql
DROP TABLE IF EXISTS t_vehicle_pass_record;
CREATE TABLE t_vehicle_pass_record (
  id bigint NOT NULL AUTO_INCREMENT,
  plate_no varchar(10) NOT NULL DEFAULT '',
  plate_color varchar(6) NOT NULL DEFAULT '',
  pass_time datetime NOT NULL DEFAULT now(),
  equip_id varchar(20) NOT NULL,
  created_at datetime NOT NULL DEFAULT now(),
  PRIMARY KEY (id)
) ENGINE=InnoDB;
```

然后定义代码所引用的C#模型代码：

```C#
    public class VehiclePassModel
    {
        public Int64 Id { get; set; }

        public string PlateNO { get; set; }

        public string PlateColor { get; set; }

        public DateTime PassTime { get; set; }

        public string EquipId { get; set; }

        public DateTime CreatedAt { get; set; }
    }
```

### 循环执行命令

循环执行命令（MySqlCommand）是一种常规的方式，一般步骤是：创建连接->打开数据库连接->创建事务->创建命令->循环执行命令->提交事务->关闭连接，结束。小提示：循环执行命令时，需要将绑定的参数清空后重新绑定，避免抛出参数已经定义的异常。

```c#
        public static int DoInsertList1(IEnumerable<VehiclePassModel> list)
        {
            const string sql = "insert into t_vehicle_pass_record (id, plate_no, plate_color, pass_time, equip_id, created_at) values (?id, ?plate_no, ?plate_color, ?pass_time, ?equip_id, now());";
            var result = 0;
            var connStr = ConfigurationManager.ConnectionStrings["DefaultConnectionString"].ConnectionString;

            using (var conn = new MySqlConnection(connStr))
            {
                conn.Open();
                using (var tran = conn.BeginTransaction())
                {
                    using (var cmd = conn.CreateCommand())
                    {
                        cmd.CommandText = sql;

                        foreach (var model in list)
                        {
                            cmd.Parameters.Clear();
                            cmd.Parameters.AddWithValue("id", model.Id);
                            cmd.Parameters.AddWithValue("plate_no", model.PlateNO);
                            cmd.Parameters.AddWithValue("plate_color", model.PlateColor);
                            cmd.Parameters.AddWithValue("pass_time", model.PassTime);
                            cmd.Parameters.AddWithValue("equip_id", model.EquipId);

                            result += cmd.ExecuteNonQuery(); //执行sql语句
                        }
                    }

                    tran.Commit();
                }
            }

            return result;
        }
```

### 适配器批量更新

使用此方法执行插入语句，内部实现同样采用循环执行原理，不同的是适配器(MySqlDataAdapter)会根据UpdateBatchSize参数进行分组后循环执行。适配器包含4个执行命令：UpdateCommand、SelectCommand、InsertCommand、DeleteCommand，用于数据的更新、查询、插入、删除操作。批量插入时应当使用InsertCommand，同时不要给其他执行命令赋值，因为给其他执行命令赋值会造成不必要的执行。适配器包含一个UpdateBatchSize属性，用于将DataTable中的Rows拆分按照数量拆分成多个（Rows.Count / UpdateBatchSize）执行命令；UpdateBatchSize未设置或者小于1时，UpdateBatchSize等于DataTable行数，此时只有一个执行命令。

适配器的Update方法，提供多个方法重载，推荐使用传入DataTable类型，并设置表名称。传入DataSet类型时方法内部会转换为DataTable，并调用UpdateFromDataTable(DataTable dataTable, DataTableMapping tableMapping)方法，因此传入DataTable类型的方法更高效。

```c#
        public static int DoInsertList2(IEnumerable<VehiclePassModel> list)
        {
            const string sql = "insert into t_vehicle_pass_record (id, plate_no, plate_color, pass_time, equip_id, created_at) values (?id, ?plate_no, ?plate_color, ?pass_time, ?equip_id, now());";
            var result = 0;
            var connStr = ConfigurationManager.ConnectionStrings["DefaultConnectionString"].ConnectionString;

            var table = new DataTable("t_vehicle_pass_record");
            table.Columns.Add("id", typeof(Int64));
            table.Columns.Add("plate_no", typeof(string));
            table.Columns.Add("plate_color", typeof(string));
            table.Columns.Add("pass_time", typeof(DateTime));
            table.Columns.Add("equip_id", typeof(string));

            foreach (var model in list)
            {
                var row = table.NewRow();
                row["id"] = model.Id;
                row["plate_no"] = model.PlateNO;
                row["plate_color"] = model.PlateColor;
                row["pass_time"] = model.PassTime;
                row["equip_id"] = model.EquipId;

                table.Rows.Add(row);
            }

            using (var conn = new MySqlConnection(connStr))
            {
                conn.Open();
                using (var tran = conn.BeginTransaction())
                {
                    using (var cmd = conn.CreateCommand())
                    {
                        cmd.CommandText = sql;

                        using (var adapter = new MySqlDataAdapter())
                        {
                            adapter.InsertCommand = cmd;
                            
                            adapter.InsertCommand.Parameters.Add("?id", MySqlDbType.Int64, 21, "id");
                            adapter.InsertCommand.Parameters.Add("?plate_no", MySqlDbType.VarChar, 10, "plate_no");
                            adapter.InsertCommand.Parameters.Add("?plate_color", MySqlDbType.VarChar, 6, "plate_color");
                            adapter.InsertCommand.Parameters.Add("?pass_time", MySqlDbType.DateTime, 8, "pass_time");
                            adapter.InsertCommand.Parameters.Add("?equip_id", MySqlDbType.VarChar, 20, "equip_id");

                            adapter.InsertCommand.UpdatedRowSource = UpdateRowSource.None;

                            adapter.UpdateBatchSize = 1000;
                            result = adapter.Update(table); //执行sql语句
                        }
                    }

                    tran.Commit();
                }
            }

            return result;
        }
```

###  执行拼接SQL语句

MySql的[`INSERT ... VALUES`](https://dev.mysql.com/doc/refman/5.7/en/insert.html) 语法中提供插入多个值列表的功能。

```mysql
INSERT INTO tbl_name (a,b,c) VALUES(1,2,3),(4,5,6),(7,8,9);
```

此方法的核心在于拼接SQL语句，绑定参数变量列表，然后执行SQL.

```c#
        public static int DoInsertList3(IEnumerable<VehiclePassModel> list)
        {
            const string insertIntoClause = "insert into t_vehicle_pass_record (id, plate_no, plate_color, pass_time, equip_id, created_at) values ";
            var result = 0;
            var connStr = ConfigurationManager.ConnectionStrings["DefaultConnectionString"].ConnectionString;

            var valueClauses = new List<string>(list.Count());
            var index = 0;
            using (var conn = new MySqlConnection(connStr))
            {
                conn.Open();
                using (var tran = conn.BeginTransaction())
                {
                    using (var cmd = conn.CreateCommand())
                    {
                        cmd.Transaction = tran;

                        //foreach (var model in list)
                        //{
                        //    var suffix = Convert.ToString(index, 16);
                        //    var clause = $"(?a{suffix}, ?b{suffix}, ?c{suffix}, ?d{suffix}, ?e{suffix}, now())";
                        //    valueClauses.Add(clause);
                        //    cmd.Parameters.AddWithValue($"a{suffix}", model.Id);
                        //    cmd.Parameters.AddWithValue($"b{suffix}", model.PlateNO);
                        //    cmd.Parameters.AddWithValue($"c{suffix}", model.PlateColor);
                        //    cmd.Parameters.AddWithValue($"d{suffix}", model.PassTime);
                        //    cmd.Parameters.AddWithValue($"e{suffix}", model.EquipId);
                        //    index++;
                        //}

                        foreach (var model in list)
                        {
                            var clause = $"(?id_{index}, ?plate_no_{index}, ?plate_color_{index}, ?pass_time_{index}, ?equip_id_{index}, now())";
                            valueClauses.Add(clause);
                            cmd.Parameters.AddWithValue($"id_{index}", model.Id);
                            cmd.Parameters.AddWithValue($"plate_no_{index}", model.PlateNO);
                            cmd.Parameters.AddWithValue($"plate_color_{index}", model.PlateColor);
                            cmd.Parameters.AddWithValue($"pass_time_{index}", model.PassTime);
                            cmd.Parameters.AddWithValue($"equip_id_{index}", model.EquipId);
                            index++;
                        }

                        var sql = insertIntoClause + string.Join(",", valueClauses) + ";";
                        cmd.CommandText = sql;

                        Console.WriteLine(sql.Length);
                        Console.WriteLine(Encoding.UTF8.GetBytes(sql).Length);

                        result = cmd.ExecuteNonQuery(); //执行sql语句
                    }

                    tran.Commit();
                }
            }

            return result;
        }
```

###  使用MySqlBulkLoader

MySqlBulkLoader主要包装MySql的语法 [`LOAD DATA INFILE`](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)，提供将数据文件从本地或远程保存到数据库。使用参数详解：

1. Local表示要加载的文件是否在客户端本地，默认值为false（如果导入程序和数据库在同一台服务器，此参数可以设置为false），使用时应当设置为true；
2. TableName为待导入的数据表名，为必传参数；
3. FileName为导入的文本文件路径，导入的格式可以为后缀为：.txt、.csv的UTF-8文本文件。推荐使用csv格式，将FieldTerminator设置为","，LineTerminator设置为Environment.NewLine；
4. 导入的文本文件包含中文字符时，文本保存的格式应当设置为UTF-8，同时将CharacterSet属性设置为"utf8mb4";
5. NumberOfLinesToSkip表示跳过多少条文本开始导入，主要用于跳过表头设置。标准的csv文件格式带有一行表头，所以此处应当设置为1;
6. Columns用于指定待导入的数据字段，字段名称和数据表中字段名称保持一致，字段顺序和内容需一一对应。通过此参数可以设置只导入部分字段内容，未设置时，需要将文本文件设置为全表字段内容。

```c#
        public static int DoInsertList4(IEnumerable<VehiclePassModel> list)
        {
            var result = 0;
            var connStr = ConfigurationManager.ConnectionStrings["DefaultConnectionString"].ConnectionString;
            var path = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, Guid.NewGuid().ToString() + ".csv");
            var head = "id,plate_color,plate_no,pass_time,equip_id";
            using (var writer = File.CreateText(path))
            {
                writer.WriteLine(head);//注释掉此行后, loader.NumberOfLinesToSkip可以设置为0
                foreach (var item in list)
                {
                    var content = $"{item.Id},{item.PlateColor},{item.PlateNO},{item.PassTime.ToString("yyyy-MM-dd HH:mm:ss")},{item.EquipId}";
                    writer.WriteLine(content);
                }
                writer.Flush();
            }

            try
            {
                using (var conn = new MySqlConnection(connStr))
                {
                    conn.Open();
                    MySqlBulkLoader loader = new MySqlBulkLoader(conn);
                    loader.Local = true;
                    loader.FieldTerminator = ",";
                    loader.LineTerminator = Environment.NewLine;
                    loader.FileName = path;
                    loader.NumberOfLinesToSkip = 1;
                    loader.CharacterSet = "utf8mb4";

                    loader.TableName = "t_vehicle_pass_record";
                    loader.Columns.AddRange(head.Split(new[] { ',' }, StringSplitOptions.RemoveEmptyEntries));

                    result = loader.Load();
                }
            }
            finally
            {
                if (File.Exists(path))
                {
                    File.Delete(path);
                }
            }

            return result;
        }
```

###  总结

使用适配器更新时，第一次验证未设置UpdateBatchSize参数测试结果和循环执行差不多，设置为1000后，性能有着明显的提升。

在导入数量超过10w时，执行拼接SQL语句异常。

```
MySql.Data.MySqlClient.MySqlException (0x80004005): Packets larger than max_allowed_packet are not allowed.
   在 MySql.Data.MySqlClient.MySqlStream.SendPacket(MySqlPacket packet)
   在 MySql.Data.MySqlClient.NativeDriver.ExecutePacket(MySqlPacket packetToExecute)
   在 MySql.Data.MySqlClient.NativeDriver.SendQuery(MySqlPacket queryPacket)
   在 MySql.Data.MySqlClient.Driver.SendQuery(MySqlPacket p)
   在 MySql.Data.MySqlClient.Statement.ExecuteNext()
   在 MySql.Data.MySqlClient.PreparableStatement.Execute()
   在 MySql.Data.MySqlClient.MySqlCommand.ExecuteReader(CommandBehavior behavior)
   在 MySql.Data.MySqlClient.MySqlCommand.ExecuteNonQuery()
```

异常原因是客户端传输的数据包消息长度超过了4MB，解决方案是：修改数据库max_allowed_packet参数，或分批执行SQL，或修改SQL缩小语句长度和变量参数名称长度。

```C#
                        foreach (var model in list)
                        {
                            var suffix = Convert.ToString(index, 16);
                            var clause = $"(?a{suffix}, ?b{suffix}, ?c{suffix}, ?d{suffix}, ?e{suffix}, now())";
                            valueClauses.Add(clause);
                            cmd.Parameters.AddWithValue($"a{suffix}", model.Id);
                            cmd.Parameters.AddWithValue($"b{suffix}", model.PlateNO);
                            cmd.Parameters.AddWithValue($"c{suffix}", model.PlateColor);
                            cmd.Parameters.AddWithValue($"d{suffix}", model.PassTime);
                            cmd.Parameters.AddWithValue($"e{suffix}", model.EquipId);
                            index++;
                        }
```

实际运用时，还可以做更进一步的导入优化：

- 缩小SQL绑定变量的大小，减少网络传输的消耗；
- 确保待执行的SQL在85k以内避免产生内存大对象；
- 设置合适的批次执行的数量，以达到最优解；



批量导入的耗时是评价导入性能的参照标准之一，这里我们分别导入100、1000、10000、50000、10000条数据测试本地MySql数据库的平均耗时，供读者参考。

| 导入数量 | 循环执行命令 | 适配器批量更新 | 执行拼接SQL语句 | 使用MySqlBulkLoader |
| -------- | ------------ | -------------- | --------------- | ------------------- |
| 100      | 121ms        | 149ms          | 124ms           | 95ms                |
| 1000     | 509ms        | 430ms          | 296ms           | 210ms               |
| 10000    | 2.87s        | 1.74s          | 787ms           | 387ms               |
| 50000    | 14.21s       | 9.12s          | 2.92s           | 2.21s               |
| 100000   | 27.73s       | 27.77s         | 异常            | 3.45s               |

从测试结果来看排名：

1. MySqlBulkLoader，缺点是会生成临时文件，优点是速度最快性能最好，推荐使用；

2. 拼接SQL语句，性能次之，编码方便，数据量不大情况推荐使用。

3. 适配器批量更新，适配器的核心在于设置UpdateBatchSize参数，需手动调优，少量数据时使用；

4. 循环执行，简单方案，少量数据时使用；




------

参考信息：

https://referencesource.microsoft.com/#System.Data/fx/src/data/System/Data/Common/DbDataAdapter.cs

https://dev.mysql.com/doc/refman/5.7/en/insert.html

https://dev.mysql.com/doc/connector-net/en/connector-net-programming-bulk-loader.html

https://dev.mysql.com/doc/dev/connector-net/8.0/html/T_MySql_Data_MySqlClient_MySqlBulkLoader.htm

https://dev.mysql.com/doc/refman/5.7/en/replication-features-max-allowed-packet.html

https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_max_allowed_packet