# 개발참고2

1. Local Hosts파일 수정
    - 파일경로 : C:\Window\System32\drivers\etc\hosts 파일 수정 (본인 IP, 개발 WAS IP 추가)
      10.94.13.78      amsdev.lgensol.com   #자산관리 개발서버
      hosts 파일 변경 후 적용 스크립트 : ipconfig /flushdns

개발WebServer
1) Server Info -> Diretory -> Require 수정
    - Required에 포함되어 있지 않은 Request Method는 “403“ Error발생 함.
    - OPTIONS : CORS관련 Request Method
2) Load Balancer Back-End API URL 설정
- Connector (TAB) -> URI Pattern Group
- 추가된 URI로 들어오는 요청은 WAS로 전달 됨.
  ※ Back-End의 RestAPI URI 와 Front-End의 Rounter Path가 겹치지 않도록 개발시 주의
3) ConfigTree ->httpd.conf 파일 수정
- 아래 붉은글씨 내용 추가
- React는 index.html을 통해 Router에 등록된 화면 Path를 찾아 화면을 Display하는 방식.
  -> 새로고침시 현재 열고있는 화면의 path로 요청되어 404 Error가 발생함.
  -> 아래 설정을 추가하여 Issue 해결
# Some examples:
#ErrorDocument 500 "The server made a boo boo."
#ErrorDocument 404 /missing.html
#ErrorDocument 404 "/cgi-bin/missingHandler.pl"
ErrorDocument 404 /index.html
#ErrorDocument 402 http://www.example.com/subscriptionInfo.html
#


개발 WAS
1) Config Tree -> Web.xml
    - REST API에서 사용할 Request Method 확인 후 < security-constraint> 주석필요
2) Config Tree -> Web.xml
    - Cors Filter 추가

   < filter>
    < filter-name>CorsFilter< /filter-name>
    < filter-class>org.apache.catalina.filters.CorsFilter< /filter-class>
< /filter>
< filter-mapping>
    < filter-name>CorsFilter< /filter-name>
    < url-pattern>/*< /url-pattern>
< /filter-mapping>


####################################################

노트패드++
스네이크 to 카멜
1. 전체 소문자로 변경
2. 바꾸기(Ctrl + F)에서 정규식 선택
3. 찾을내용 : [_]{1,1}([a-z])
4. 바꿀내용 : \u$1

카멜 to 스네이크
1. 찾을내용 : (\b[a-z]+|\G(?!^))((?:[A-Z]|\d+)[a-z]*)
2. 바꿀내용 : \U\1_\2
3. 카멜 적용안돼있는 경우는 소문자로 남음

맨 앞에 텍스트 넣기
1. 찾을내용 : ^
2. 바꿀내용 : ;

맨 뒤에 텍스트 넣기
1. 찾을내용 : $
2. 바꿀내용 : ;

####################################################

VO 생성쿼리

SELECT RPAD(' ', 4) || 'private ' ||
CASE
WHEN A.DATA_TYPE = 'VARCHAR2' THEN 'String'
WHEN A.DATA_TYPE = 'NUMBER' THEN 'Integer'
WHEN A.DATA_TYPE = 'FLOAT' THEN 'Float'
WHEN A.DATA_TYPE = 'CHAR' AND A.DATA_LENGTH > 1 THEN 'String'
WHEN A.DATA_TYPE = 'DATE' THEN 'Date'
ELSE 'Object'
END ||
' ' ||
CONCAT
(
LOWER(SUBSTR(B.COLUMN_NAME, 1, 1)),
SUBSTR(REGEXP_REPLACE(INITCAP(B.COLUMN_NAME), ' |_'), 2)
) || CHR(59) || CHR(13)
FROM   ALL_TAB_COLUMNS A
, ALL_COL_COMMENTS B
WHERE  A.TABLE_NAME = B.TABLE_NAME
AND    A.COLUMN_NAME = B.COLUMN_NAME
AND    A.OWNER = 'ADMIN' -- 오너명
AND    B.OWNER = 'ADMIN' -- 오너명
AND    A.TABLE_NAME = 'TB_VARS_MNU_ACES_LOG_M' -- 테이블명
ORDER BY A.COLUMN_ID;

####################################################

VO 만들기 COMMENT 확인

SELECT a.COLUMN_NAME ,b.COMMENTS , a.COLUMN_ID  
FROM USER_TAB_COLUMNS a
INNER JOIN ALL_COL_COMMENTS b
ON a.COLUMN_NAME = b.COLUMN_NAME AND a.TABLE_NAME = b.TABLE_NAME  
WHERE a.TABLE_NAME = 'TB_VARS_VDCP_INFO_MGT'
ORDER BY a.COLUMN_ID ;
--
SELECT a.COLUMN_NAME, a.DATA_TYPE, DECODE(a.NULLABLE, 'Y', 'N', 'N', 'Y', ''), A.CHAR_COL_DECL_LENGTH ,b.COMMENTS , a.COLUMN_ID  
FROM USER_TAB_COLUMNS a
INNER JOIN ALL_COL_COMMENTS b
ON a.COLUMN_NAME = b.COLUMN_NAME AND a.TABLE_NAME = b.TABLE_NAME  
WHERE a.TABLE_NAME = 'TB_VARS_ACCN_INFO_MGT'
ORDER BY a.COLUMN_ID;


####################################################
Oracle Object Dependency 조회 쿼리

/* DB Ojbect break Check */
SELECT 'ALTER '||A.OBJECTTYPE||' '||A.OBJECTNAME||' COMPILE;' AS COMPILESCRIPT
, A.OBJECTTYPE
, A.OBJECTNAME
, A.STATUS
FROM USEROBJECTS A
, USERDEPENDENCIES B
WHERE A.OBJECTNAME = B.NAME
AND B.REFERENCEDNAME = 'TBAAMSEASSTM';

STATUS 확인해서 INVALID로 표시되는 OBJECT는 맨앞의 compileScript 복사해서 스크립트 실행.

####################################################

/* Table Lock  확인 */
SELECT A.OBJECTID
, A.SESSIONID
, A.ORACLEUSERNAME
, A.OSUSERNAME
FROM V$LOCKEDOBJECT A;

####################################################


/*Session Monitoring*/

SELECT 'begin

rdsadmin.rdsadminUtil.kill(

sid => '||A.SID||',

serial => '||A.SERIAL#||');

end;' AS KILLSCRIPT

, A.SID

, A."SERIAL#"

, A.SQLID

, A.USERNAME

, A.STATUS

-- , A.SCHEMANAME

, A.OSUSER

-- , A.MACHINE

-- , DECODE(A.SQLEXECSTART,NULL, A.PREVEXECSTART, A.SQLEXECSTART) SQLEXECSTART

-- , A.PREVEXECID

, A.MODULE

, B.SQLFULLTEXT

FROM v$session A

, V$SQL B

WHERE 1=1

AND A.SQLID = B.SQLID

-- AND A.MODULE != 'JDBC Thin Client'

AND A.SQLID IS NOT NULL

-- AND TYPE = 'USER'

AND STATUS ='ACTIVE'

ORDER BY A.OSUSER, A.MODULE ;

첫 컬럼 킬스크립트 복사후 우클릭, 스크립트 실행(Alt + x)

####################################################

린트 패스
/* eslint-disable */
// eslint-disable-next-line

타입스크립트 전역변수
declare const 변수명: any;

####################################################


-- VO생성
SELECT RPAD(' ', 4) || 'private ' ||
CASE
WHEN A.DATA_TYPE = 'VARCHAR2' THEN 'String'
WHEN A.DATA_TYPE = 'NUMBER' THEN 'Integer'
WHEN A.DATA_TYPE = 'FLOAT' THEN 'Float'
WHEN A.DATA_TYPE = 'CHAR' AND A.DATA_LENGTH > 1 THEN 'String'
WHEN A.DATA_TYPE = 'DATE' THEN 'Date'
ELSE 'Object'
END ||
' ' ||
CONCAT
(
LOWER(SUBSTR(B.COLUMN_NAME, 1, 1)),
SUBSTR(REGEXP_REPLACE(INITCAP(B.COLUMN_NAME), ' |_'), 2)
) || CHR(59) || CHR(13)
FROM   ALL_TAB_COLUMNS A
, ALL_COL_COMMENTS B
WHERE  A.TABLE_NAME = B.TABLE_NAME
AND    A.COLUMN_NAME = B.COLUMN_NAME
AND    A.OWNER = 'ADMIN' -- 오너명
AND    B.OWNER = 'ADMIN' -- 오너명
AND    A.TABLE_NAME = 'TB_VARS_VDCP_INFO_MGT' -- 테이블명
ORDER BY A.COLUMN_ID;

--COMMENT 확인
SELECT a.COLUMN_NAME ,b.COMMENTS , a.COLUMN_ID  
FROM USER_TAB_COLUMNS a
INNER JOIN ALL_COL_COMMENTS b
ON a.COLUMN_NAME = b.COLUMN_NAME AND a.TABLE_NAME = b.TABLE_NAME  
WHERE a.TABLE_NAME = 'TB_VARS_VDCP_INFO_MGT'
ORDER BY a.COLUMN_ID;

--DATATYPE 확인
SELECT a.COLUMN_NAME, a.DATA_TYPE, DECODE(a.NULLABLE, 'Y', 'N', 'N', 'Y', ''), A.CHAR_COL_DECL_LENGTH ,b.COMMENTS , a.COLUMN_ID  
FROM USER_TAB_COLUMNS a
INNER JOIN ALL_COL_COMMENTS b
ON a.COLUMN_NAME = b.COLUMN_NAME AND a.TABLE_NAME = b.TABLE_NAME  
WHERE a.TABLE_NAME = 'TB_VARS_ACCN_INFO_MGT'
ORDER BY a.COLUMN_ID ;





################# ORACLE 관련 [S] #################
-- session Kill --
SELECT 'begin
rdsadmin.rdsadmin_util.kill(
sid => '||A.SID||',
serial => '||A.SERIAL#||');
end;' AS KILL_SCRIPT
, A.SID
, A."SERIAL#"
, A.SQL_ID
, A.USERNAME
, A.STATUS
-- , A.SCHEMANAME
, A.OSUSER
-- , A.MACHINE
-- , DECODE(A.SQL_EXEC_START,NULL, A.PREV_EXEC_START, A.SQL_EXEC_START) SQL_EXEC_START
-- , A.PREV_EXEC_ID
, A.MODULE
, B.SQL_FULLTEXT
FROM v$session A , V$SQL B
WHERE 1=1
AND A.SQL_ID = B.SQL_ID
-- AND A.MODULE != 'JDBC Thin Client'
AND A.SQL_ID IS NOT NULL
-- AND TYPE = 'USER'
AND STATUS = 'ACTIVE'
ORDER BY A.OSUSER, A.MODULE ;


-- Object Dependency --
SELECT 'ALTER '||A.OBJECT_TYPE||' '||A.OBJECT_NAME||' COMPILE;' AS COMPILESCRIPT
, A.OBJECT_TYPE
, A.OBJECT_NAME
, A.STATUS
, B.REFERENCED_NAME
FROM USER_OBJECTS A
, USER_DEPENDENCIES B
WHERE A.OBJECT_NAME = B.NAME
AND A.STATUS = 'INVALID';
--   AND B.REFERENCED_NAME = 'TBAAMSEASSTM';
################# ORACLE 관련 [E] #################

-- Table Lock 확인 --
SELECT A.OBJECT_ID
, A.SESSION_ID
, A.ORACLE_USERNAME
, A.OS_USER_NAME
FROM V$LOCKED_OBJECT A;

--- 열 만들기 참고 {1|2|3|4 data}---
SELECT DISTINCT A.*
, REGEXP_SUBSTR(A.WK_USER_ID_LST_CTN, '[^|]+', 1, LEVEL) AS WK_USER_ID
FROM AMS_ADM.TB_AAMSE_TODO_M A
WHERE A.USE_YN = 'Y'
AND TO_CHAR(A.TRNM_DTM, 'YYYYMMDD') BETWEEN '20231229' AND '20240129'
CONNECT BY INSTR(A.WK_USER_ID_LST_CTN, '|', 1, LEVEL -1) > 0



postgresql 16-2.1



console.log('%c  ${url}', 'background: #F00000; color: white');
console.log(newSideMenuList);


		
-----------------------------------------------------------------------------
//textarea 바이트 수 체크하는 함수
function fn_checkByte(obj){
const maxByte = 100; //최대 100바이트
const text_val = obj.value; //입력한 문자
const text_len = text_val.length; //입력한 문자수

    let totalByte=0;
    for(let i=0; i< text_len; i++){
    	const each_char = text_val.charAt(i);
        const uni_char = escape(each_char); //유니코드 형식으로 변환
        if(uni_char.length>4){
        	// 한글 : 2Byte
            totalByte += 2;
        }else{
        	// 영문,숫자,특수문자 : 1Byte
            totalByte += 1;
        }
    }
    
    if(totalByte>maxByte){
    	alert('최대 100Byte까지만 입력가능합니다.');
        	document.getElementById("nowByte").innerText = totalByte;
            document.getElementById("nowByte").style.color = "red";
        }else{
        	document.getElementById("nowByte").innerText = totalByte;
            document.getElementById("nowByte").style.color = "green";
        }
    }
}
-----------------------------------------------------------------------------

개선된FOR 로직 바이트 체크

function getByteLength(s, b, i, c) {
for(b=i=0; c=s.charCodeAt(i++); b+=c>>11?3:c>>7?2:1);
}

사용예??

var stringByteLength = (function(s,b,i,c){
for(b=i=0;c=s.charCodeAt(i++);b+=c>>11?3:c>>7?2:1);
return b
})(string);



####################################################
WSDL 생성
wsdl2java.bat -uri C:\workspace\erpm-be\src\main\java\com\lgensol\erpm\webService\client\empWebServiceClient\LGCY_LCHC_EA_EMPBATCH_02_LGCY_SOService.wsdl



#### DB ####

jdbc:log4jdbc:postgresql://rds-an2-prd-lesdsfwwapp01-a.cdakvggnfwys.ap-northeast-2.rds.amazonaws.com/WAPDB?useSSL=false&currentSchema=erpm



1. WAPDB에 인프라 CSR로 WAPDB에 유저 생성요청
   유저명 : meta_app
2. DB에이전트 등록할 아이디를 알아야대는디??? 멀까???



이수원책임 참조, 프로젝트 명, PM명


curl http://wap.lgensol.com/api/v1/mailBatch -d {}


--SELECT organization_code , COUNT(*)
--FROM tb_rpjte_hr_inno_dept_i
--GROUP BY organization_code
--HAVING COUNT(*) > 1;

GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO wsfapp;



<select id="selectAllDepartments" resultType="com.lginnotek.wsf.department.model.DepartmentVO">
        WITH RECURSIVE RCS_TB_RPJTE_DEPT_M AS (
        SELECT 1 AS level
        ,S.*
        ,C.CMN_CD_NM
        ,C.SORT_ORD
        FROM TB_RPJTE_DEPT_M_TEST S
        INNER JOIN TB_RPJTE_CMN_C C ON S.COP_CD = C.CMN_CD
        AND C.CMN_GR_CD = 'COP_CD'
        WHERE S.USE_YN = 'Y'
        AND S.UPPR_DEPT_CD IS NULL
        AND S.UPPR_GRP_DEPT_CD = ''
        AND SUBSTRING(S.GRP_DEPT_CD, 1, 1) = SUBSTRING(C.CMN_CD, 5, 1)
        UNION ALL
        SELECT LEVEL + 1 AS LEVEL
        ,S.*
        ,C.CMN_CD_NM
        ,C.SORT_ORD
        FROM RCS_TB_RPJTE_DEPT_M R
        INNER JOIN TB_RPJTE_DEPT_M_TEST S ON r.DEPT_CD = s.UPPR_DEPT_CD
        INNER JOIN TB_RPJTE_CMN_C C ON S.COP_CD = C.CMN_CD
        AND C.CMN_GR_CD = 'COP_CD'
        )
        SELECT 1 AS LEVEL
              ,'' AS DEPT_CD
              ,'LGINNOTEK' AS COP_CD
              ,'LGINNOTEK' AS COP_CD_NM
              ,'LGINNOTEK' AS deptnm
              ,'' AS UPPR_DEPT_CD
              ,'Y' AS USE_YN
              ,0 AS SORT_ORD
        UNION
        SELECT LEVEL
              ,DEPT_CD
              ,COP_CD
              ,CMN_CD_NM AS COP_CD_NM
              <if test='langCd != null and langCd != "" and langCd.equals("ko")'>
                ,DEPT_NM AS deptNm
              </if>
              <if test='langCd != null and langCd != "" and (langCd.equals("en") or langCd.equals("pl"))'>
                ,DEPT_ENG_NM AS deptNm
              </if>
              <if test='langCd != null and langCd != "" and (langCd.equals("zhC") or langCd.equals("zhT"))'>
                ,DEPT_CNG_NM AS deptNm
              </if>
              ,CASE WHEN UPPR_DEPT_CD IS NULL THEN '0' ELSE UPPR_DEPT_CD END AS UPPR_DEPT_CD
              ,USE_YN
              ,SORT_ORD
        FROM RCS_TB_RPJTE_DEPT_M
        WHERE USE_YN = 'Y'
        <if test='searchItem != null and searchItem != ""'>
            AND (
            UPPER(DEPT_CD) LIKE UPPER(CONCAT('%'||#{searchItem},'%'))
            OR UPPER(DEPT_NM) LIKE UPPER(CONCAT('%'||#{searchItem},'%'))
            OR UPPER(DEPT_ENG_NM) LIKE UPPER(CONCAT('%'||#{searchItem},'%'))
            OR UPPER(DEPT_CNG_NM) LIKE UPPER(CONCAT('%'||#{searchItem},'%'))
            OR UPPER(CMN_CD_NM) LIKE UPPER(CONCAT('%'||#{searchItem},'%'))
            )
        </if>
        ORDER BY LEVEL, SORT_ORD, COP_CD
    </select>