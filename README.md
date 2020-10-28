# OBIEE

```{}
where A.INPUTTER in (@{P_INPUTTER}{''})
and A.DAYID BETWEEN @{P_FROM_DATE}{''} AND @{P_TO_DATE}{''};
```

```{}
-- Gõ nhiều mã vào prompt filter
WITH tmp_tab AS
 (SELECT upper(regexp_replace(regexp_substr('@{P_POS}', ',*[^,]+', 1, LEVEL), ',| ', '')) all_values
  FROM dual a
  CONNECT BY regexp_substr('@{P_POS}', ',*[^,]+', 1, LEVEL) IS NOT NULL)
SELECT * FROM SB_BI.B_TID_ERROR_DAILY_OBIEE
 WHERE (MA_POS IN (SELECT * FROM tmp_tab) OR '@{P_POS}' = '@' || '{P_POS}')
   AND NGAY_GIAO_DICH BETWEEN @{P_FROM_DATE}{TRUNC(SYSDATE-20)} AND @{P_TO_DATE}{TRUNC(SYSDATE-1)};
```

## PROMPT
```{}
Server Variable: SYSDATE_RPD
```

# Gõ tiếng việt PLSQL
```{}
-- Them vao target / plsqldev.exe => khong loi font
nls_lang=AMERICAN_AMERICA.AL32UTF8
```

# NHỮNG LỆNH CƠ BẢN

## CREATE
```{}
CREATE TABLE employees
( employee_number number(10) NOT NULL,
  employee_name varchar2(50) NOT NULL,
  department_id number(10),
  salary number(6),
  CONSTRAINT employees_pk PRIMARY KEY (employee_number),
  CONSTRAINT fk_departments
  FOREIGN KEY (department_id)
  REFERENCES departments(department_id)
);

CREATE TABLE suppliers
  AS (SELECT company_id, address, city, state, zip
      FROM companies
      WHERE company_id < 5000);
```

## DROP

```{}
DROP TABLE customers PURGE;

SELECT * FROM USER_RECYCLEBIN; -- danh sach bang trong thung rac
FLASHBACK TABLE "BIN$JdOC4qukDkzgUx44qMDdYw==$0" TO BEFORE DROP; -- khoi phuc bang
PURGE RECYCLEBIN; -- xoa het bang trong thung rac
```

## INSERT INTO
```{}
INSERT INTO sinh_vien VALUES ("Trinh Giao Kim","Nam","44","Bac Lieu");

INSERT INTO sinh_vien (Full_name, Gender, Age, City)
VALUES ("Trinh Giao Kim","Nam","44","Bac Lieu");
```

Note: Nếu trường NOT NULL thì tự thêm giá trị mặc định

## SELECT 

### Cơ bản
```{}
SELECT *
FROM sinh_vien
WHERE Gender="Nam" AND Age<=35
ORDER BY Gender DESC, Age ASC;
```

### Chọn hàng có giá trị khác nhau

```{}
SELECT DISTINCT Age, Gender
FROM sinh_vien;
```

### Giới hạn số lượng dòng
```{}
SELECT *
FROM sinh_vien
LIMIT 3;
```

### LIKE 
```{}
-- Chọn sinh viên bắt đầu bằng chữ Nguyen
SELECT *
FROM sinh_vien
WHERE Full_name LIKE "Nguyen%";

-- Có chứa ít nhất 2 ký tự T và T
SELECT *
FROM sinh_vien
WHERE Full_name LIKE "%T%T%";
```

## DELETE
```{}
DELETE FROM sinh_vien
WHERE Gender = "Nam";

Note: Nếu không có WHERE thì xóa hết dữ liệu trong bảng
```

## UPDATE Cập nhật dữ liệu trong bảng 
```{}
UPDATE table_name
SET column_name1=value1, column_name2=value2
WHERE column_name=value;
```

## JOIN
```{}
SELECT columns
FROM table1 
INNER JOIN table2
ON table1.column = table2.column;
```

## ALTER TABLE

### Thêm cột mới
```{}
ALTER TABLE table_name
ADD column_name datatype;
```
### Xóa cột
```{}
ALTER TABLE table_name
DROP COLUMN column_name;
```
### Đổi tên
```{}
ALTER TABLE table_name
RENAME COLUMN old_column_name TO new_column_name;
```
### Sửa cột
```{}
ALTER TABLE table_name
MODIFY column_name datatype;
```

# TIPS & TRICKS

## DATE
```{}
SELECT 
TRUNC(ADD_MONTHS(SYSDATE, -7), 'MONTH'),
, TRUNC(LAST_DAY(ADD_MONTHS(SYSDATE, -1)))+86399/86400
, TRUNC(SYSDATE -1)
, SYSDATE -1
FROM DUAL;
```

```{}
-- CREATE TABLE B_LAST_DAY AS 
SELECT TRUNC(LAST_DAY(ADD_MONTHS(LAST_DAY('31-dec-2016'), LEVEL))) LAST_DAY
FROM DUAL
CONNECT BY LEVEL <= 60;

SELECT TO_CHAR(SYSDATE, 'YYYY') AS YEAR,
       TO_CHAR(SYSDATE, 'MM') AS MONTH,
       TO_CHAR(SYSDATE, 'DD') AS DAY,
       TO_CHAR(SYSDATE, 'HH24') AS HOUR,
       TO_CHAR(SYSDATE, 'MI') AS MINUTE
  FROM DUAL;
```

```{}
 SELECT TO_CHAR(SYSDATE, 'DD-MM-YYYY') FROM dual;
```
## COMPARE TWO TABLE
```{}
-- all rows that are in T1 but not in T2
(SELECT /*+ PARALLEL(11) */ * FROM B_TMP MINUS SELECT * FROM B_TMP1) 
UNION ALL
-- ALL ROWS THAT ARE IN T2 BUT NOT IN T1
(SELECT * FROM B_TMP1 MINUS SELECT * FROM B_TMP)  
;
```
## PARTITION
```{}
SELECT /*+ PARALLEL(11) */
 C.*,
 MIN(FIRST_ACTIVED_DATE) OVER(PARTITION BY CUSTOMER) MIN_BY_CUSTOMER,
 FIRST_VALUE(FIRST_ACTIVED_DATE) IGNORE NULLS OVER(PARTITION BY CUSTOMER ORDER BY FIRST_ACTIVED_DATE ASC) FIRST_BY_CUSTOMER, -- EQUAL MIN IF NOT NULL
 MAX(FIRST_ACTIVED_DATE) OVER(PARTITION BY CUSTOMER) MAX_BY_CUSTOMER,
 LAST_VALUE(FIRST_ACTIVED_DATE) IGNORE NULLS OVER(PARTITION BY CUSTOMER ORDER BY FIRST_ACTIVED_DATE ASC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING ) LAST_BY_CUSTOMER -- EQUAL MAX IF NOT NULL -- NOTE ORDER
  FROM B_RT_CARD C;
```

## Check size of tables
```{}
SELECT /* Segment size - Total */
 OWNER,
 SEGMENT_NAME,
 SEGMENT_TYPE,
 ROUND(SUM(S.BYTES) / 1024 / 1024, 1) AS "MB"
  FROM DBA_SEGMENTS S
 WHERE OWNER = 'SB_BI'
 AND SEGMENT_NAME LIKE '%B_%'
 GROUP BY OWNER, SEGMENT_NAME, SEGMENT_TYPE
 ORDER BY ROUND(SUM(S.BYTES) / 1024 / 1024, 1) DESC;
```

## RUN PROCEDURE
```{}
begin 
    b_tid_trans_daily;
end;
```

## CODE đang chạy
```{}
SELECT * FROM RN;
```
## KIEM TRA PHAN QUYEN
```{}
SELECT regexp_substr(ATTRVAL,'[^=]+$') login_id, dn.*, attr.*
FROM exa_opss.jps_attrs attr, exa_opss.jps_dn dn
WHERE attr.jps_dn_entryid = dn.entryid
      AND parentdn = 'cn=opssroot,cn=jpscontext,cn=opsssecuritystore,cn=obi,cn=roles,'
      AND attrname = 'orcljaznprincipal'
      --AND regexp_like(attrval, 'thuy.dtt4|mai.tt6$', 'i')
      AND rdn IN ('cn=khcn_quan_tri_danh_muc_chinh_sach_kh')
ORDER BY attrval, rdn;
```

## Running job

```{}
select * from all_scheduler_jobs
WHERE JOB_NAME LIKE 'B_%';
-- stop 
EXEC DBMS_SCHEDULER.STOP_JOB (job_name => 'B_DAILY_ALL_PROCEDURES_JOB');
```

## KIỂM TRA DUNG LƯỢNG Ổ CỨNG CÒN LẠI

```{}
SELECT SUM(bytes) / 1024 / 1024 mb FROM dba_segments WHERE owner = 'SB_BI';
```
## GRANT PACKAGE

```{}
GRANT EXECUTE ON LOAN TO VHCN_PTKD_BI;
```

## TO_CHAR
```{}
alter session set nls_territory = 'UNITED KINGDOM';
SELECT TO_CHAR(SYSDATE - 3, 'D') FROM dual; -- monday = 1 - sunday = 7 - saturday = 6
alter session set nls_territory = 'AMERICA';
SELECT TO_CHAR(SYSDATE - 3, 'D') FROM dual; -- monday = 2 - sunday = 1 - saturday = 7
```
## GROUP MULTIPLE ROWS

```{}
SELECT LC.LIMIT_ID
      ,LISTAGG(LC.LIMIT_ID, ',') WITHIN GROUP(ORDER BY LC.COLLATERAL_ID) GRP_COLLATERAL_ID
  FROM SB_DTM.MAP_LIMIT_COLLA LC
 WHERE LC.DAYID = TRUNC(SYSDATE - 1)
 GROUP BY LC.LIMIT_ID;
```
