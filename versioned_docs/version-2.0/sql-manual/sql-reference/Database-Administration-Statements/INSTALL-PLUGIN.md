---
{
    "title": "INSTALL-PLUGIN",
    "language": "en"
}
---

## INSTALL-PLUGIN

### Name

INSTALL PLUGIN

### Description

This statement is used to install a plugin.

grammar:

```sql
INSTALL PLUGIN FROM [source] [PROPERTIES ("key"="value", ...)]
```

source supports three types:

1. An absolute path to a zip file.
2. An absolute path to a plugin directory.
3. Point to a zip file download path with http or https protocol

### Example

1. Install a local zip file plugin:

    ```sql
    INSTALL PLUGIN FROM "/home/users/doris/auditdemo.zip";
    ```

2. Install the plugin in a local directory:

    ```sql
    INSTALL PLUGIN FROM "/home/users/doris/auditdemo/";
    ```

3. Download and install a plugin:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip";
    ```

    Note than an md5 file with the same name as the `.zip` file needs to be placed, such as `http://mywebsite.com/plugin.zip.md5` . 
    The content is the MD5 value of the .zip file.

4. Download and install a plugin, and set the md5sum value of the zip file at the same time:

    ```sql
    INSTALL PLUGIN FROM "http://mywebsite.com/plugin.zip" PROPERTIES("md5sum" = "73877f6029216f4314d712086a146570");
    ```

### Keywords

    INSTALL, PLUGIN

### Best Practice

