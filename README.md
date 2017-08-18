# checkstyle
##主要就是采用androidStudio的checkstyle进行项目代码的规范化
##采用以下的步骤来进行

  1、在androidStudio上装上CheckStyle的插件，成功之后会有一个checkStyle界面
  
  2、配置checkstyle规则，可以在网上（https://github.com/checkstyle/checkstyle/blob/master/src/main/resources/google_checks.xml）查找，
  然后对其稍加改造成适合自己的编码规则，在androidStudio上配置上checkstyle.xml，就可以在checkstyle上进行代码检查了
  
  3（可选）、可以在项目编译运行前进行代码检查，在app的gradle中加入以下的代码，然后在配置启动项Edit Configurations，在before launch中添加这个task即可
  
   task checkstyle(type: Checkstyle) {
    source 'src/main/java'
//    include '**/*.java'
    //只对修改过的文件进行检查
    if (project.hasProperty('checkCommit') && project.property("checkCommit")) {
        def ft = filterCommitter(getChangeFiles());
        def includeList = new ArrayList<String>()
        for (int i = 0; i < ft.size(); i++) {
            String spliter = ft.getAt(i)
            String[] spliterlist = spliter.split("/")
            String fileName = spliterlist[spliterlist.length - 1]
            log("Checkstyle:file=" + fileName)
            includeList.add("**/" + fileName)
        }
        if (includeList.size() == 0) {
            exclude '**/*.java'
        } else {
            include includeList
        }
    } else {
        include '**/*.java'
    }
    configFile rootProject.file('checkstyle.xml')
    classpath = files()
}

def filterCommitter(String gitstatusinfo) {
    ArrayList<String> filterList = new ArrayList<String>();
    String[] lines = gitstatusinfo.split("\\n")
    for (String line : lines) {
        if (line.contains(".java")) {
            String[] spliters = line.trim().split(" ");
            for (String str : spliters) {
                if (str.contains(".java")) {
                    filterList.add(str)
                }
            }
        }
    }
    return filterList;
}


def getChangeFiles() {
    try {
        String changeInfo = 'git status -s'.execute(null, project.rootDir).text.trim()
        return changeInfo == null ? "" : changeInfo
    } catch (Exception e) {
        return ""
    }
}


4（可选）在commit代码前进行代码检查，gradle依照上面的配置，只要在项目目录中的.git\hooks目录下，新建一个pre.commit文件，贴入以下代码即可

\#!/bin/sh
\#
\# An example hook script to verify what is about to be committed.
\# Called by "git commit" with no arguments.  The hook should
\# exit with non-zero status after issuing an appropriate message if
\# it wants to stop the commit.
\#
\# To enable this hook, rename this file to "pre-commit".

if git rev-parse --verify HEAD >/dev/null 2>&1
then
	against=HEAD
else
	# Initial commit: diff against an empty tree object
	against=4b825dc642cb6eb9a060e54bf8d69288fbee4904
fi

SCRIPT_DIR=$(dirname "$0")
SCRIPT_ABS_PATH=`cd "$SCRIPT_DIR"; pwd`
$SCRIPT_ABS_PATH/../../gradlew  -PcheckCommit="true" checkstyle 
if [ $? -eq 0   ]; then
    echo "checkstyle OK"
else
    exit [[ $ERROR_INFO =~ "checkstyle" ]] && exit 1  
fi
  
   
