.. _tutorial-schema:

Step 1: 数据库模式
=======================

首先我们要创建数据库模式。对于这个应用仅一张表就足够了，而且我们只想支持 SQLite ，所以很简单。
只要把下面的内容放入一个名为 `schema.sql` 的文件，文件置于刚才创建的 `flaskr` 文件夹中：

.. sourcecode:: sql

    drop table if exists entries;
    create table entries (
      id integer primary key autoincrement,
      title string not null,
      text string not null
    );

这个模式由一个称为 `entries` 的单表构成，在这个表中每行包含一个 `id` ，一个 `title` 和一个 `text`。
`id` 是一个自增的整数而且是主键，其余两个为非空的字符串。

继续浏览 :ref:`tutorial-setup` 。
