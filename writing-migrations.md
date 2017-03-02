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

所有迁移脚本都继承类 `AbstractMigration` 。这个类提供了迁移脚本的基本方法。迁移脚本可以通过很多方法改变你的数据库，比如创建新表、插入数据、增加索引和修改字段等等。

### Change 方法

Phinx 0.2.0 介绍了一个新功能-逆迁移（回滚）。现在这个功能成为了脚本的默认方法。在这个方法中，你只需要定义 `up` 的逻辑，Phinx 可以在回滚的时候自动识别出如何down。比如：

```
<?php

use Phinx\Migration\AbstractMigration;

class CreateUserLoginsTable extends AbstractMigration
{
    /**
     * Change Method.
     *
     * More information on this method is available here:
     * http://docs.phinx.org/en/latest/migrations.html#the-change-method
     *
     * Uncomment this method if you would like to use it.
     */
    public function change()
    {
        // create the table
        $table = $this->table('user_logins');
        $table->addColumn('user_id', 'integer')
              ->addColumn('created', 'datetime')
              ->create();
    }

    /**
     * Migrate Up.
     */
    public function up()
    {

    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

当执行这个迁移脚本，Phinx 将创建 `user_logins` 表，并且在回滚的时候自动删除该表。注意，当 `change` 方法存在的时候，`up` 和 `down` 方法会被自动忽略。如果你想用这些方法建议你创建另外一个迁移脚本。

> 当在 `change()` 方法中创建或者更新表的时候你必须使用 `create()` 或者 `update()` 方法。当使用 `save()` 方法时，Phinx无法识别是创建还是修改数据表。

Phinx只能回滚以下命令：

1. createTable

2. renameTable

3. addColumn

4. renameColumn

5. addIndex

6. addForeignKey

如果一个命令无法回滚，那 Phinx 会抛出 `IrreversibleMigrationException` 异常。

### Up 方法

up方法会在Phinx执行迁移命令时自动执行，并且该脚本之前并没有执行过。你应该将修改数据库的方法写在这个方法里。

### Down 方法

down方法会在Phinx执行回滚命令时自动执行，并且该脚本之前已经执行过迁移。你应该将回滚代码放在这个方法里。

## 执行查询

可以使用 `execute()` 和 `query()` 方法进行查询。`execute()` 方法会返回查询条数，`query()` 方法会返回结果。结果参照 [PDOStatement](http://php.net/manual/en/class.pdostatement.php)

```
<?php

use Phinx\Migration\AbstractMigration;

class MyNewMigration extends AbstractMigration
{
    /**
     * Migrate Up.
     */
    public function up()
    {
        // execute()
        $count = $this->execute('DELETE FROM users'); // returns the number of affected rows

        // query()
        $rows = $this->query('SELECT * FROM users'); // returns the result as an array
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

## 获取数据

有2个方法可以获取数据。 `fetchRow()` 方法可以返回一条数据， `fetchAll()` 可以返回多条数据。2个方法都可以使用原生 SQL 语句作为参数。

```
<?php

use Phinx\Migration\AbstractMigration;

class MyNewMigration extends AbstractMigration
{
    /**
     * Migrate Up.
     */
    public function up()
    {
        // fetch a user
        $row = $this->fetchRow('SELECT * FROM users');

        // fetch an array of messages
        $rows = $this->fetchAll('SELECT * FROM messages');
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

## 插入数据

Phinx 可以很简单的帮助你在表中插入数据。尽管这个功能也在 [seed](/database-seeding.md) 中实现了。你也可以在迁移脚本中实现插入数据。

```
<?php

use Phinx\Migration\AbstractMigration;

class NewStatus extends AbstractMigration
{
    /**
     * Migrate Up.
     */
    public function up()
    {
        // inserting only one row
        $singleRow = [
            'id'    => 1,
            'name'  => 'In Progress'
        ]

        $table = $this->table('status');
        $table->insert($singleRow);
        $table->saveData();

        // inserting multiple rows
        $rows = [
            [
              'id'    => 2,
              'name'  => 'Stopped'
            ],
            [
              'id'    => 3,
              'name'  => 'Queued'
            ]
        ];

        // this is a handy shortcut
        $this->insert('status', $rows);
    }

    /**
     * Migrate Down.
     */
    public function down()
    {
        $this->execute('DELETE FROM status');
    }
}
```

> 不能在 _change\(\)_ 方法中使用插入数据，只能在 _up\(\)_ 和 _down\(\)_ 中使用

## 操作数据表

### Table 对象

Table对象是Phinx中最有用的API之一。它可以让你方便的用 PHP 代码操作数据库。我们可以通过 `table()` 方法取到Table对象。

```
<?php

use Phinx\Migration\AbstractMigration;

class MyNewMigration extends AbstractMigration
{
    /**
     * Migrate Up.
     */
    public function up()
    {
        $table = $this->table('tableName');
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

接下来，你可以用 Table 对象提供的方法来操作修改数据库了

### Save 方法





