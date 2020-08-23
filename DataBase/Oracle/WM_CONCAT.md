**先执行**

```sql
create or replace type string_sum_obj as object ( 
--聚合函数的实质就是一个对象 
     sum_string varchar2(4000), 
     static function ODCIAggregateInitialize(v_self in out string_sum_obj) return number, 
     --对象初始化 
     member function ODCIAggregateIterate(self in out string_sum_obj, value in varchar2) return number, 
     --聚合函数的迭代方法(这是最重要的方法) 
     member function ODCIAggregateMerge(self in out string_sum_obj, v_next in string_sum_obj) return number, 
     --当查询语句并行运行时,才会使用该方法,可将多个并行运行的查询结果聚合 
      
     member function ODCIAggregateTerminate(self in string_sum_obj, return_value out varchar2 ,v_flags in number) return number 
     --终止聚集函数的处理,返回聚集函数处理的结果. 
)
```
**再执行：创建type具体** 

```sql
create or replace type body string_sum_obj is 
     static function ODCIAggregateInitialize(v_self in out string_sum_obj) return number is 
     begin 
         v_self := string_sum_obj(null); 
         return ODCICONST.Success; 
     end; 
     member function ODCIAggregateIterate(self in out string_sum_obj, value in varchar2) return number is 
     begin 
          /* 连接，解决逗号分隔第一个字母是逗号的问题 */    
           if not self.sum_string is null then 
          self.sum_string := self.sum_string ||','|| value; 
          else 
          self.sum_string := self.sum_string || value; 
          end if; 
          return ODCICONST.Success; 
          /* 最大值 */ 
          if self.sum_string<value then 
              self.sum_string:=value; 
          end if; 
          /* 最小值 */ 
          if self.sum_string>value then 
       self.sum_string:=value;           
          end if; 
           
          return ODCICONST.Success; 
     end; 
     member function ODCIAggregateMerge(self in out string_sum_obj, v_next in string_sum_obj) return number is 
     begin 
          /* 连接 */    
          self.sum_string := self.sum_string || v_next.sum_string; 
          return ODCICONST.Success; 
          /* 最大值 */ 
          if self.sum_string<v_next.sum_string then 
              self.sum_string:=v_next.sum_string; 
          end if; 
 
          /* 最小值 */ 
          if self.sum_string>v_next.sum_string then 
              self.sum_string:=v_next.sum_string;           
          end if; 
           
          return ODCICONST.Success; 
     end; 
     member function ODCIAggregateTerminate(self in string_sum_obj, return_value out varchar2 ,v_flags in number) return number is 
     begin 
          return_value:= self.sum_string; 
          return ODCICONST.Success; 
     end; 
end;
```
**最后执行：**

```sql
--创建函数 
create or replace function wm_concat(value Varchar2) return Varchar2 
     parallel_enable aggregate using string_sum_obj;
```

**最后去执行WM_CONCAT()函数会发现，他的效果和以前一样一样！**