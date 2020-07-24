## DRS6.0-CISP报表数据溯源

### CISP报表主要业务所在服务地址

> com.hundsun.drs.sentmanagement
>
> > CispSendReportController



### 获取CISP主表信息

> com.hundsun.drs.sentmanagement 
>
> > mapper.TfCispSendReportMapper.selectCispZbReportInfoByExample

```sql
# 核心表 ts_report_sendset、tf_reportinfo、tb_managers、tcisp_zb_report、ts_report_check_log、ts_userlogin、ts_report_approve

select* from
  (select * from ts_report_sendset rs, tf_reportinfo ri, tb_managers mr
     where rs.report_id = ri.report_id,
     and rs.manager_id = mr.order_number
     and rs.isreport=1) a
  left join (SELECT mxzb.*,
             usrlgn.user_full_name,
             d.approve_status,
             chklog.remark
             FROM (SELECT t.*
                   FROM tcisp_zb_report t
                   WHERE t.report_date = #{reportDate}
                   and (t.manager_id, t.report_date, t.report_no,
                        t.batch_id) IN
                   (SELECT manager_id,
                    report_date,
                    report_no,
                    max(batch_id) batch_id
                    FROM tcisp_zb_report
                    GROUP BY manager_id, report_date, report_no)) mxzb
             LEFT JOIN ts_report_check_log chklog
             on chklog.file_id = mxzb.file_id
             LEFT JOIN ts_userlogin usrlgn
             on usrlgn.user_id = mxzb.user_audit
             LEFT JOIN (SELECT file_id, process_id, approve_status
                        FROM ts_report_approve
                        WHERE (file_id, process_id) in
                        (SELECT file_id,
                         max(process_id) AS process_id
                         FROM ts_report_approve
                         GROUP BY file_id)) d
             on mxzb.file_id = d.file_id) zb
                    on (a.manager_id = zb.manager_id and a.report_no = zb.report_no)
                    where 1=1
                    and a.report_no in (#{reportNo})
                    and a.manager_id in (#{managerId})
                    and a.frequency = #{frequency}
                    and a.report_category = #{reportCategory}
                    and nvl(zb.make_status, 0) = #{makeStatus}
                    order by a.report_code, a.manager_id
```



### 生成CISP报表

> com.hundsun.drs.sentmanagement.service.impl
>
> > CispSendReportServiceImpl

- 获得cisp报表信息

  ```sql
  # 核心表 TF_REPORTINFO，可查询对应数据表，TF_REPORTINFO.TABLE_NAME
  
  select
  REPORT_NO, REPORT_NAME, STRUCTURE_FLAG, FILE_FORM, FREQUENCY, SEND_TIME, INCREMENT_FLAG, 
  RECEIVER, DESCRIBE, CONTENT_EXPLAIN, REPORT_CATEGORY, TABLE_NAME, ISPAGING, 
  LIMIT_NUMBER, REPORT_COLNUM, FUNCTION_MODULE, VALUE_MODULE, REPORT_CODE, REPORT_ID
  from TF_REPORTINFO
  where REPORT_NO = #{reportNo}
  ```

- 获得报表数据

  ```java
  com.hundsun.drs.sentmanagement.atom.impl.getExportReportData
  # mapper报表数据查询地址
  com.hundsun.drs.sentmanagement.mapper
  
  ```

  



### 根据报表ID获取模板设计信息

```sql
select REPORT_ID, CREATE_USER, CREATE_TIME, TEMPLATE_FILE, CONFIG_FILE 
from TS_REPORT_TEMPLATE
where REPORT_ID = #{reportId,jdbcType=NUMERIC}
```

