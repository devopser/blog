---
date: 2015-12-10 03:46:31.000Z
layout: post
title: mysql父子递归查询
tags: mysql
categories: mysql
---
```python
#父子表
CREATE TABLE `treeNodes` (  `id` int(11) NOT NULL AUTO_INCREMENT,  `nodename` varchar(40) DEFAULT NULL,  `pid` int(11) DEFAULT NULL,  PRIMARY KEY (`id`)) ENGINE=MyISAM AUTO_INCREMENT=16 DEFAULT CHARSET=utf8

createChildLst
@parm: IN rootId INT,IN nDepth INT
BEGIN      DECLARE done INT DEFAULT 0;      DECLARE b INT;      DECLARE cur1 CURSOR FOR SELECT id FROM treeNodes where pid=rootId;      DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;          insert into tmpLst values (null,rootId,nDepth);          OPEN cur1;          FETCH cur1 INTO b;      WHILE done=0 DO              CALL createChildLst(b,nDepth+1);              FETCH cur1 INTO b;      END WHILE;          CLOSE cur1;
END

showChildLst 获取某个ID下的所有子ID（递归）
@parm:IN rootId INT
BEGIN     CREATE TEMPORARY TABLE IF NOT EXISTS tmpLst        (sno int primary key auto_increment,id int,depth int);      DELETE FROM tmpLst;      set max_sp_recursion_depth=12;      CALL createChildLst(rootId,0);      select idd,group_concat(id)as ids from        (select '0' as idd,cast(tmpLst.id as char(5)) as id from tmpLst,treeNodes where tmpLst.id=treeNodes.id and tmpLst.id<> rootId order by tmpLst.sno       )a where id<>1 group by idd;     END

showtrees 树型结构展示各层级关系
@parm: IN rootid INT
BEGIN DECLARE Level int ; drop TABLE IF EXISTS tmpLst; CREATE TABLE tmpLst (  id int,  nLevel int,  sCort varchar(8000) );  Set Level=0 ; INSERT into tmpLst SELECT id,Level,ID FROM treeNodes WHERE PID=rootid; WHILE ROW_COUNT()>0 DO  SET Level=Level+1 ;  INSERT into tmpLst    SELECT A.ID,Level,concat(B.sCort,A.ID) FROM treeNodes A,tmpLst B     WHERE  A.PID=B.ID AND B.nLevel=Level-1  ; END WHILE;  END
```




 

