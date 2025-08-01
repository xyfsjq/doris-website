---
{
    "title": "columns",
    "language": "en"
}
---

## Overview

View all column information.

## Database


`information_schema`


## Table Information

| Column Name              | Type          | Description                                                  |
| ------------------------ | ------------- | ------------------------------------------------------------ |
| TABLE_CATALOG            | varchar(512)  | Catalog name                                                 |
| TABLE_SCHEMA             | varchar(64)   | Database name                                                |
| TABLE_NAME               | varchar(64)   | Table name                                                   |
| COLUMN_NAME              | varchar(64)   | Column name                                                  |
| ORDINAL_POSITION         | bigint        | The position of the column in the table                      |
| COLUMN_DEFAULT           | varchar(1024) | Default value of the column                                  |
| IS_NULLABLE              | varchar(3)    | Whether NULL is allowed                                      |
| DATA_TYPE                | varchar(64)   | Data type                                                    |
| CHARACTER_MAXIMUM_LENGTH | bigint        | Maximum number of characters allowed for character types     |
| CHARACTER_OCTET_LENGTH   | bigint        | Maximum number of bytes allowed for character types          |
| NUMERIC_PRECISION        | bigint        | Precision for numeric types                                  |
| NUMERIC_SCALE            | bigint        | Scale for numeric types                                      |
| DATETIME_PRECISION       | bigint        | Precision for datetime types                                 |
| CHARACTER_SET_NAME       | varchar(32)   | Character set name for character types, always NULL          |
| COLLATION_NAME           | varchar(32)   | Collation algorithm name for character types, always NULL    |
| COLUMN_TYPE              | varchar(32)   | Column type                                                  |
| COLUMN_KEY               | varchar(3)    | If 'UNI', it indicates that the column is a Unique Key column |
| EXTRA                    | varchar(27)   | Additional information about the column, including whether it is an auto-increment column, a generated column, etc. |
| PRIVILEGES               | varchar(80)   | Always empty                                                 |
| COLUMN_COMMENT           | varchar(255)  | Comment information for the column                           |
| COLUMN_SIZE              | bigint        | Width of the column                                          |
| DECIMAL_DIGITS           | bigint        | Number of decimal places for numeric types                   |
| GENERATION_EXPRESSION    | varchar(64)   | Always NULL                                                  |
| SRS_ID                   | bigint        | Always NULL                                                  |