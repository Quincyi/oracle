# 实验四：对象管理
#### 刘倩 软件18-4 学号：201810414403
用户名：new_lq
### 实验目的：
了解Oracle表和视图的概念，学习使用SQL语句Create Table创建表，学习Select语句插入，修改，删除以及查询数据，学习使用SQL语句创建视图，学习部分存储过程和触发器的使用。
### 实验场景：
假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
### 实验内容：
#### 录入数据：
要求至少有1万个订单，每个订单至少有4个详单。至少有两个部门，每个部门至少有1个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如A领导B，B领导C）。
#### 序列的应用
插入ORDERS和ORDER_DETAILS 两个表的数据时，主键ORDERS.ORDER_ID, ORDER_DETAILS.ID的值必须通过序列SEQ_ORDER_ID和SEQ_ORDER_ID取得，不能手工输入一个数字。
#### 触发器的应用
维护ORDER_DETAILS的数据时（insert,delete,update）要同步更新ORDERS表订单应收货款ORDERS.Trade_Receivable的值。
#### 查询数据
1.查询某个员工的信息
2.递归查询某个员工及其所有下属，子下属员工。
3.查询订单表，并且包括订单的订单应收货款: Trade_Receivable= sum(订单详单表.ProductNum*订单详单表.ProductPrice)- Discount。
4.查询订单详表，要求显示订单的客户名称和客户电话，产品类型用汉字描述。
5.查询出所有空订单，即没有订单详单的订单。
6.查询部门表，同时显示部门的负责人姓名。
7.查询部门表，统计每个部门的销售总金额。
### 表结构
部门表DEPARTMENTS,表空间：USERS 
<table>
    <tr>
        <th>编号</th>
        <th>字段名</th>
        <th>数据类型</th>
        <th>可以为空</th>
        <th>注释</th>
    </tr>
    <tr>
        <td>1</td>
        <td>DEPARTMENT_ID</td>
        <td>NUMBER(6,0)</td>
        <td>NO</td>
        <td>部门ID，主键</td>
    </tr>
    <tr>
        <td>2</td>
        <td>DEPARTMENT_NAME</td>
        <td>VARCHAR2(40 BYTE)	</td>
        <td>NO</td>
        <td>部门名称，非空</td>
    </tr>
</table>
产品表PRODUCTS,表空间：USERS
<table>
    <tr>
        <th>编号</th>
        <th>字段名</th>
        <th>数据类型</th>
        <th>可以为空</th>
        <th>注释</th>
    </tr>
    <tr>
        <td>1</td>
        <td>PRODUCT_NAME</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>NO</td>
        <td>产品名称，产品表的主键</td>
    </tr>
    <tr>
        <td>2</td>
        <td>PRODUCT_TYPE</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>NO</td>
        <td>产品类型，只能取值：耗材,手机,电脑</td>
    </tr>
</table>
员工表EMPLOYEES,表空间：USERS
<table>
    <tr>
        <th>编号</th>
        <th>字段名</th>
        <th>数据类型</th>
        <th>可以为空</th>
        <th>注释</th>
    </tr>
    <tr>
        <td>1</td>
        <td>EMPLOYEE_ID</td>
        <td>NUMBER(6,0)</td>
        <td>NO</td>
        <td>员工ID，员工表的主键。</td>
    </tr>
    <tr>
        <td>2</td>
        <td>NAME</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>NO</td>
        <td>员工姓名，不能为空，创建不唯一B树索引。</td>
    </tr>
    <tr>
        <td>3</td>
        <td>EMAIL</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>YES</td>
        <td>电子信箱</td>
    </tr>
    <tr>
        <td>4</td>
        <td>PHONE_NUMBER</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>YES</td>
        <td>电话</td>
    </tr>
    <tr>
        <td>5</td>
        <td>HIRE_DATE</td>
        <td>DATE</td>
        <td>NO</td>
        <td>雇佣日期</td>
    </tr>
    <tr>
        <td>6</td>
        <td>SALARY</td>
        <td>NUMBER(8,2)</td>
        <td>YES</td>
        <td>月薪，必须>0</td>
    </tr>
    <tr>
        <td>7</td>
        <td>MANAGER_ID</td>
        <td>NUMBER(6,0)</td>
        <td>YES</td>
        <td>员工的上司，是员工表EMPOLYEE_ID的外键，MANAGER_ID不能等于EMPLOYEE_ID,即员工的领导不能是自己。主键删除时MANAGER_ID设置为空值。</td>
    </tr>
    <tr>
        <td>8</td>
        <td>DEPARTMENT_ID</td>
        <td>NUMBER(6,0)</td>
        <td>YES</td>
        <td>员工所在部门，是部门表DEPARTMENTS的外键</td>
    </tr>
    <tr>
        <td>9</td>
        <td>PHOTO</td>
        <td>BLOB</td>
        <td>YES</td>
        <td>员工照片</td>
    </tr>
</table>
订单表ORDERS, 表空间：分区表：USERS,USERS02
<table>
    <tr>
        <th>编号</th>
        <th>字段名</th>
        <th>数据类型</th>
        <th>可以为空</th>
        <th>注释</th>
    </tr>
    <tr>
        <td>1</td>
        <td>ORDER_ID</td>
        <td>NUMBER(10,0)</td>
        <td>NO</td>
        <td>订单编号，主键，值来自于序列：SEQ_ORDER_ID</td>
    </tr>
    <tr>
        <td>2</td>
        <td>CUSTOMER_NAME</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>NO</td>
        <td>客户名称，B树索引</td>
    </tr>
    <tr>
        <td>3</td>
        <td>CUSTOMER_TEL</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>NO</td>
        <td>客户电话</td>
    </tr>
    <tr>
        <td>4</td>
        <td>ORDER_DATE</td>
        <td>DATE</td>
        <td>NO</td>
        <td>订单日期，根据该属性分区存储：2015年及以前的数据存储在USERS表空间，2016年及以后的数据存储在USERS02表空间中。</td>
    </tr>
    <tr>
        <td>5</td>
        <td>EMPLOYEE_ID</td>
        <td>NUMBER(6,0)</td>
        <td>NO</td>
        <td>订单经手人，员工表EMPLOYEES的外键</td>
    </tr>
    <tr>
        <td>6</td>
        <td>DISCOUNT</td>
        <td>Number(8,2)</td>
        <td>YES</td>
        <td>订单整体优惠金额。默认值为0</td>
    </tr>
    <tr>
        <td>7</td>
        <td>TRADE_RECEIVABLE</td>
        <td>Number(8,2)</td>
        <td>YES</td>
        <td>订单应收货款，默认为0，Trade_Receivable= sum(订单详单表.Product_Num*订单详单表.Product_Price)- Discount</td>
    </tr>
</table>
订单详单表ORDER_DETAILS, 表空间：分区表：USERS,USERS02，分区参照ORDERS表。
<table>
    <tr>
        <th>编号</th>
        <th>字段名</th>
        <th>数据类型</th>
        <th>可以为空</th>
        <th>注释</th>
    </tr>
    <tr>
        <td>1</td>
        <td>ID</td>
        <td>NUMBER(10,0)</td>
        <td>NO</td>
        <td>本表的主键，值来自于序列：SEQ_ORDER_DETAILS_ID</td>
    </tr>
    <tr>
        <td>2</td>
        <td>ORDER_ID</td>
        <td>NUMBER(10,0)</td>
        <td>NO</td>
        <td>所属的订单号，订单表ORDERS的外键</td>
    </tr>
    <tr>
        <td>4</td>
        <td>PRODUCT_NAME</td>
        <td>VARCHAR2(40 BYTE)</td>
        <td>NO</td>
        <td>产品名称, 是产品表PRODUCTS的外键</td>
    </tr>
    <tr>
        <td>5</td>
        <td>PRODUCT_NUM</td>
        <td>NUMBER(8,2)</td>
        <td>NO</td>
        <td>产品销售数量，必须>0</td>
    </tr>
    <tr>
        <td>6</td>
        <td>PRODUCT_PRICE</td>
        <td>NUMBER(8,2)</td>
        <td>NO</td>
        <td>产品销售价格</td>
    </tr>
</table>

### 实验步骤：
1、删除表和序列，删除表的同时会一起删除主外键、触发器、程序包
![1](1.png)
2、DDL for Table DEPARTMENTS
![2](2.png)
3、DDL for Table EMPLOYEES
![3](3.png)
4、DDL for Table ORDER_ID_TEMP
![4](4.png)
5、DDL for Table ORDERS
![5](5.png)
6、创建本地分区索引ORDERS_INDEX_DATE：
![6](6.png)
![7](7.png)
![8](8.png)
![9](9.png)
![10](10.png)
![11](11.png)
![12](12.png)
![13](13.png)

7、创建3个触发器
![14](14.png)
8、批量插入订单数据之前，禁用触发器
![15](15.png)
9、DDL for Trigger ORDER_DETAILS_ROW_TRIG
![16](16.png)
10、DDL for Trigger ORDER_DETAILS_SNTNS_TRIG
![17](17.png)
11、DDL for Sequence
![18](18.png)
12、DDL for View VIEW_ORDER_DETAILS
![19](19.png)
13、插入DEPARTMENTS，EMPLOYEES数据
![20](20.png)
14、批量插入订单数据，注意ORDERS.TRADE_RECEIVABLE（订单应收款）的自动计算,注意插入数据的速度。2千万条记录，插入的时间是：18100秒（约5小时）
![21](21.png)
![22](22.png)
15、最后动态增加一个PARTITION_BEFORE_2018分区
![23](23.png)
16、一切就绪，开始测试。以下时间在0.02秒以内才正常：（id取值从1到20000000）
![24](24.png)
![25](25.png)
![26](26.png)
17、递归查询某个员工及其所有下属，子下属员工。
![27](27.png)
![28](28.png)
18、特殊查询语句：查询分区表情况
![29](29.png)
19、查询一个分区中的数据
![30](30.png)
![31](31.png)
![32](32.png)
![33](33.png)
20、统计用户的所有表
![34](34.png)
![35](35.png)
21、查看数据文件的使用情况（以system用户登录查询）
![36](36.png)
22、查看表空间的使用情况
![37](37.png)
