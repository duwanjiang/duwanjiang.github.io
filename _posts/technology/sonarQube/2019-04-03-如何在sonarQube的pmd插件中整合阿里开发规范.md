---
layout: 	post
title: 		"如何在sonarQube的pmd插件"
subtitle:	" \"如何在sonarQube的pmd插件\""
date:		2019-04-03 18:34:00
author:		"duwanjiang"
header-img:	"img/home-bg-o.jpg"
catalog: true
categories: 代码质量
tags:
    - sonarQube
---

> “如何在sonarQube的pmd插件 ”

# 1、sonarqube简介

Sonar是一个用于代码质量管理的开源平台，用于管理Java源代码的质量。通过插件机制，Sonar 可以集成不同的测试工具，代码分析工具，以及持续集成工具，比如pmd-cpd、checkstyle、findbugs、Jenkins。通过不同的插件对这些结果进行再加工处理，通过量化的方式度量代码质量的变化，从而可以方便地对不同规模和种类的工程进行代码质量管理。同时 Sonar 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用 Sonar。 此外，Sonar 的插件还可以对 Java 以外的其他编程语言提供支持，对国际化以及报告文档化也有良好的支持。



# 2、pmd插件整合阿里开发规则

sonar-pmd是sonar官方的支持pmd的插件，但是还不支持p3c，需要在pmd插件源码中添加p3c支持(p3c是阿里在pmd基础上根据阿里巴巴开发手册实现了其中的48开发规则)。

* 目前阿里巴巴的P3C_PMD项目地址如下：[阿里巴巴的P3C-PMD](https://github.com/alibaba/p3c/tree/master/p3c-pmd)

* 这里我参考了网上[mrprince/sonar-p3c-pmd](https://github.com/mrprince/sonar-p3c-pmd)的插件项目，我自己的项目地址是[duwanjiang/sonar-p3c-pmd](https://github.com/duwanjiang/sonar-p3c-pmd)，直接编写了sonarQube的pcm插件。此源码工程已经添加了P3C支持


``` xml
<dependency>
    <groupId>com.alibaba.p3c</groupId>
    <artifactId>p3c-pmd</artifactId>
    <version>1.3.6</version>
</dependency>
```

并在pmd插件的默认268条规则上添加了阿里的48条规则，阿里的p3c实现了49条阿里开发规范规则，目前此工程中少了一条AvoidManuallyCreateThreadRule规则，只需要将sonar-p3c-pmd工程中的默认的268条规则屏蔽掉，添加AvoidManuallyCreateThreadRule 规则，就是我们现在的含有49条阿里开发规则的 pmd插件。

## 2.1、sonar-p3c-pmd工程介绍
一条校验规则对应分别对应3个配置文件：

1. src\main\resources\org\sonar\l10n\ **pmd.properties** 
2. src\main\resources\org\sonar\plugins\ **pmd\rules.xml** 
3. src\main\resources\com\sonar\sqale\ **pmd-model.xml** 

### 2.1.1、屏蔽一条rule
屏蔽掉了一条默认的pmd规则，需要如下3步：

1. 首先注释掉pmd.properties文件中的“StringInstantiation”，如下图：

   ![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\4.png" | prepend:site.url}})

   * （注意：规则配置在pmd.properties文件中是以 .name 结尾，不要多注释掉其它的以.param.xxx结尾的配置，否则可能导致打出的jar包放到sonar下时sonar启动失败 ）

2. 注释掉rules.xml中的 StringInstantiation对应的配置，如下图：

   ![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\6.png" | prepend:site.url}})

3. 注释掉pmd-model.xml中的StringInstantiation对应的配置，如下图：

   ![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\3.png" | prepend:site.url}})

### 2.1.2、添加阿里的开发规则

为了区别阿里的p3c规则，这里新建了一个rules-p3c.xml文件，如下图：

![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\7.png" | prepend:site.url}})

然后在PmdRulesDefinition类中指定一下rules-p3c.xml 路径，如下图：

![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\8.png" | prepend:site.url}})

添加阿里的规则，例如添加CommentsMustBeJavadocFormatRule 规则：

1. 在 pmd.properties中添加

   ``` properties
   rule.pmd.CommentsMustBeJavadocFormatRule.name=CommentsMustBeJavadocFormatRule
   ```

2. 在 rules-p3c.xml中添加

   ``` xml
   <rule key="CommentsMustBeJavadocFormatRule">
   <priority>MAJOR</priority>
   <configKey><![CDATA[rulesets/java/ali-comment.xml/CommentsMustBeJavadocFormatRule]]></configKey>
   </rule>
   ```

3. 在pmd-model.xml中添加

   ``` xml
   <!--AlibabaJavaComments-->
     <chc>
       <rule-repo>pmd</rule-repo>
       <rule-key>CommentsMustBeJavadocFormatRule</rule-key>
       <prop>
           <key>remediationFunction</key>
           <txt>CONSTANT_ISSUE</txt>
       </prop>
       <prop>
           <key>offset</key>
           <val>2</val>
           <txt>min</txt>
       </prop>
   </chc>
   ```

4. 添加描述文件— org/sonar/l10n/pmd/rules/pmd-p3c/CommentsMustBeJavadocFormatRule.html，内容来自p3c对应xml 用于错误详情页面的展示：

   ``` html
   <p>Look for qualified this usages in the same class.</p>
     <p>Examples:</p>
     <pre>
     /**
     *
     * XXX class function description.
     *
     */
     public class XxClass implements Serializable {
        private static final long serialVersionUID = 113323427779853001L;
        /**
        * id
        */
        private Long id;
        /**
        * title
        */
        private String title;
        /**
        * find by id
        *
        * @param ruleId rule id
        * @param page start from 1
        * @return Result<Xxxx>
        */
        public Result<Xxxx> funcA(Long ruleId, Integer page) {
            return null;
        }
     }
   </pre>
   ```


### 2.1.3、修改p3c的提示语
如leader要求添加 PASS-ALI 前缀。

1. 下载p3c-pmd源码 [https://github.com/alibaba/p3c](https://github.com/alibaba/p3c)，描述内容都在p3c/p3c-pmd/src/main/resources/messages.xml，在messages.xml中每条规则的提示语前添加前缀 **PASS-ALI**:
   ![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\1.png" | prepend:site.url}})
2. 然后将p3c-pmd打包，装到本地仓库 mvn clean install -Dgpg.skip
3. 重新打包sonar-p3c-pmd工程，将打好的jar包放到sonarqube的 ..\extensions\plugins目录下，重启sonarqube。即可安装好整合好含有阿里开发规则的pmd插件。

### 2.1.4、修改pmd插件的显示名

如果想修改pmd插件在sonarqube中的插件显示名，可以修改 sonar-p3c-pmd 工程中的org.sonar.plugins.pmd.PmdConstants类  Sring REPOSITORY_NAME 名字即可，如下图：
![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\5.png" | prepend:site.url}})
![]({{"\img\posts_img\technology\如何在sonarQube的pmd插件中整合阿里开发规范\2.png" | prepend:site.url}})
