## storage_engine

***core function***:  read and write the data in memory and Disc by its execution.

*HOW does InnoDB perform to write data?*

**First**,Mysql optimize the SQL words.

**Second**,Mysql send the optimized execution plan to storage_engine.

**Then**,the execution in storage_engine will run the commands,because InnoDB engine knows that the speed of write and read in memory is larger than whom in Disc.InnoDB engine only operator the data in memory.

> The district is named *Buffer Pool*.InnoDB are supposed to  insert or update the data into the Buffer Pool. 
>
> Of course, In order to support rollback operation , InnoDB record the old data on *undo.Log* in Disc.

**After** writting data in the Buffer Pool in memory,InnoDB engine runs its threads which can read the data need update from memory in some occasions(特定时机)and write in Disc.

![](F:\幕布代码位置\图片库\InnoDB写入流程.png)

But this Architecture has a deadly problem:Turn off ! If I have no electricity,the data which haven't written in Disc will lose totally.

In order to solve the problem, InnoDB use a redo Architecture,when updating, InnoDB will write the 'information need update' in the other memory district:*Redo Log Buffer*.And then,flush it into Disc.

InnoDB provide some flush strategy about Redo Log.

```sql
-- Redo log 刷盘策略
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
SET GLOBAL innodb_flush_log_at_trx_commit = 1;
```

***Strategy 1***

InnoDB write the 'information need update' in the Redo Log Buffer in the memory Before commiting affairs. Meanwhile, write it into Page cache ,and flush it into Disc.

![](F:\幕布代码位置\图片库\刷盘策略1.png)

***Strategy 0*** 

Only write in the Redo Log Buffer,the next work runs every second.

***Strategy 2*** 

write in Redo Log Buffer and Page cache,then flush Disc every second.



The InnoDB restarts,give the priority to the redo Log.

![](F:\幕布代码位置\图片库\redo Architecture.png)

BinLog can provide lots of functions ,including ***changing history search,dataBase backup and restore, host servant copy.***When writting Redo Logs,flush the BinLog.Mysql will tell the Redo Log the information '已提交',Redo Log also write 'commit'.

![](F:\幕布代码位置\图片库\BinLog.png)