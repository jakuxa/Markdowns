## Kettle在DRS中的运用

### 定时任务

- 利用kettle定时到数据源（库）里面运行脚本，将数据抽取到接口表，再调用清洗过程

### 数据导入

- 产品信息

  > 直接导入EXCEL

- 咨询数据

  >  读取dbf文件

### 外部数据导入

- > 也是dbf导入



## Kettle脚本

```sql
//所有kettle脚本的位置和名称
select * from tf_etlscript;
```

```sql
select * from tf_interfaceset a order by a.interface_id
//执行结果2-24是数据导入中的脚本
//101开始是外部数据导入
```



## DRS

### 定时任务执行器

> drs-plantform-6.0
>
> > drs-scheduleclient
> >
> > > src.main.java.com.hundsun.drs
> > >
> > > > job
> > > >
> > > > > handler.etl
> > > > >
> > > > > > 三个日、周、月定时任务执行器



### 数据导入

> drs-plantform-6.0
>
> > drs-datamanagement