---
layout:     post
title:      "Mybatis Generator Usage"
subtitle:   ""
date:       2017-8-18
author:     "GeniusV"
header-img: "img/post-bg-2015.jpg"
catalog: false
tags:
    - Mybatis
    - Java
---

<h2>Table of Content</h2>
<!-- MarkdownTOC -->

- [Main Structure](#main-structure)
    - [`validate` method](#validate-method)
    - [`sqlMapDocumentGenerated` method](#sqlmapdocumentgenerated-method)
    - [`clientGenerated` method](#clientgenerated-method)
- [How to run it](#how-to-run-it)
- [Changelog](#changelog)
- [Contact](#contact)

<!-- /MarkdownTOC -->

## Main Structure

To customize Mybatis Generator, we just need to extends `org.mybatis.generator.api.PluginAdapter`. Following methos must be implemented:

- [`public boolean validate(List<String> list)` ](#validate-method) 
- [`public boolean sqlMapDocumentGenerated(Document document, IntrospectedTable introspectedTable)` ](#sqlmapdocumentgenerated-method)
- [`public boolean clientGenerated(Interface interfaze, TopLevelClass topLevelClass, IntrospectedTable introspectedTable)`](#clientgenerated-method)

Go to [MyBatis Generator – Introduction to MyBatis Generator](http://www.mybatis.org/generator/) for more usage.

APIs: [org.mybatis.generator:mybatis-generator-core:1.3.5 API Doc :: Javadoc.IO](http://org.mybatis.generator:mybatis-generator-core:1.3.5 API Doc :: Javadoc.IO) 

### `validate` method

This method determins if the generator will run by return true or false.   Theoretically, we can determind which table can use this generator but I had never tried yet.

I just directly return true.

### `sqlMapDocumentGenerated` method 

This method determins how to generate our custom method xml files.

It is recommended to separate all statements in to methods by element. Then add the element to the root element.Like this:  

``` java
    @Override
    public boolean sqlMapDocumentGenerated(Document document, IntrospectedTable introspectedTable) {

//        selectPrimaryKeyLimitedByExample, this custom method
        XmlElement selectPrimaryKeyLimitedByExample = generateSelectLimitedPrimaryKeyByExample(introspectedTable);


        //add all elements
        XmlElement parentElement = document.getRootElement();
        parentElement.addElement(select);
        parentElement.addElement(cacheElement);
        parentElement.addElement(selectPrimaryKeyLimitedByExample);


        return super.sqlMapDocumentGenerated(document, introspectedTable);
    }
```

Here is the custom method:

``` java
    public XmlElement generateSelectLimitedPrimaryKeyByExample(IntrospectedTable introspectedTable) {
//          <select id="selectPrimaryKeyLimitedByExample" parameterType="map" resultType="java.lang.Long">
        XmlElement select = new XmlElement("select");
        select.addAttribute(new Attribute("id", "selectPrimaryKeyLimitedByExample"));
        select.addAttribute(new Attribute("parameterType", "map"));
        select.addAttribute(new Attribute("resultType", introspectedTable.getPrimaryKeyColumns().get(0).getFullyQualifiedJavaType().toString()));

        //        select
        select.addElement(new TextElement("select"));

//          <if test="example.distinct != null">
//          distinct
//           </if>
        XmlElement ifDinstinct = new XmlElement("if");
        ifDinstinct.addAttribute(new Attribute("test", "example.distinct != null"));
        ifDinstinct.addElement(new TextElement("distinct"));
        select.addElement(ifDinstinct);

//      id from v_role
        select.addElement(new TextElement(introspectedTable.getPrimaryKeyColumns().get(0).getActualColumnName() + " from " + introspectedTable.getAliasedFullyQualifiedTableNameAtRuntime()));

//    <if test="_parameter != null">
//      <include refid="Update_By_Example_Where_Clause" />
//    </if>
        XmlElement ifElement = new XmlElement("if");
        ifElement.addAttribute(new Attribute("test", "_parameter != null"));
        XmlElement includeElement = new XmlElement("include");
        includeElement.addAttribute(new Attribute("refid",
                introspectedTable.getMyBatis3UpdateByExampleWhereClauseId()));
        ifElement.addElement(includeElement);
        select.addElement(ifElement);

//    <if test="example.orderByClause != null">
//                order by ${example.orderByClause}
//    </if>
        ifElement = new XmlElement("if");
        ifElement.addAttribute(new Attribute("test", "example.orderByClause != null"));  //$NON-NLS-2$
        ifElement.addElement(new TextElement("order by ${example.orderByClause}"));
        select.addElement(ifElement);

//    <if test="offset != null and num != null">
//                limit #{offset}, #{num}
//    </if>
        ifElement = new XmlElement("if");
        ifElement.addAttribute(new Attribute("test", "offset != null and num != null"));  //$NON-NLS-2$
        ifElement.addElement(new TextElement("limit #{offset}, #{num}"));
        select.addElement(ifElement);

        return select;
    }
```

### `clientGenerated` method

This method is used to generate mapper interfaces. 
Similarly, it is recommended to separate statements into methods by interface method.

``` java
 @Override
    public boolean clientGenerated(Interface interfaze, TopLevelClass topLevelClass, IntrospectedTable introspectedTable) {

        interfaze.addMethod(generateSelectPrimaryKeyLimitedByExample());

                //add @Repository
                interfaze.addImportedType(new FullyQualifiedJavaType("org.springframework.stereotype.Repository"));
        interfaze.addAnnotation("@Repository");
        return true;
    }
```

Here is the custom method:

```java
public Method generateSelectPrimaryKeyLimitedByExample(){
    //        selectPrimaryKeyLimitedByExample
        Method selectPrimaryKeyLimitedByExample = new Method();
        selectPrimaryKeyLimitedByExample.setName("selectPrimaryKeyLimitedByExample");
        selectPrimaryKeyLimitedByExample.setVisibility(JavaVisibility.DEFAULT);
        selectPrimaryKeyLimitedByExample.setReturnType(returnType);
        Parameter offset = new Parameter(new FullyQualifiedJavaType("Long"), "offset", "@Param(\"offset\")");
        Parameter num = new Parameter(new FullyQualifiedJavaType("Long"), "num", "@Param(\"num\")");
        Parameter example = new Parameter(new FullyQualifiedJavaType(introspectedTable.getExampleType()), "example", "@Param(\"example\")");
        selectPrimaryKeyLimitedByExample.addParameter(offset);
        selectPrimaryKeyLimitedByExample.addParameter(num);
        selectPrimaryKeyLimitedByExample.addParameter(example);
        return selectPrimaryKeyLimitedByExample;
}
```

To use annotations or other type of date typies, remember to use `interfaze.addImportedType(new FullyQualifiedJavaType("your.type"))` to add import statements.

## How to run it 

Firstly, create a `generatorConfig.xml`, here is the example:

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <classPathEntry
            location="/Users/GeniusV/.m2/repository/mysql/mysql-connector-java/5.1.39/mysql-connector-java-5.1.39.jar"/>

    <context id="mysqlgenerator" targetRuntime="MyBatis3">

        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <plugin type="dao.SelectPrimaryKeyByExamplePlugin"/>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/Cache_Demo"
                        userId="root"
                        password="123456" />

        <javaModelGenerator targetPackage="io.github.geniusv.dao.model" targetProject="src/main/java" />
        <sqlMapGenerator targetPackage="io.github.geniusv.dao.mapper" targetProject="src/main/resources" />
        <javaClientGenerator type="XMLMAPPER" targetPackage="io.github.geniusv.dao.mapper" targetProject="src/main/java" />

        <!--<javaModelGenerator targetPackage="dao.model" targetProject="src/main/java" />-->
        <!--<sqlMapGenerator targetPackage="dao.mapper" targetProject="src/main/resources" />-->
        <!--<javaClientGenerator type="XMLMAPPER" targetPackage="dao.mapper" targetProject="src/main/java" />-->

        <table tableName="v_user" domainObjectName="User"/>
        <table tableName="v_role" domainObjectName="Role"/>
        <table tableName="v_user_role" domainObjectName="UserRole"/>
    </context>
</generatorConfiguration>
```

Go to [MyBatis Generator – MyBatis Generator XML Configuration File Reference](http://www.mybatis.org/generator/configreference/xmlconfig.html) for more confiuration information.

There are serveral ways to run the generator, but I recommend to run by JunitTest.We just have to put the `generatorConfig.xml` in `resources` directory and run the test like this:

```java
public class MappersGenerateTest {
    private File configFile;

    @Before
    public void before() {
        configFile = new File("/Users/GeniusV/Documents/Java/IdeaProjects/Cache-Demo/src/main/resources/generatorConfig.xml");

    }

    @Test
    public void test() throws Exception{
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```

## Changelog

- 2017/07/16
    + create file

## Contact

Email: eliot310100@outlook.com


