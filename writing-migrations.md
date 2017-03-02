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

## 数据表操作

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

当操作 Table 对象时，Phinx 提供了一些操作来改变数据库。

如果你不清楚该使用什么操作，建议你使用 save 方法。它将自动识别插入或者更新操作，并将改变应用到数据库。

### 创建一个表

使用 Table 可以很简单的创建一个表，现在我们创建一个存储用户信息的表

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
        $users = $this->table('users');
        $users->addColumn('username', 'string', array('limit' => 20))
              ->addColumn('password', 'string', array('limit' => 40))
              ->addColumn('password_salt', 'string', array('limit' => 40))
              ->addColumn('email', 'string', array('limit' => 100))
              ->addColumn('first_name', 'string', array('limit' => 30))
              ->addColumn('last_name', 'string', array('limit' => 30))
              ->addColumn('created', 'datetime')
              ->addColumn('updated', 'datetime', array('null' => true))
              ->addIndex(array('username', 'email'), array('unique' => true))
              ->save();
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

字段使用 `addColumn()` 方法创建。并且用 `addIndex()` 方法将 username 和 email 设置为唯一。最后调用 `save()` 提交我们的改变。

> Phinx 会为每个表自动创建一个自增的主键字段 `id`

id 选项会自动创建一个唯一字段，`primary_key`_ 选项设置哪个字段为主键。 _`primary_key` 选项默认值是 `id` 。这2个选项可以设置为false。

如果要指定一个主键，你可以设置 `primary_key` 选项，关闭自动生成 `id` 选项，并使用2个字段定义为主键。

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
        $table = $this->table('followers', array('id' => false, 'primary_key' => array('user_id', 'follower_id')));
        $table->addColumn('user_id', 'integer')
              ->addColumn('follower_id', 'integer')
              ->addColumn('created', 'datetime')
              ->save();
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

单独设置 `primary_key`_ 选项并不能开启 _`AUTO_INCREMENT` 选项。如果想简单的改变主键名，我们只有覆盖 `id` 字段名即可。

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
        $table = $this->table('followers', array('id' => 'user_id'));
        $table->addColumn('follower_id', 'integer')
              ->addColumn('created', 'timestamp', array('default' => 'CURRENT_TIMESTAMP'))
              ->save();
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

另外，MySQL adapter 支持一下选项

| 选项 | 描述 |
| :--- | :--- |
| comment | 给表设置注释 |
| engine | 定义表的引擎（默认 InnoDB） |
| collation | 定义表的语言（默认 utf8-general-ci） |

## 字段类型

字段类型如下：

* biginteger
* binary
* boolean
* date
* datetime
* decimal
* float
* integer
* string
* text
* time
* timestamp
* uuid

另外，MySQL adapter 支持 `enum` 、`set` 、`blob` 和 `json` （`json` 需要 MySQL 5.7 或者更高）

Postgres adapter 支持 `smallint` 、`json` 、`jsonb` 和 `uuid` （需要 PostgresSQL 9.3 或者更高）

### 表是否存在

可以使用 `hasTable()` 判断表是否存在。

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
        $exists = $this->hasTable('users');
        if ($exists) {
            // do something
        }
    }

    /**
     * Migrate Down.
     */
    public function down()
    {

    }
}
```

### 删除表

可以用 `dropTable()` 方法删除表。这时可以在 \`down\(\)\` 方法中重新创建表，可以在回滚的时候恢复。

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
        $this->dropTable('users');
    }

    /**
     * Migrate Down.
     */
    public function down()
    {
        $users = $this->table('users');
        $users->addColumn('username', 'string', array('limit' => 20))
              ->addColumn('password', 'string', array('limit' => 40))
              ->addColumn('password_salt', 'string', array('limit' => 40))
              ->addColumn('email', 'string', array('limit' => 100))
              ->addColumn('first_name', 'string', array('limit' => 30))
              ->addColumn('last_name', 'string', array('limit' => 30))
              ->addColumn('created', 'datetime')
              ->addColumn('updated', 'datetime', array('null' => true))
              ->addIndex(array('username', 'email'), array('unique' => true))
              ->save();
    }
}
```

### 重命名表名

可以用 `rename()` 方法重命名表名。

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
        $table = $this->table('users');
        $table->rename('legacy_users');
    }

    /**
     * Migrate Down.
     */
    public function down()
    {
        $table = $this->table('legacy_users');
        $table->rename('users');
    }
}
```

## 字段操作

### 字段类型

字段类型如下：

* biginteger
* binary
* boolean
* date
* datetime
* decimal
* float
* integer
* string
* text
* time
* timestamp
* uuid

另外，MySQL adapter 支持 `enum` 、`set` 、`blob` 和 `json` （`json` 需要 MySQL 5.7 或者更高）

Postgres adapter 支持 `smallint` 、`json` 、`jsonb` 和 `uuid` （需要 PostgresSQL 9.3 或者更高）

### 字段选项

以下是有效的字段选项：

所有字段：

| 选项 | 描述 |
| :--- | :--- |
| limit | 为string设置最大长度 |
| length | limit 的别名 |
| default | 设置默认值 |
| null | 允许空 |
| after | 指定字段放置在哪个字段后面 |
| comment | 字段注释 |

`decimal` 类型字段：

| 选项 | 描述 |
| :--- | :--- |
| precision | 和 scale 组合设置精度 |
| scale | 和 precision 组合设置精度 |
| signed | 开启或关闭 unsigned 选项（仅适用于 MySQL） |

`enum` 和 `set` 类型字段：

| 选项 | 描述 |
| :--- | :--- |
| values | 用逗号分隔代表值 |

`integer` 和 `biginteger` 类型字段：

| 选项 | 描述 |
| :--- | :--- |
| identity | 开启或关闭自增长 |
| signed | 开启或关闭 unsigned 选项（仅适用于 MySQL） |

`timestamp` 类型字段：

| 选项 | 描述 |
| :--- | :--- |
| default | 设置默认值 （CURRENT\_TIMESTAMP） |
| update | 当数据更新时的触发动作 （CURRENT\_TIMESTAMP） |
| timezone | 开启或关闭 with time zone 选项 |

可以在标准使用 `addTimestamps()` 方法添加 `created_at`_ 和 `updated_at`_ 。方法支持自定义名字。

```
<?php

use Phinx\Migration\AbstractMigration;

class MyNewMigration extends AbstractMigration
{
    /**
     * Migrate Change.
     */
    public function change()
    {
        // Override the 'updated_at' column name with 'amended_at'.
        $table = $this->table('users')->addTimestamps(null, 'amended_at')->create();
    }
}
```

`boolean` 类型字段：

| 选项 | 描述 |
| :--- | :--- |
| signed | 开启或关闭 unsigned 选项（仅适用于 MySQL） |

`string` 和` text` 类型字段：

| 选项 | 描述 |
| :--- | :--- |
| collation | 设置字段的 collation （仅适用于 MySQL） |
| encoding | 设置字段的 encoding （仅适用于 MySQL） |

外键定义：

| 选项 | 描述 |
| :--- | :--- |
| update | 设置一个触发器当数据更新时 |
| delete | 设置一个触发器当数据删除时 |

你可以将一个或者多个选项到第三个选项参数数组中。

### Limit 选项 和 PostgreSQL





