
```sql
SELECT DEFAULT_DOMAIN() AS mydomain;

SELECT master.sys.fn_varbintohexstr(SUSER_SID('SIGNED\IT'));  
SELECT master.sys.fn_varbintohexstr(SUSER_SID('SIGNED\mssqlsvc'));
```