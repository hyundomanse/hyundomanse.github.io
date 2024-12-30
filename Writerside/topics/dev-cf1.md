# 개발참고1

*** PostgreSQL ***

PostgreSQL에서 테이블의 컬럼에 대한 주석(comment) 정보를 포함하여 조회하려면, 다음과 같이 column_description 칼럼을 포함한 쿼리를 사용할 수 있습니다:

sql

SELECT
table_schema,
table_name,
column_name,
data_type,
col_description(
CONCAT_WS('.', table_schema, table_name)::regclass, ordinal_position
) AS column_description
FROM
information_schema.columns
ORDER BY
table_schema,
table_name,
ordinal_position;
이 쿼리는 다음과 같은 정보를 반환합니다:

table_schema: 테이블의 스키마 이름
table_name: 테이블 이름
column_name: 컬럼 이름
data_type: 컬럼의 데이터 타입
column_description: 컬럼에 대한 주석 (comment)
col_description 함수는 지정된 테이블과 칼럼의 주석을 반환합니다. 이 함수는 테이블의 OID와 컬럼의 위치 (ordinal_position)를 입력으로 받아야 합니다. 스키마와 테이블 이름을 연결(concatenate)하여 OID로 변환한 후, 해당 테이블의 특정 컬럼에 대한 주석을 가져오는 방식입니다.

이렇게 하면 각 컬럼에 대한 주석을 포함한 상세한 정보를 조회할 수 있습니다.

- 테이블 vacuum, analyze 날짜 조회
  select pg_stat_all_tables.relname, last_vacuum, last_autovacuum, last_analyze
  from pg_stat_all_tables
  join pg_class on pg_stat_all_tables.relid = pg_class.oid
  where pg_stat_all_tables.relname like '%tb_%';

--superuser 권한 확인
SELECT current_user, session_user, rolsuper
FROM pg_roles
WHERE rolname = current_user;


*** IntelliJ ***
Poject setting(options) > Always Select Opened File 체크 : 네이게이터 자동 이동
Shift + Alt + U : CamelCase 변환
Ctrl + Alt + S : Setting
Editor > General > Auto Import > Add unambiguous imports on the fly : 자동 import 넣기
> optimize imports on the fly : 미사용 import 자동 제거


*** NotePad++ ***
스네이크 to 카멜
1. 전체 소문자로 변경
2. 바꾸기 (Ctrl + H) 에서 정규식 선택
3. 찾을 내용 : [_]{1,1}([a-z])
4. 바꿀 내용 : \u$1

카멜 to 스네이크
1. 찾을 내용 : (\b[a-z]+|\G(?!^))((?:[A-Z]|\d+)[a-z]*)
2. 바꿀 내용 : \U\1_\2
3. 카멜 적용 안돼있는 경우는 소문자로 남음


맨 앞에 텍스트 넣기
1. 찾을 내용 : ^
2. 바꿀 내용 : ;

맨 뒤에 텍스트 넣기
1. 찾을 내용 : $
2. 바꿀 내용 : ;

Ctrl + L : 선택 된 줄 또는 현재 줄 삭제 한다
Ctrl + D : 선택 된 줄 또는 현재 줄 복사해서 붙여넣기 해준다
Ctrl + J : 선택 된 줄 병합

Ctrl + Shift + 위,아래 방향키 : 선택된 줄 또는 현재 줄을 위 아래로 이동
Ctrl + Q : 선택 된 줄 또는 현재 줄 주석처리 및 주석 삭제
Ctrl + K : 선택 된 줄 또는 현재 줄 주석처리

Ctrl + Shift + Q : Multi Line 주석 처리 (선택 된 줄 또는 현재 줄) /**/
Ctrl + Shift + K : Multi Line 주석 삭제 (선택 된 줄 또는 현재 줄) /**/ or --


Ctrl + u : 선택된 부분 소문자로 변환
Ctrl + U : 선택된 부분 대문자로 변환

Ctrl + Shift + Delete : 현재 위치에서 문단의 끝까지 삭제, Multi Line 선택 시에도 적용 됨
Ctrl + Shift + Backspace : 현재 위치에서 문단의 시작까지 삭제, Multi Line 선택 시에도 적용 됨
Ctrl + w : 현재 문서 닫기



WITH first_and_last AS (                                                                                                                                                                                                
select row_number()over (order by data_ins_dtm asc) as no,                                                                                                                                                          
data_ins_dtm,                                                                                                                                                                                                
seq_id,                                                                                                                                                                                                      
subsidiary_code,                                                                                                                                                                                             
employee_number                                                                                                                                                                                              
from tb_rpjte_hr_inno_emp_i order by data_ins_dtm asc offset 165100 limit 100                                                                                                                                       
)                                                                                                                                                                                                                       
(                                                                                                                                                                                                                       
SELECT *
FROM first_and_last                                                                                                                                                                                                 
where no = (select min(no) from first_and_last)                                                                                                                                                                     
LIMIT 1                                                                                                                                                                                                             
)                                                                                                                                                                                                                       
UNION ALL                                                                                                                                                                                                               
(                                                                                                                                                                                                                       
SELECT *
FROM first_and_last                                                                                                                                                                                                 
where no = (select max(no) from first_and_last)                                                                                                                                                                     
LIMIT 1                                                                                                                                                                                                             
);

-- 100 건씩                                                                                                                                                                                                               
WITH numbered_rows AS (                                                                                                                                                                                                 
SELECT ROW_NUMBER() OVER (ORDER BY data_ins_dtm ASC) AS rownum,                                                                                                                                                     
LEAD(data_ins_dtm) OVER (ORDER BY data_ins_dtm ASC) AS next_date_col,                                                                                                                                               
data_ins_dtm, seq_id, subsidiary_code, employee_number                                                                                                                                                              
FROM tb_rpjte_hr_inno_emp_i                                                                                                                                                                                         
)                                                                                                                                                                                                                       
SELECT rownum, data_ins_dtm,                                                                                                                                                                                            
next_date_col - data_ins_dtm AS date_diff,                                                                                                                                                                      
seq_id, subsidiary_code, employee_number                                                                                                                                                                        
FROM numbered_rows                                                                                                                                                                                                      
WHERE (rownum - 1) % 100 = 0;


-- 시간 차이                                                                                                                                                                                                                
with tab as (                                                                                                                                                                                                           
WITH numbered_rows AS (                                                                                                                                                                                             
SELECT ROW_NUMBER() OVER (ORDER BY data_ins_dtm ASC) AS rownum                                                                                                                                                      
,data_ins_dtm, seq_id, subsidiary_code, employee_number                                                                                                                                                     
FROM tb_rpjte_hr_inno_emp_i                                                                                                                                                                                         
)                                                                                                                                                                                                                   
SELECT *, LEAD(data_ins_dtm) OVER (ORDER BY data_ins_dtm ASC) AS next_date_col                                                                                                                                      
FROM numbered_rows                                                                                                                                                                                                  
WHERE (rownum - 1) % 100 = 0                                                                                                                                                                                        
)                                                                                                                                                                                                                       
SELECT rownum, data_ins_dtm,                                                                                                                                                                                            
next_date_col - data_ins_dtm AS date_diff,                                                                                                                                                                      
seq_id, subsidiary_code, employee_number                                                                                                                                                                        
FROM tab                                                                                                                                                                                                                
WHERE (rownum - 1) % 100 = 0;                                                                                                                                                                                           
order by date_diff desc;         