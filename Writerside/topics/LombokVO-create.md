# LombokVO생성

1. IntelliJ IDEA 설치
2. IntelliJ IDEA 의 Database plugin 설치
3. Database 연동
4. 테이블 우클릭
5. Tools > Scripted Extenstions > Go to Scriptes Directory
6. Generate POJOs.groovy 선택

```
import com.intellij.database.model.DasTable
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil

/*
* Available context bindings:
*   SELECTION   Iterable<DasObject>
*   PROJECT     project
*   FILES       files helper
    */
    packageName = "_packageName_;"
    typeMapping = [
    (~/(?i)int/)                      : "int",
    (~/(?i)float|double|decimal|real/): "double",
    (~/(?i)datetime|timestamp/)       : "String",
    (~/(?i)date/)                     : "String",
    (~/(?i)time/)                     : "String",
    (~/(?i)/)                         : "String"
    ]

FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
SELECTION.filter { it instanceof DasTable }.each { generateVo(it, dir) }
SELECTION.filter { it instanceof DasTable }.each { generateSql(it, dir) }
}

def generateVo(table, dir) {
def className = javaName(table.getName(), true)
def fields = calcFields(table)
def folderName = "${dir}/model"
def folder = new File(folderName)
if (!folder.exists()) {
folder.mkdirs()
}
new File(folderName, className + ".java").withPrintWriter { out -> generateVo(out, className, fields) }
}

def generateSql(table, dir) {
def camelClassName = javaName(table.getName(), true)
def className = table.getName()
def fields = calcFields(table)
def folderName = "${dir}/sql"
def folder = new File(folderName)
if (!folder.exists()) {
folder.mkdirs()
}
new File(folder, className + ".xml").withPrintWriter { out -> generateSql(out, camelClassName, className, fields) }
}

def generateVo(out, className, fields) {
out.println "package $packageName"
out.println ""
out.println "import lombok.*;"
out.println "import com.fasterxml.jackson.annotation.JsonInclude;"
out.println "import io.swagger.v3.oas.annotations.media.Schema;"
out.println ""
out.println "@AllArgsConstructor(access = AccessLevel.PRIVATE)"
out.println "@NoArgsConstructor"
out.println "@Data"
out.println "@Builder"
out.println "@JsonInclude(JsonInclude.Include.NON_NULL)"
out.println "public class $className {"
out.println ""
fields.each() {
if (it.annos != "") out.println "  ${it.annos}"
out.println "\t@Schema(description = \"${it.comment}\", example = \"${it.comment}\")"
out.println "\tprivate ${it.type} ${it.camelName};"
out.println ""
}
out.println "}"
}

def generateSql(out, camelClassName, className, fields) {
out.println '<?xml version="1.0" encoding="UTF-8"?>'
out.println '<!DOCTYPE mapper'
out.println '\t\tPUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"'
out.println '\t\t"http://mybatis.org/dtd/mybatis-3-mapper.dtd">'
out.println "<mapper namespace=\"_namespace_\">"

    out.println ""
    out.println "\t<select id=\"select${camelClassName}\" parameterType=\"\" resultType=\"\">"
    out.println "\t\t/* Query ID : _namespace_.select${camelClassName} */"
    out.println "\t\tSELECT "
    def index = 0;

    fields.each() {
        if (index != 0) {
            out.println "\t\t\t,${Case.UPPER.apply(it.name)}"
        } else {
            index = 1;
            out.println "\t\t\t${Case.UPPER.apply(it.name)}"
        }
    }
    out.println "\t\tFROM ${Case.UPPER.apply(className)}"
    out.println "\t\tWHERE 1=1"
    out.println "\t</select>"


    out.println ""
    out.println "\t<insert id=\"insert${camelClassName}\" parameterType=\"\">"
    out.println "\t\t/* Query ID : _namespace_.insert${camelClassName} */"
    out.println "\t\tINSERT INTO ${Case.UPPER.apply(className)} ("
    index = 0;
    fields.each() {
        if (index != 0) {
            out.println "\t\t\t,${Case.UPPER.apply(it.name)}"
        } else {
            index = 1;
            out.println "\t\t\t${Case.UPPER.apply(it.name)}"
        }
    }
    out.println "\t\t) VALUES ("
    index = 0;
    fields.each() {
        if (index != 0) {
            out.println "\t\t\t,#{${it.camelName}}"
        } else {
            index = 1;
            out.println "\t\t\t#{${it.camelName}}"
        }
    }
    out.println "\t\t)"
    out.println "\t</insert>"

    out.println ""
    out.println "\t<update id=\"update${camelClassName}\" parameterType=\"\">"
    out.println "\t\t/* Query ID : _namespace_.update${camelClassName} */"
    out.println "\t\tUPDATE ${Case.UPPER.apply(className)} SET "
    index = 0;

    fields.each() {
        if (index != 0) {
            out.println "\t\t\t,${Case.UPPER.apply(it.name)} = #{${it.camelName}}"
        } else {
            index = 1;
            out.println "\t\t\t${Case.UPPER.apply(it.name)} = #{${it.camelName}}"
        }
    }
    out.println "\t\tWHERE 1=1"
    out.println "\t</update>"


    out.println '</mapper>'
}

def calcFields(table) {
DasUtil.getColumns(table).reduce([]) { fields, col ->
def spec = Case.LOWER.apply(col.getDasType().getSpecification())
def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
fields += [[
camelName: javaName(col.getName(), false),
name     : col.getName(),
type     : typeStr,
comment  : col.getComment(),
annos    : ""]]
}
}

def javaName(str, capitalize) {
def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
.collect { Case.LOWER.apply(it).capitalize() }
.join("")
.replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}
```

[made by yt.seo](https://rundevelrun.github.io)

7. groovy 실행
8. 생성 디렉토리 확정





