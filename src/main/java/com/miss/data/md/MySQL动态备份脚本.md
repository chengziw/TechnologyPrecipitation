## 1.热钱记录表-创建存储过程

存储过程名称：p_create_hot_asset_record_change_table

```sql
DELIMITER $$

USE `hardware_saas`$$

DROP PROCEDURE IF EXISTS `p_create_hot_asset_record_change_table`$$

CREATE DEFINER=`root`@`%` PROCEDURE `p_create_hot_asset_record_change_table`()
BEGIN
SET @sql_create_table_hardware_saas = CONCAT(  
'CREATE TABLE IF NOT EXISTS hot_asset_record_',YEARWEEK(DATE_ADD(CURDATE(), INTERVAL 7 DAY),1),  
" (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `asset_id` bigint(20) NOT NULL COMMENT '资金id',
  `account_id` bigint(20) NOT NULL COMMENT '账户ID',
  `account_type` tinyint(2) DEFAULT NULL COMMENT '账户类型 1、普通玩家',
  `business_id` varchar(64) DEFAULT NULL COMMENT '申请业务标识',
  `business_type` tinyint(2) DEFAULT NULL COMMENT '业务申请类型',
  `pre_money` decimal(40,10) DEFAULT NULL COMMENT '操作前冻结积分',
  `cur_money` decimal(40,10) DEFAULT NULL COMMENT '当前账户冻结积分',
  `peer_money` decimal(40,10) DEFAULT NULL COMMENT '可用账户积分',
  `trans_money` decimal(40,10) DEFAULT NULL COMMENT '可用账户积分',
  `remark` varchar(64) DEFAULT NULL COMMENT '备注',
  `remark_code` text COMMENT '备注编号',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `modify_time` datetime NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `operate_type` int(11) DEFAULT NULL COMMENT '操作类型',
  `record_type` int(11) DEFAULT NULL COMMENT '操作类型',
  `status` int(11) DEFAULT NULL COMMENT '状态',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1000000 DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci ");  
  
PREPARE sql_create_table_hardware_saas FROM @sql_create_table_hardware_saas;     
EXECUTE sql_create_table_hardware_saas;  
DEALLOCATE PREPARE sql_create_table_hardware_saas;
END$$

DELIMITER ;
```

## 2.冷钱记录表-创建存储过程

存储过程名称：p_create_cold_asset_record_change_table

```sql
DELIMITER $$

USE `hardware_saas`$$

DROP PROCEDURE IF EXISTS `p_create_cold_asset_record_change_table`$$

CREATE DEFINER=`root`@`%` PROCEDURE `p_create_cold_asset_record_change_table`()
BEGIN
SET @sql_create_table_hardware_saas = CONCAT(  
'CREATE TABLE IF NOT EXISTS cold_asset_record_',YEARWEEK(DATE_ADD(CURDATE(), INTERVAL 7 DAY),1),  
" (
   `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `asset_id` bigint(20) NOT NULL COMMENT '资金id',
  `account_id` bigint(20) NOT NULL COMMENT '账户ID',
  `account_type` tinyint(2) DEFAULT NULL COMMENT '账户类型 1、普通玩家',
  `business_id` varchar(64) DEFAULT NULL COMMENT '申请业务标识',
  `business_type` tinyint(2) DEFAULT NULL COMMENT '业务申请类型',
  `pre_money` decimal(40,10) DEFAULT NULL COMMENT '操作前冻结积分',
  `cur_money` decimal(40,10) DEFAULT NULL COMMENT '当前账户冻结积分',
  `peer_money` decimal(40,10) DEFAULT NULL COMMENT '可用账户积分',
  `trans_money` decimal(40,10) DEFAULT NULL COMMENT '可用账户积分',
  `remark` varchar(64) DEFAULT NULL COMMENT '备注',
  `remark_code` text COMMENT '备注编号',
  `create_time` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `modify_time` datetime NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `operate_type` int(11) DEFAULT NULL COMMENT '操作类型',
  `record_type` int(11) DEFAULT NULL COMMENT '操作类型',
  `status` int(11) DEFAULT NULL COMMENT '状态',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1000000 DEFAULT CHARSET=utf8 COLLATE=utf8_general_ci ");  
  
PREPARE sql_create_table_hardware_saas FROM @sql_create_table_hardware_saas;     
EXECUTE sql_create_table_hardware_saas;  
DEALLOCATE PREPARE sql_create_table_hardware_saas;
END$$

DELIMITER ;
```

## 3.创建热钱记录表新增事件

```sql
DELIMITER $$

CREATE DEFINER=`root`@`127.0.0.1` EVENT `hot_event` ON SCHEDULE EVERY 1 DAY STARTS '2020-05-28 10:00:00' ON COMPLETION NOT PRESERVE ENABLE DO CALL p_create_hot_asset_record_change_table()$$

DELIMITER ;
```

## 4.创建冷钱记录表新增事件

```sql
DELIMITER $$

CREATE DEFINER=`root`@`127.0.0.1` EVENT `cold_event` ON SCHEDULE EVERY 1 DAY STARTS '2020-05-28 10:00:00' ON COMPLETION NOT PRESERVE ENABLE DO CALL p_create_cold_asset_record_change_table()$$

DELIMITER ;
```

## 5.查看数据库变量---event_scheduler

```sql
SHOW VARIABLES LIKE 'event_scheduler';
```

## 6.更改变量

备注：如果步骤5查询的值为OFF，则进行该步骤，如果为ON，到此为止

配置文件的[mysqld]加上event_scheduler=ON，重启mysql

