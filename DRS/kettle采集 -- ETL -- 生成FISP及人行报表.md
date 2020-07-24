## 20200707  kettle采集 --> ETL --> 生成FISP及人行报表

### TA&估值



数据库定时脚本

FA30-Daily1,Daily2,Daily3

![image-20200707195732288](/Users/jakuxa/Library/Application Support/typora-user-images/image-20200707195732288.png)

![image-20200707195832465](/Users/jakuxa/Library/Application Support/typora-user-images/image-20200707195832465.png)

fa： kettle(获取数据源)---->DRS系统(TID表   ---> ETL存储过程转换  --->tb表 )

- 客户代码与个性化，系统参数表——ts_systemparam

execute immediate 执行str字符串中的sql语句，str可拼接

### 会计科目结存

- v_l_sojo： 科目首次结存与否

- zx：数据来自DC数据中心，定时任务抽取





编写脚本时，只有编译才能排查语法错误。