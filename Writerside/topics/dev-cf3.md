# 개발참고3

select * from pg_class

select pg_stat_all_tables.relname, last_vacuum, last_autovacuum, last_analyze
from pg_stat_all_tables
join pg_class on pg_stat_all_tables.relid = pg_class.oid
where pg_stat_all_tables.relname like '%tb_%';

select relname, last_vacuum, last_autovacuum, last_analyze from pg_stat_all_tables where relname like '%tb_%' order by last_autovacuum;

SELECT c.relname AS table_name, pg_catalog.pg_stat_file(pg_catalog.pg_relation_filepath(c.oid)).modification AS modification_time
FROM pg_catalog.pg_class c
WHERE c.relname = 'tb_dep_prj_base_info_d';

select  * from pg_catalog.pg_class c

SELECT current_user, session_user, rolsuper
FROM pg_roles
WHERE rolname = current_user;

SELECT *
FROM information_schema.role_table_grants;

----------------------------------------------------------------------------------------------

update tb_rpjte_dept_m
set dept_cd = B.organization_code
from tb_rpjte_inno_hr_dept_i B
where tb_rpjte_dept_m.dept_cd = B.organization_id::VARCHAR
and tb_rpjte_dept_m.dept_nm = B.organization_name;


update tb_rpjte_emp_m
set eml_svr_dmn_ifo_nm = t2.email_address
from tb_rpjte_inno_hr_emp_i t2
where tb_rpjte_emp_m.user_id = t2.sso_user_id;


update tb_rpjte_emp_m
set dept_id = T.organization_id
from (select D.organization_id, A.user_id
from tb_rpjte_emp_m A
inner join tb_rpjte_cmn_c C on C.cmn_gr_cd = 'COP_CD' and C.use_yn = 'Y' and C.cmn_cd = A.cop_cd
inner join tb_rpjte_inno_hr_dept_i D on D.organization_code = A.dept_cd and D.organization_name = A.dept_nm
where 1=1) T
where tb_rpjte_emp_m.user_id = T.user_id;


ALTER TABLE tb_rpjte_dept_m DROP CONSTRAINT tb_rpjte_dept_m_pk;


UPDATE tb_rpjte_emp_m
SET dept_id = T.organization_id
FROM (
SELECT E.user_id, E.dept_nm, E.dept_cd, I.organization_id
FROM tb_rpjte_emp_m E
INNER JOIN tb_rpjte_inno_hr_dept_i I ON I.organization_code = E.dept_cd AND I.organization_name = E.dept_nm
WHERE E.dept_id IS NULL
AND E.cop_cd != 'B100'
) T
WHERE tb_rpjte_emp_m.user_id = T.user_id;

----------------------------------------------------------------------------------------------

WITH
RECURSIVE
rcs_tb AS (SELECT
1 AS level
, s.*
FROM
tb_rpjte_inno_hr_dept_i s
WHERE
subsidiary_code = 'KR'
AND organization_code LIKE 'K%'
AND parent_org_id IS NULL
UNION ALL
SELECT
level + 1 AS level
, s.*
FROM
rcs_tb r
INNER JOIN tb_rpjte_inno_hr_dept_i s ON r.organization_id = s.parent_org_id
AND s.subsidiary_code = 'KR'
AND s.organization_code LIKE 'K%')
SELECT *
FROM
rcs_tb
ORDER BY
level, sortcode ASC ;




-- 테이블 코멘트 조회
select T1.table_name,T2.description from (SELECT table_name
FROM information_schema.tables
where table_catalog = 'depdb'
and table_schema = 'public') T1
left join (SELECT description, (select relname from pg_class where oid = a.objoid) as relname
FROM pg_catalog.pg_description a
WHERE objoid in ( SELECT oid
FROM pg_class
WHERE 1=1
AND relnamespace = ( SELECT oid FROM pg_catalog.pg_namespace WHERE nspname = 'public' ) )  AND objsubid = 0) T2

                                                       on T1. table_name = T2.relname


-- 컬럼 코멘트 조회
SELECT
concat('comment on column ', c.table_name,'.',c.column_name, ' is '''';'),
c.table_schema,
c.table_name,
c.column_name,
d.description AS column_comment
FROM
information_schema.columns c
LEFT JOIN
pg_description d
ON
c.table_name::regclass = d.objoid AND c.ordinal_position::int - 1 = d.objsubid
WHERE 1=1
AND
c.table_schema = 'public'
and d.description is null
ORDER BY
c.table_schema, c.table_name, c.column_name;


-- 테이블 코멘트 생성
select
concat('comment on table ',T1.table_name,' is '''';' )

     ,T2.description from (SELECT table_name
                                          FROM information_schema.tables
                                          where table_catalog = 'wsfdb'
                                            and table_schema = 'public') T1
                                             left join (SELECT description, (select relname from pg_class where oid = a.objoid) as relname
                                                        FROM pg_catalog.pg_description a
                                                        WHERE objoid in ( SELECT oid
                                                                          FROM pg_class
                                                                          WHERE 1=1
                                                                            AND relnamespace = ( SELECT oid FROM pg_catalog.pg_namespace WHERE nspname = 'public' ) )  AND objsubid = 0) T2

                                                       on T1. table_name = T2.relname

where T2.description is null;


-- 컬럼 코멘트 생성

SELECT
concat('comment on column ', c.table_name,'.',c.column_name, ' is '''';'),
c.table_schema,
c.table_name,
c.column_name,
d.description AS column_comment
FROM
information_schema.columns c
LEFT JOIN
pg_description d
ON
c.table_name::regclass = d.objoid AND c.ordinal_position::int - 1 = d.objsubid
WHERE 1=1
AND
c.table_schema = 'public'
and d.description is null
ORDER BY
c.table_schema, c.table_name, c.column_name;




Insert on Conflict(Upsert) 시 수정/추가된 Row 수 알기 : xmax
- 특정 컬럼에 권한 부여하기
- 멀티 패턴 매칭 : SIMILAR TO 또는 ~ 정규식
- 현재 시퀀스값 증가시키지 않고 알아내기 : pg_sequence_last_value()
- 멀티라인 SQL과 \COPY 같이 사용하려면 : TEMPORARY VIEW 또는 COPY와 \g 이용
- 자동 생성 키에 값 설정 막기 : 테이블 생성시 GENERATED ALWAYS 지정 (GENERATED BY DEFAULT 대신)
- Pivot 테이블 만들기 : pandas.pivot_table, \crosstabview 또는 tablefunc 확장 사용
- Dollar Quoting
  ㅤ→ $$ 와 $$ 사이의 모든 글자는 문자열로 인식
  ㅤ→ $JSON$ / $function$ 처럼 안에 Tag이용 가능
  ㅤ→ ::jsonb 를 붙이면 빠르게 jsonb객체 생성
- DB객체에 코멘트 달기 : COMMENT ON TABLE/COLUMN, Dollar Quoting 이용해서 긴 문자열 설명 추가도 가능
  ㅤ→ 작성 : COMMENT ON TABLE sale IS 'Sales made in the system';
  ㅤ→ 보기 : \d+ sale
- DB 별 History 별도로 기록하기
  ㅤ→ \set HISTFILE ~/.psql_history- :DBNAME
- 자동완성을 대문자로 하기 : \set COMP_KEYWORD_CASE upper
- 슬립 주기 : pg_sleep(초), pg_sleep_for('4 minutes 14 seconds')
- 서브 쿼리 없이 그룹의 첫/마지막 줄 가져오기 : DISTINCT ON (그룹 컬럼)
- uuid-ossp 확장없이 UUID 생성하기 : gen_random_uuid() version 4 UUID를 생성
- 재현 가능한 랜덤 데이터 생성 : setseed() 로 시드를 같게
- 기존 데이터를 즉시 검증하지 않고 조건 추가하기 : ALTER 할때 NOT VALID 로 조건만 먼저 추가하고, ALTER VALIDATE로 기존 데이터 검증은 따로 실행
- 오라클의 Synonym 같은 기능을 이용하기 : search_path 변경 (Zero Downtime Migration시 유용)
- 겹치는 Range 찾기 : OVERLAPS 연산자

	