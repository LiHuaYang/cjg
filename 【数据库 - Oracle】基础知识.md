
# Oracle 学习小结

## 导入表数据

```sql
insert into icbcreceipt4fingardtrans@msp_temp(ID, ACCOUNTINGFLAG, BATNO, BUSINESSTYPE, FLOWNO, PAYACCOUNT, RECEIPTCODE, RECEIPTDATE, CUSTOMERID) (select ID, ACCOUNTINGFLAG, BATNO, BUSINESSTYPE, FLOWNO, PAYACCOUNT, RECEIPTCODE, RECEIPTDATE, CUSTOMERID
 from icbcreceipt4fingardtrans where customerId='80808080617805f30161878959b06614')


insert into fingardtransactions@msp_temp(ID, ACCOUNTINGDATE, AMOUNT, AREACODE, BANKLOCATIONCODE, BATID, BATCHNO, CARDTYPE, CERTNUM, CERTTYPE, CREATETIME, CURRENCY, CUSTACCNAME, CUSTACCNUM, CUSTBANKCODE, CUSTBANKLOCATIONNAME, CUSTBANKNAME, CYCLEDATE, ENTERPRISEACCNAME, ENTERPRISEACCNUM, ENTERPRISEBANKCODE, ENTERPRISEBANKLOCATIONNAME, ENTERPRISEBANKNAME, ENTERPRISECODE, ENTERPRISENAME, FIELD1, FIELD2, INFO, INFOCODE, ISPRIVATE, ISURGENT, LASTIMPORTSEQ, LASTMODIFYTIME, MEMO, MOBILE, MONEYWAY, ORGCODE, PAYSTATE, PAYTYPE, PURPOSE, ROWVERSION, TRANSDATE, TRANSNO, TRANSWAY, URID, CUSTOMERID) ( select 
ID, ACCOUNTINGDATE, AMOUNT, AREACODE, BANKLOCATIONCODE, BATID, BATCHNO, CARDTYPE, CERTNUM, CERTTYPE, CREATETIME, CURRENCY, CUSTACCNAME, CUSTACCNUM, CUSTBANKCODE, CUSTBANKLOCATIONNAME, CUSTBANKNAME, CYCLEDATE, ENTERPRISEACCNAME, ENTERPRISEACCNUM, ENTERPRISEBANKCODE, ENTERPRISEBANKLOCATIONNAME, ENTERPRISEBANKNAME, ENTERPRISECODE, ENTERPRISENAME, FIELD1, FIELD2, INFO, INFOCODE, ISPRIVATE, ISURGENT, LASTIMPORTSEQ, LASTMODIFYTIME, MEMO, MOBILE, MONEYWAY, ORGCODE, PAYSTATE, PAYTYPE, PURPOSE, ROWVERSION, TRANSDATE, TRANSNO, TRANSWAY, URID, CUSTOMERID from fingardtransactions where lastimportseq>496375
)
```

## 零散知识点
- between and 相当于 <=、>=，是包含边界的
- 创建表是不能回退的，即使在事务里面
- 

## 错误和技巧
#### 1. Oracle 无法解析指定的连接标识符
[PLSQL 安装实战](http://blog.csdn.net/smstong/article/details/24359659)

#### 2. 查询 null 值时，使用 is null
```sql
select * from table_name where column is null;
select * from table_name where column is not null;
```

#### 3. 添加一个 oracle 数据库
找到 tnsnames.ora （参考路径：\oracle11g\product\11.2.0\dbhome_1\NETWORK\ADMIN），同时也要修改客户端的 tnsnames.ora （参考路径：D:\program\instantclient_11_2），修改完 oracle 下的拷贝即可
```text
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```
新增这个节点，替换 ORCL 为自己想要的名字，可以任取。HOST 为 所要添加的库的地址，PORT 为端口，SERIVICE_NAME 为服务名

#### 4. PL/SQL 使用小技巧
 - 创建 shortcuts.txt 文本文件，内容见下，置于 PL/SQL 安装目录下的 PlugIns 文件中
 - 菜单中选 Tools -> Perferences -> Editor -> Autoreplaces
 - 重启，输入 = 左侧命令空格即可自动补全

快速补全参考命令：
```txt
i=INSERT
u=UPDATE
s=SELECT
f=FROM
w=WHERE
o=ORDER BY
d=DELETE
dp=DROP TABLE
fp=for update
df=DELETE FROM
sf=SELECT * FROM
sc=SELECT COUNT(*) FROM
sfu=SELECT * FROM FOR UPDATE
cor=CREATE OR REPLACE
p=PROCEDURE
fn=FUNCTION
t=TIGGER
v=VIEW

```

## 用户操作
新增用户

删除用户

修改密码


## 表操作
添加一列
```shell
-- 16 后面不要加 char，如 varchar2(16 char) 不然 oracle 会卡死
alter table singlerefund add orgcode varchar2(16);
```

修改某列长度
```shell
alter table customerconfig modify (dbpassword varchar2(64 char));
```

添加外键
```shell
alter table subscriberconf add constraint FK_subscriberconf_UKCServerid
foreign key (UKCSERVERID)
references serverconf(SERVERID)
```

删除外键
```
ALTER TABLE SubscriberConf DROP CONSTRAINT FK_UKCSERVERID;
```

添加唯一性约束
```shell
ALTER TABLE ServerConf  
ADD CONSTRAINT tb_SERVERID
UNIQUE (SERVERID);
```

查询不是 null 的记录
```shell
select * from supportbusinessconf where SUPPORTBUSINESSID='';
select * from supportbusinessconf where SUPPORTBUSINESSID<>'';
select * from supportbusinessconf where SUPPORTBUSINESSID is null;
select * from supportbusinessconf where SUPPORTBUSINESSID is not null;
```

添加主键
```shell
alter table SingleTransAccountConf 
add constraint pk_SingleTransAccountConf primary key(SUBSCRIBERID, BANKID); 
```
删除主键
```shell
alter table SingleTransAccountConf drop constraint SYS_C0011706;
SELECT * from user_cons_columns where table_name='SingleTransAccountConf';
```

外键值可以为空


## 内置函数
```sql

decode()
-- 生成 uuid
sys_guid()
-- 当前日期
sysdate
-- 转大写
upper()
-- 转小写
lower()
-- 去除字符串中所有的空格
replace("",' ','')
```

## 常用命令

显示某用户所有表(例如：SCOTT，必须大写)  
```sql
select TABLE_NAME from all_tables where owner = 'SCOTT'
```

显示当前的所有用户表  
```sql
select * from user_tables
```

显示当前数据库的所有表  
```sql
select * from tab
```

查找 oracle 外键
```sql
select * from user_cons_columns cl where cl.constraint_name ='SYS_C0022876'
```


## 常用语句

在记录不存在时插入记录
```sql
insert into BankTransActions (id,transNo,drid,balance,bankdetailtype,custaccname,custaccnum,
custbanklocationname,enterpriseaccname,enterpriseaccnum,ledgeramount,moneyway,purpose,receiptstate,
reconciliationcode,reconciliationstate,transdate,memo,createtime,lastmodifytime,datafrom,gived) 
select concat(concat(transdate , '_') ,drid),sys_guid(),drid,balance,bankdetailtype,oppactname,oppactnum,
oppbankname,corpactname,corpactnum,amount,decode(moneyway,'F','0','S','1'),purpose,decode(receiptstate,'0','0','1','0','2','0','3','0','4','1'),
reconcilecode,decode(state,'0','0','1','0','2','0','3','0','4','1'),transdate, memo,sysdate,sysdate,0,0 
from SE_DR20170814 se
where  not exists (
select 1 from BankTransActions bta  where concat(concat(se.transdate , '_') ,drid) = bta.id 
)
```



