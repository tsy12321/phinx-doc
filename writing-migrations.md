# 迁移脚本

Phinx 使用迁移脚本来管理数据库。 每个迁移脚本都是一个 PHP 类。首选使用 Phinx API 来写迁移脚本，但是纯 SQL 语句也是支持的。

## 创建迁移脚本

### 生成一个迁移脚本框架

让我们从创建一个新的 Phinx 迁移脚本开始。使用 `create` 命令：

```
$ php vendor/bin/phinx create MyNewMigration
```

这将创建一个新的迁移脚本，格式是 `YYYYMMDDHHMMSS_my_new_migration.php` ，前14个字符是当前的timestamp，精确到秒。

如果你指定了多个脚本路径，将会提示你选择哪一个。



Phinx 自动创建的迁移脚本框架有一个方法：

```
<?php

use Phinx\Migration\AbstractMigration;

class MyNewMigration extends AbstractMigration
{
    /**
     * Change Method.
     *
     * Write your reversible migrations using this method.
     *
     * More information on writing migrations is available here:
     * http://docs.phinx.org/en/latest/migrations.html#the-abstractmigration-class
     *
     * The following commands can be used in this method and Phinx will
     * automatically reverse them when rolling back:
     *
     *    createTable
     *    renameTable
     *    addColumn
     *    renameColumn
     *    addIndex
     *    addForeignKey
     *
     * Remember to call "create()" or "update()" and NOT "save()" when working
     * with the Table class.
     */
    public function change()
    {

    }
}
```



