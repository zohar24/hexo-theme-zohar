---
title: PostgreSQL数据库迁移clickhouse数据库
top: false
author: zhaohuan
date: 2023-10-25 18:17:31
tags:
- 数据迁移
- clickhouse
- postgreSQL
categories:
- 数据库
---


## 1. 建表语句

### 1.1  导出表结构

通过数据库工具导出历史年份所有的表结构



### 1.2 提取目标表

导出所有的表结构中，一些表并不是我们需要的，需要提取出我们需要的表（明细表和代码表）

可通过正则将对应语句提取出来，不同文本工具可能有不同的换行符

```sh
# 明细表
CREATE.+mx".+(\n)(.+(\n))+;
CREATE.+mx".+(\r\n)(.+(\r\n))+;

# 代码表
CREATE.+"d_.+(\n)(.+(\n))+;
CREATE.+"d_.+(\r\n)(.+(\r\n))+;
```

如何建表语句没有drop语句，需给每张表添加上去

```sql
drop table if exists [database.]tablename;
```



### 1.3 转换字段类型

CK数据库和postgreSQL数据库的字段类型不一致，需做转化

```sql
varchar -> Nullable(String)
int -> Nullable(Int8)、 Nullable(Int32)
timestamp -> Nullable(String)
numeric(12) -> Nullable(Decimal(12,0))
……
```



### 1.4 添加数据库引擎

```sql
# 如：
ENGINE = MergeTree
ORDER BY tjq
SETTINGS index_granularity = 8192;
```



### 1.5 order字段处理

去掉order字段的Nullable。



### 1.6 验证建表语句



### 1.7 去掉不需要的表

```sql
select * from system.tables
where database = 'db_sftj_ods2017' and name not in(
	select name from system.tables
	where database = 'db_sftj_ods' and name like '%mx'
)

--查找出需要的表名后，生成drop语句
```



## 2. 查询语句

### 2.1 生成查询语句

在clickhouse执行以下脚本

```sql
select  database ||'.' ||table || '.sql' as name_,
'SELECT\n' || arrayStringConcat(groupArray(name), ',\n') || '\nFROM ' || database || '.' ||table  AS query  
from (
	select database, table, case when name = 'ay' then 'trim(ay) as ay' else name end as name from system.columns
	where database in ('db_sftj_ods2016','db_sftj_ods2017','db_sftj_ods2018')
	and table like '%_mx'
) t
group by database,table
```

### 2.2 导出到Excel中

### 2.3 将Excel每行的数据导出到SQL文件中

编写java程序，将Excel的第一列作为SQL文件名，第二列作为文本内容。

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileWriter;
import java.math.BigDecimal;

import org.apache.commons.lang3.StringUtils;
import org.apache.poi.hssf.usermodel.HSSFDateUtil;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellType;
import org.apache.poi.ss.usermodel.DateUtil;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

/**
 * Test
 *
 * @author zhaohuan
 * @description
 * @date 2023/10/25 10:24
 */
public class Test {


    private final static String EXCEL2003 = "xls";

    private final static String EXCEL2007 = "xlsx";


    public static void readExcel(String fileName, String path) {
        if (fileName == null || (!fileName.matches("^.+\\.(?i)(xls)$") && !fileName.matches("^.+\\.(?i)(xlsx)$"))) {
            System.out.println("错误");

        }
        File file = new File(path + File.separator + fileName);
        try {
            FileInputStream fileInputStream = new FileInputStream(file);
            try (Workbook workbook = StringUtils.endsWithIgnoreCase(fileName, EXCEL2007) ? new XSSFWorkbook(fileInputStream) :
                    (StringUtils.endsWithIgnoreCase(fileName, EXCEL2003) ? new HSSFWorkbook(fileInputStream) : null)) {
                if (workbook == null) {
                    System.out.println("没数据");
                    return;
                }
                // 默认读取第一个sheet
                Sheet sheet = workbook.getSheetAt(0);
                int total = sheet.getLastRowNum() - sheet.getFirstRowNum();
                if (total <= 0) {
                    System.out.println("没数据");
                    return;
                }
                // 处理从第二行开始
                for (int i = sheet.getFirstRowNum() + 1; i <= sheet.getLastRowNum(); i++) {
                    Row row = sheet.getRow(i);
                    if (row == null) {
                        continue;
                    }
                    String name = getCellValue(row.getCell(0), CellType.STRING);
                    String value = getCellValue(row.getCell(1), CellType.STRING);

                    // 目标文件
                    FileWriter writer = new FileWriter("D:\\business\\zg\\old\\" + name);
                    writer.write(value);
                    writer.close();

                }
            }
        } catch (Exception e) {
        }

    }


    private static String getCellValue(Cell cell, CellType type) {
        if (cell == null) {
            return StringUtils.EMPTY;
        }
        if (!type.equals(cell.getCellType())) {
            // 如果读到的单元格类型和期望不一致，需要改为期望的类型
            cell.setCellType(type);
        }
        switch (type) {
            case NUMERIC:
                return DateUtil.isCellDateFormatted(cell) ? HSSFDateUtil.getJavaDate(cell.getNumericCellValue()).toString()
                        : BigDecimal.valueOf(cell.getNumericCellValue()).toString();
            case STRING:
                return StringUtils.trimToEmpty(cell.getStringCellValue());
            case FORMULA:
                return StringUtils.trimToEmpty(cell.getCellFormula());
            case BLANK:
                return StringUtils.EMPTY;
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            case ERROR:
                return "ERROR";
            default:
                return StringUtils.trimToEmpty(cell.toString());
        }
    }

    public static void main(String[] args) {
        readExcel("test.xlsx", "C:\\Users\\hu\\Desktop");
    }
}
```



### 2.4 整理抽数任务

以代码表为例，执行SQL生成，将返回的结果放到solution的xml文件中。

```sql
select '<table><source_table>db_data2017.'||name ||'</source_table><truncate>1</truncate><compute>1</compute><target_table>'||database || '.' || name || '</target_table></table>' from system.tables 
where database = 'db_sftj_ods2017'
and name like 'd_%'
order by name;
```







