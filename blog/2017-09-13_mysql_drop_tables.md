# mysql删除所有表，不删除数据库——删库跑路小脚本，用存储过程删除全部表

## 创建存储过程脚本

```mysql
DELIMITER $$
CREATE PROCEDURE `drop_all_tables`()
BEGIN
    DECLARE count INT;
    DECLARE tb VARCHAR(200);
    DECLARE dbname VARCHAR(200) DEFAULT DATABASE();
    DECLARE tbnames cursor FOR SELECT CONCAT('DROP TABLE `',dbname,'`.`',table_name,'`') FROM information_schema.tables WHERE table_schema = dbname;
    SELECT count(*) INTO count FROM information_schema.tables WHERE table_schema = dbname;
    OPEN tbnames;
    loop_i:LOOP
        IF count = 0 THEN 
            LEAVE loop_i;
        END IF;
        FETCH tbnames INTO tb;
        SET @tb = tb;
        PREPARE stmt FROM @tb;  
        EXECUTE stmt;  
        DEALLOCATE PREPARE stmt;
        SET count = count - 1;
	END LOOP;
    CLOSE tbnames;
END$$
DELIMITER ;
```

## 使用时调用存储过程
```mysql
call drop_all_tables();
```

## mysql命令行下效果：
```
mysql> use test;# 要创建在指定的数据库中才能被调用到
Database changed
mysql> DELIMITER $$  
mysql> CREATE PROCEDURE `drop_all_tables`()  
    -> BEGIN  
    ->     DECLARE count INT;  
    ->     DECLARE tb VARCHAR(200);  
    ->     DECLARE dbname VARCHAR(200) DEFAULT DATABASE();  
    ->     DECLARE tbnames cursor FOR SELECT CONCAT('DROP TABLE `',dbname,'`.`',table_name,'`') FROM information_schema.tables WHERE table_schema = dbname;  
    ->     SELECT count(*) INTO count FROM information_schema.tables WHERE table_schema = dbname;  
    ->     OPEN tbnames;  
    ->     loop_i:LOOP  
    ->         IF count = 0 THEN   
    ->             LEAVE loop_i;  
    ->         END IF;  
    ->         FETCH tbnames INTO tb;  
    ->         SET @tb = tb;  
    ->         PREPARE stmt FROM @tb;    
    ->         EXECUTE stmt;    
    ->         DEALLOCATE PREPARE stmt;  
    ->         SET count = count - 1;  
    ->     END LOOP;  
    ->     CLOSE tbnames;  
    -> END$$  
Query OK, 0 rows affected (0.00 sec)

mysql> DELIMITER ;
mysql> call drop_all_tables();
Query OK, 1 row affected (0.00 sec)
```
