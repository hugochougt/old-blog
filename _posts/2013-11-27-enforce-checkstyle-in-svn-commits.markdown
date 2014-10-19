---
layout: post
title: "用 Checkstyle 进行 SVN 提交的 Java 代码规范检查"
date: 2013-11-27 21:37
comments: true
categories: [checkstyle, SVN, pre-commit]
---

本文主要介绍怎样设置 [Checkstyle](checkstyle.sourceforge.net) 使得向 SVN 提交 Java 代码时进行代码规范检查。

## 安装 Checkstyle
到 Checkstyle 的[下载页面](http://sourceforge.net/projects/checkstyle/files/checkstyle)选择你需要的版本下载，一般都是选最新版，然后解压。对我们有用的是 checkstyle-x.x-all.jar 这个 jar 包。

## 配置 Checkstyle 检查规则
在上一步下载的压缩包里，有一个 sun_xhecks.xml 的文件，结合 Checkstyle 的配置说明文档將其修改成适合你的项目的规则。

## 测试 Checkstyle
运行 Checkstyle 需要 JRE 的运行环境，请预先准备好。这里不赘述。运行以下命令进行测试：

    $ java -jar /path/to/checkstyle-x.x-all.jar -c /path/to/sun_checks.xml -r /path/to/project/src

其中將 `/path/to/project/src` 指向你的 Java 工程源码目录即可。

## 配置 SVN pre-commit 钩子
进入 svnrepo/hooks 目录，將 pre-commit.tmpl 复制一份并重命名为 pre-commit，然后在其 exit 0 这一行前添加执行我写的一个脚本命令：

    /path/to/pre-commit-checkstyle.sh $REPOS $TXN 1>&2 || exit 1

**记得给 pre-commit 脚本添加执行权限。**
<!-- more -->
``` bash pre-commit-checkstyle.sh - SVN pre-commit Checkstyle
#!/bin/bash
# Filename: pre-commit-checkstyle.sh
# Author: zhaqiang
# Description: To execute checkstyle of Java source code before
# svn commit, add following line to svn pre-commit hook before 
# exit 0:
#
# /path/to/pre-commit-checkstyle.sh $REPOS $TXN 1>&2 || exit 1
#

repoPath=$1
txnName=$2

currPath=`dirname $0`
currPath=`cd ${currPath} ; pwd`

checkstyleJar=/path/to/chechstyle.jar
checkstyleXml=/path/to/styelchecks.xml

tmpDir=${currPath}/temp
[ -d ${tmpDir} ] || mkdir -p -m 777 ${tmpDir} 2>/dev/null

# 1. filter added or updated Java files from files changed
javaFile=`svnlook changed -t ${txnName} ${repoPath} | awk '/^[AU].*\.java/ {print $2}'`

# exit successfully when there is no Java file changed
if [ "X${javaFile}" = "X" ] ; then
    exit 0
fi

tmpWorkspace=${tmpDir}/work.$$
mkdir -p -m 777 ${tmpWorkspace}

# 2. copy committed Java source code to a tmp workspace
for file in ${javaFile} ; do
    targetFile="${tmpWorkspace}/${file}"
    targetDir=`dirname $0`

    [ -d ${targetDir} ] || mkdir -p -m 777 ${targetDir}
    svnlook cat -t ${txnName} ${repoPath} > ${targetFile}
done

# 3. execute checkstyle
checkstyleLog=${tmpDir}/tmp$$.log
java -jar ${checkstyleJar} -c ${checkstyleXml} -r ${tmpWorkspace} > ${checkstyleLog}

# 4. check and print result of checkstyle
lineNum=`sed -n '$=' ${checkstyleLog}`
regex=`echo ${tmpWorkspace} | sed 's#\/#\\\/#g'`
if [ "X${lineNum}" != "X2" ] ; then
    echo "Checkstyle errors:"
    sed "s/${regex}//" ${checkstyleLog}
    rm -rf ${tmpWorkspace} ${checkstyleLog} # clean up
    exit 1
fi

# 5. clean up
rm -rf ${tmpWorkspace} ${checkstyleLog}

echo "Checkstyle done without errors."

exit 0
```

根据你的安装配置修改 `checkstyleJar` 和 `checkstyleXml` 两个变量的位置即可。这个脚本会对提交至 SVN 的新增(Added)和更新(Updated)的 Java 文件进行 Checkstyle 检查，其他后缀名的文件不予理会。

参考文献：

 1. [How to enforce Checkstyle in SVN commits : Simple Guide](http://www.iamjk.com/2009/06/how-to-enforce-checkstyle-in-svn.html)
 
 2. [用checkstyle实现svn的代码规范性检查](http://tech.it168.com/a2011/0608/1201/000001201666.shtml)

 3. [Svn与Checkstyle整合](http://xtony.blog.51cto.com/3964396/811418)

**-EOF-**

版权声明：自由转载-非商用-非衍生-保持署名 | [Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh "CC 3.0")
