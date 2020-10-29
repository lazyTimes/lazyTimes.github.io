---
title: 使用Go语言+IDEA+mysql 逆向生成dao 和 domain 的方法
subtitle: 学习数据结构的内容-韩顺平
author: lazytime
url_suffix: goidea
tags:
  - 数据结构
categories:
  - 笔记
keywords: 数据结构的一些遗留笔记
description: 学习数据结构的内容-韩顺平
copyright: true
date: 2020-10-27 22:53:01
---

# 参考地址：

https://www.cnblogs.com/gaomanito/p/10682260.html

<!-- more -->

# 使用方式

## 第一步：安装idea:自行百度

## 第二步：Idea配置mysql 连接

+ View -> Tool Windows -> DataSource 出现数据连接页面
+ 点击 **+** 号，出现数据库连接设置，
  + name: 连接名称
  + comments: 备注
  + host:本地或者内网ip
  + user:用户名
  + password：密码
  + Database：连接数据库名称
  + url: jdbc连接
    + 注意此处如果是mysql 5.5 以上 的版本需要加入如下参数
    + **serverTimezone=UTC**

## 第三步：使用groovy逆向生成model, dao, daoxml 三种类型的文件

+ 右击连接：选中`Scripted Extendsions` 
+ 选择`Go to Scripts Directory` 调转到go 语言的模板地址
+ 如果需要扩展则只需要在该文件夹内加入对应的模板即可



## 第四步：几个默认的模板

### Generate MyPOJOs.groovy文件（生成实体类）：

```go
import com.intellij.database.model.DasTable
import com.intellij.database.model.ObjectKind
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil
import java.io.*
import java.text.SimpleDateFormat

/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */
packageName = ""
typeMapping = [
        (~/(?i)tinyint|smallint|mediumint/)      : "Integer",
        (~/(?i)int/)                             : "Long",
        (~/(?i)bool|bit/)                        : "Boolean",
        (~/(?i)float|double|decimal|real/)       : "Double",
        (~/(?i)datetime|timestamp|date|time/)    : "Date",
        (~/(?i)blob|binary|bfile|clob|raw|image/): "InputStream",
        (~/(?i)/)                                : "String"
]


FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable && it.getKind() == ObjectKind.TABLE }.each { generate(it, dir) }
}

def generate(table, dir) {
    //def className = javaClassName(table.getName(), true)
    def className = javaName(table.getName(), true)
    def fields = calcFields(table)
    packageName = getPackageName(dir)
    PrintWriter printWriter = new PrintWriter(new OutputStreamWriter(new FileOutputStream(new File(dir, className + ".java")), "UTF-8"))
    printWriter.withPrintWriter {out -> generate(out, className, fields,table)}

//    new File(dir, className + ".java").withPrintWriter { out -> generate(out, className, fields,table) }
}

// 获取包所在文件夹路径
def getPackageName(dir) {
    return dir.toString().replaceAll("\\\\", ".").replaceAll("/", ".").replaceAll("^.*src(\\.main\\.java\\.)?", "") + ";"
}

def generate(out, className, fields,table) {
    def tableName = table.getName()
    out.println "package $packageName"
    out.println ""
    out.println "import javax.persistence.Column;"
    out.println "import javax.persistence.Entity;"
    out.println "import javax.persistence.Table;"
    out.println "import java.io.Serializable;"
    out.println "import lombok.Data;"
    out.println "import lombok.AllArgsConstructor;"
    out.println "import lombok.Builder;"
    out.println "import lombok.NoArgsConstructor;"

    Set types = new HashSet()

    fields.each() {
        types.add(it.type)
    }

    if (types.contains("Date")) {
        out.println "import java.util.Date;"
    }

    if (types.contains("InputStream")) {
        out.println "import java.io.InputStream;"
    }
    out.println ""
    out.println "/**\n" +
            " * @Description  \n" +
            " * @Author  GX\n" +
            " * @Date "+ new SimpleDateFormat("yyyy-MM-dd").format(new Date()) + " \n" +
            " */"
    out.println ""
    out.println "@Data"
    out.println "@Entity"
    out.println "@AllArgsConstructor"
    out.println "@Builder"
    out.println "@NoArgsConstructor"
    out.println "@Table ( name =\""+table.getName() +"\" )"
    out.println "public class $className  implements Serializable {"
    out.println genSerialID()


    // 判断自增
    if ((tableName + "_id").equalsIgnoreCase(fields[0].colum) || "id".equalsIgnoreCase(fields[0].colum)) {
        out.println "\t@Id"
        out.println "\t@GeneratedValue(generator = \"idGenerator\")"
        out.println "\t@GenericGenerator(name = \"idGenerator\", strategy = ChiticCoreConstant.ID_GENERATOR_COMMON)"
    }


    fields.each() {
        out.println ""
        // 输出注释
        if (isNotEmpty(it.commoent)) {
            out.println "\t/**"
            out.println "\t * ${it.commoent.toString()}"
            out.println "\t */"
        }

        if (it.annos != "") out.println "   ${it.annos.replace("[@Id]", "")}"

        // 输出成员变量
        out.println "\tprivate ${it.type} ${it.name};"
    }

    // 输出get/set方法
//    fields.each() {
//        out.println ""
//        out.println "\tpublic ${it.type} get${it.name.capitalize()}() {"
//        out.println "\t\treturn this.${it.name};"
//        out.println "\t}"
//        out.println ""
//
//        out.println "\tpublic void set${it.name.capitalize()}(${it.type} ${it.name}) {"
//        out.println "\t\tthis.${it.name} = ${it.name};"
//        out.println "\t}"
//    }
    out.println ""
    out.println "}"
}

def calcFields(table) {
    DasUtil.getColumns(table).reduce([]) { fields, col ->
        def spec = Case.LOWER.apply(col.getDataType().getSpecification())

        def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
        def comm =[
                colName : col.getName(),
                name :  javaName(col.getName(), false),
                type : typeStr,
                commoent: col.getComment(),
                annos: "\t@Column(name = \""+col.getName()+"\" )"]
        if("id".equals(Case.LOWER.apply(col.getName())))
            comm.annos +=["@Id"]
        fields += [comm]
    }
}

// 处理类名（这里是因为我的表都是以t_命名的，所以需要处理去掉生成类名时的开头的T，
// 如果你不需要那么请查找用到了 javaClassName这个方法的地方修改为 javaName 即可）
def javaClassName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    // 去除开头的T  http://developer.51cto.com/art/200906/129168.htm
    s = s[1..s.size() - 1]
    capitalize || s.length() == 1? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

def javaName(str, capitalize) {
//    def s = str.split(/(?<=[^\p{IsLetter}])/).collect { Case.LOWER.apply(it).capitalize() }
//            .join("").replaceAll(/[^\p{javaJavaIdentifierPart}]/, "_")
//    capitalize || s.length() == 1? s : Case.LOWER.apply(s[0]) + s[1..-1]
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    capitalize || s.length() == 1? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

def isNotEmpty(content) {
    return content != null && content.toString().trim().length() > 0
}

static String changeStyle(String str, boolean toCamel){
    if(!str || str.size() <= 1)
        return str

    if(toCamel){
        String r = str.toLowerCase().split('_').collect{cc -> Case.LOWER.apply(cc).capitalize()}.join('')
        return r[0].toLowerCase() + r[1..-1]
    }else{
        str = str[0].toLowerCase() + str[1..-1]
        return str.collect{cc -> ((char)cc).isUpperCase() ? '_' + cc.toLowerCase() : cc}.join('')
    }
}

static String genSerialID()
{
    return "\tprivate static final long serialVersionUID =  "+Math.abs(new Random().nextLong())+"L;"
}
```

### Generate Dao.groovy文件（生成dao）：

```go
package src

import com.intellij.database.model.DasTable
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil

/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */

packageName = "**;" // 需手动配置 生成的 dao 所在包位置

FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable }.each { generate(it, dir) }
}

def generate(table, dir) {
    def baseName = javaName(table.getName(), true)
    new File(dir, baseName + "Mapper.java").withPrintWriter { out -> generateInterface(out, baseName) }
}

def generateInterface(out, baseName) {
    def date = new Date().format("yyyy/MM/dd")
    out.println "package $packageName"
    out.println "import cn.xx.entity.${baseName}Entity;" // 需手动配置
    out.println "import org.springframework.stereotype.Repository;"
    out.println ""
    out.println "/**"
    out.println " * Created on $date."
    out.println " *"
    out.println " * @author GX" // 可自定义
    out.println " */"
    out.println "@Repository"
    out.println "public interface ${baseName}Dao extends BaseDao<${baseName}Entity> {" // 可自定义
    out.println ""
    out.println "}"
}

def javaName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    name = capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}
```

### Generate DaoXml.groovy文件（生成dao.xml）：

```go
package src

import com.intellij.database.model.DasTable
import com.intellij.database.util.Case
import com.intellij.database.util.DasUtil

/*
 * Available context bindings:
 *   SELECTION   Iterable<DasObject>
 *   PROJECT     project
 *   FILES       files helper
 */

// entity(dto)、mapper(dao) 与数据库表的对应关系在这里手动指明,idea Database 窗口里只能选下列配置了的 mapper
// tableName(key) : [mapper(dao),entity(dto)]
typeMapping = [
        (~/(?i)int/)                      : "INTEGER",
        (~/(?i)float|double|decimal|real/): "DOUBLE",
        (~/(?i)datetime|timestamp/)       : "TIMESTAMP",
        (~/(?i)date/)                     : "TIMESTAMP",
        (~/(?i)time/)                     : "TIMESTAMP",
        (~/(?i)/)                         : "VARCHAR"
]

basePackage = "com.chitic.bank.mapping" // 包名需手动填写

FILES.chooseDirectoryAndSave("Choose directory", "Choose where to store generated files") { dir ->
    SELECTION.filter { it instanceof DasTable }.each { generate(it, dir) }
}

def generate(table, dir) {
    def baseName = mapperName(table.getName(), true)
    def fields = calcFields(table)
    new File(dir, baseName + "Mapper.xml").withPrintWriter { out -> generate(table, out, baseName, fields) }
}

def generate(table, out, baseName, fields) {
    def baseResultMap = 'BaseResultMap'
    def base_Column_List = 'Base_Column_List'
    def date = new Date().format("yyyy/MM/dd")
    def tableName = table.getName()

    def dao = basePackage + ".dao.${baseName}Mapper"
    def to = basePackage + ".to.${baseName}TO"

    out.println mappingsStart(dao)
    out.println resultMap(baseResultMap, to, fields)
    out.println sql(fields, base_Column_List)
    out.println selectById(tableName, fields, baseResultMap, base_Column_List)
    out.println deleteById(tableName, fields)
    out.println delete(tableName, fields, to)
    out.println insert(tableName, fields, to)
    out.println update(tableName, fields, to)
    out.println selectList(tableName, fields, to, base_Column_List, baseResultMap)
    out.println mappingsEnd()

}

static def resultMap(baseResultMap, to, fields) {

    def inner = ''
    fields.each() {
        inner += '\t\t<result column="' + it.sqlFieldName + '" jdbcType="' + it.type + '" property="' + it.name + '"/>\n'
    }

    return '''\t<resultMap id="''' + baseResultMap + '''" type="''' + to + '''">
        <id column="id" jdbcType="INTEGER" property="id"/>
''' + inner + '''\t</resultMap>
'''
}

def calcFields(table) {
    DasUtil.getColumns(table).reduce([]) { fields, col ->
        def spec = Case.LOWER.apply(col.getDataType().getSpecification())
        def typeStr = typeMapping.find { p, t -> p.matcher(spec).find() }.value
        fields += [[
                           comment     : col.getComment(),
                           name        : mapperName(col.getName(), false),
                           sqlFieldName: col.getName(),
                           type        : typeStr,
                           annos       : ""]]
    }

}

def mapperName(str, capitalize) {
    def s = com.intellij.psi.codeStyle.NameUtil.splitNameIntoWords(str)
            .collect { Case.LOWER.apply(it).capitalize() }
            .join("")
            .replaceAll(/[^\p{javaJavaIdentifierPart}[_]]/, "_")
    name = capitalize || s.length() == 1 ? s : Case.LOWER.apply(s[0]) + s[1..-1]
}

// ------------------------------------------------------------------------ mappings
static def mappingsStart(mapper) {
    return '''<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="''' + mapper + '''">
'''
}

// ------------------------------------------------------------------------ mappings
static def mappingsEnd() {
    return '''</mapper>'''
}

// ------------------------------------------------------------------------ selectById
static def selectById(tableName, fields, baseResultMap, base_Column_List) {
    return '''
    <select id="selectById" parameterType="java.lang.Integer" resultMap="''' + baseResultMap + '''">
        select
        <include refid="''' + base_Column_List + '''"/>
        from ''' + tableName + '''
        where id = #{id}
    </select>'''
}

// ------------------------------------------------------------------------ insert
static def insert(tableName, fields, parameterType) {

    return '''
    <insert id="insert" parameterType="''' + parameterType + '''">
        insert into ''' + tableName + '''
        <trim prefix="(" suffix=")" suffixOverrides=",">
            ''' + testNotNullStr(fields) + '''
        </trim>
        <trim prefix="values (" suffix=")" suffixOverrides=",">
            ''' + testNotNullStrSet(fields) + '''
        </trim>
    </insert>
'''

}
// ------------------------------------------------------------------------ update
static def update(tableName, fields, parameterType) {

    return '''
    <update id="update" parameterType="''' + parameterType + '''">
        update ''' + tableName + '''
        <set>
            ''' + testNotNullStrWhere(fields) + '''
        </set>
        where id = #{id}
    </update>'''
}

// ------------------------------------------------------------------------ deleteById
static def deleteById(tableName, fields) {

    return '''
    <delete id="deleteById" parameterType="java.lang.Integer">
        delete
        from ''' + tableName + '''
        where id = #{id}
    </delete>'''
}

// ------------------------------------------------------------------------ delete
static def delete(tableName, fields, parameterType) {

    return '''
    <delete id="delete" parameterType="''' + parameterType + '''">
        delete from ''' + tableName + '''
        where 1 = 1
        ''' + testNotNullStrWhere(fields) + '''
    </delete>'''
}

// ------------------------------------------------------------------------ selectList
static def selectList(tableName, fields, parameterType, base_Column_List, baseResultMap) {

    return '''
    <select id="selectList" parameterType="''' + parameterType + '''" resultMap="''' + baseResultMap + '''">
        select
        <include refid="''' + base_Column_List + '''"/>
                from ''' + tableName + '''
        where 1 = 1
        ''' + testNotNullStrWhere(fields) + '''
        order by id desc
    </select>'''
}

// ------------------------------------------------------------------------ sql
static def sql(fields, base_Column_List) {
    def str = '''\t<sql id="''' + base_Column_List + '''">
        @inner@
    </sql> '''

    def inner = ''
    fields.each() {
        inner += ('\t\t' + it.sqlFieldName + ',\n')
    }

    return str.replace("@inner@", inner.substring(0, inner.length() - 2))

}

static def testNotNullStrWhere(fields) {
    def inner = ''
    fields.each {
        inner += '''
        <if test="''' + it.name + ''' != null">
            and ''' + it.sqlFieldName + ''' = #{''' + it.name + '''}
        </if>\n'''
    }

    return inner
}

static def testNotNullStrSet(fields) {
    def inner = ''
    fields.each {
        inner += '''
        <if test="''' + it.name + ''' != null">
            #{''' + it.name + '''},
        </if>\n'''
    }

    return inner
}

static def testNotNullStr(fields) {
    def inner1 = ''
    fields.each {
        inner1 += '''
        <if test = "''' + it.name + ''' != null" >
        \t''' + it.sqlFieldName + ''',
        </if>\n'''
    }

    return inner1
}
```