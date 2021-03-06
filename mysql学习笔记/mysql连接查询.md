### mysql连接查询笔记###
----

联接条件可在ROM或WHERE子句中指定，建议在FROM子句中指定联接条件。WHERE和HAVING子句也可以包含搜索条件，以进一步筛选联接条件所选的行。

#### 1、内联接

内联接（典型的联接运算，使用像 =或<>之类的比较运算符）。包括相等联接和自然联接。内联接使用比较运算符根据每个表共有的列的值匹配两个表中的行。例如，检索students和courses表中学生标识号相同的所有行。

#### 2、外联接

外联接可以是左向外联接、右向外联接或完整外部联接，在 FROM子句中指定外联接。

* 1、左向外联接

LEFT JOIN或LEFT OUTER JOIN，左向外联接的结果集包括LEFT JOIN子句中指定的左表的所有行，而不仅仅是联接列所匹配的行。如果左表的某行在右表中没有匹配行，则在相关联的结果集行中右表的所有选择列表列均为空值。

* 2、右向外联接

RIGHT JOIN或RIGHT OUTER JOIN,右向外联接是左向外联接的反向联接。将返回右表的所有行。如果右表的某行在左表中没有匹配行，则将为左表返回空值。

* 3、全联接

FULL JOIN或FULL OUTER JOIN，完整外部联接返回左表和右表中的所有行。当某行在另一个表中没有匹配行时，则另一个表的选择列表列包含空值。如果表之间有匹配行，则整个结果集行包含基表的数据值。
