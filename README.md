### Maven发布插件&Bintray发布插件

使用此插件时，请移除

```
classpath "com.github.dcendents:android-maven-gradle-plugin:1.5"
classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.3'
```

以及移除原先的发布脚本

然后在buildscript中加入classpath依赖

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'io.github.lizhangqu:core-publish:1.0.11'
    }
}
```

最低支持gradle 2.10，低于gradle 2.10时会报错，最高支持到gradle 4.4

配置相关属性并应用插件
```
ext {
    RELEASE_REPOSITORY_URL = "file://${project.rootProject.file('repo/release')}"
    SNAPSHOT_REPOSITORY_URL = "file://${project.rootProject.file('repo/snapshot')}"
    RELEASE_REPOSITORY_USERNAME = ""
    RELEASE_REPOSITORY_PASSWORD = ""
    SNAPSHOT_REPOSITORY_USERNAME = ""
    SNAPSHOT_REPOSITORY_PASSWORD = ""
}
apply plugin: 'core.publish'

release {
    versionKey = 'PROJECT_POM_VERSION'
}
```

gradle.properties中配置

```
PROJECT_POM_GROUP_ID=io.github.lizhangqu
PROJECT_POM_ARTIFACT_ID=core-publish
PROJECT_POM_VERSION=1.0.0-SNAPSHOT
```

如果使用android gradle plugin 3.0 implementation 依赖，请使用gradle3.0.0对应的版本


## Maven发布配置

需要配置maven地址，账号和用户名，值可以存在gradle.properties,local.properties或者project.ext属性中

```
RELEASE_REPOSITORY_URL = 
SNAPSHOT_REPOSITORY_URL = 
RELEASE_REPOSITORY_USERNAME = 
RELEASE_REPOSITORY_PASSWORD =
SNAPSHOT_REPOSITORY_USERNAME = 
SNAPSHOT_REPOSITORY_PASSWORD = 
```

## Bintray发布配置

发布到Bintray时，需要提供 BINTRAY_APIKEY 和 BINTRAY_USER, 值可以存在gradle.properties,local.properties或者project.ext属性中，建议放在local.properties中，不进行版本跟踪

```
BINTRAY_USER = 
BINTRAY_APIKEY = 
```

## 支持属性列表

其中发布到maven时PROJECT_POM_GROUP_ID，PROJECT_POM_ARTIFACT_ID，PROJECT_POM_VERSION是必选项，如果未设置，则使用默认值rootProject.name:project.name:unspecific。其他为可选项。
对于发布到Bintray时，其他配置基本都是必选项，建议根据控制台错误信息进行配置。

### gradle.properties

```
PROJECT_POM_GROUP_ID=
PROJECT_POM_ARTIFACT_ID=
PROJECT_POM_VERSION=
POM_DESCRIPTION=
POM_LICENSE=
POM_LICENSE_URL=
POM_WEBSITE_URL=
POM_VCS_URL=
POM_ISSUE_URL=
POM_DEVELOPER_ID=
POM_DEVELOPER_NAME=
POM_DEVELOPER_EMAIL=
POM_ENABLE_JAVADOC=false //是否生成javadoc
POM_ENABLE_COORDINATE=false //是否主动设置project的group，archivesBaseName，version
```

### build.gradle

```
project.ext{
	PROJECT_POM_GROUP_ID=
	PROJECT_POM_ARTIFACT_ID=
	PROJECT_POM_VERSION=
	POM_DESCRIPTION=
	POM_LICENSE=
	POM_LICENSE_URL=
	POM_WEBSITE_URL=
	POM_VCS_URL=
	POM_ISSUE_URL=
	POM_DEVELOPER_ID=
	POM_DEVELOPER_NAME=
	POM_DEVELOPER_EMAIL=
	POM_ENABLE_JAVADOC=false //是否生成javadoc
	POM_ENABLE_COORDINATE=false //是否主动设置project的group，archivesBaseName，version
}
```

### local.properties

```
PROJECT_POM_GROUP_ID=
PROJECT_POM_ARTIFACT_ID=
PROJECT_POM_VERSION=
POM_DESCRIPTION=
POM_LICENSE=
POM_LICENSE_URL=
POM_WEBSITE_URL=
POM_VCS_URL=
POM_ISSUE_URL=
POM_DEVELOPER_ID=
POM_DEVELOPER_NAME=
POM_DEVELOPER_EMAIL=
POM_ENABLE_JAVADOC=false //是否生成javadoc
POM_ENABLE_COORDINATE=false //是否主动设置project的group，archivesBaseName，version
```

## maven发布

### release发布

release发布使用命令

```
gradle :moduleName:uploadRelease
```
uploadRelease这个task只是依赖了release这个task，没做其他多余工作

release发布依赖io.github.lizhangqu:core-release:1.0.0插件，默认使用递增版本号配置，如果需要，请覆写其配置项

```
release {
    failOnCommitNeeded = true // 本地有文件未提交，构建失败
    failOnPublishNeeded = true // 本地代码未push，构建失败
    failOnSnapshotDependencies = true // 依赖了SNAPSHOT包，构建失败
    failOnUnversionedFiles = true // 本地有未进行版本追踪的文件，构建失败
    failOnUpdateNeeded = true // 未将远端代码更新至本地，构建失败
    revertOnFail = true // 构建发生错误时，回滚插件的提交
    preCommitText = ''  // 插件创建的提交的message
    preTagCommitMessage = '[Gradle Release Plugin] - pre tag commit: ' // 如果存在snapshot版本，去snapshot字样时使用的message
    tagCommitMessage = '[Gradle Release Plugin] - creating tag: ' // 创建tag的message
    newVersionCommitMessage = '[Gradle Release Plugin] - new version commit: ' // 创建新版本的message
    tagTemplate = '${version}' // tag的模板，内部使用模板引擎渲染，支持${name}和${version}
    versionPropertyFile = 'gradle.properties' // 版本号所在文件
    versionProperties = [] //需要同步更新的版本号属性
    versionKey = null // 默认使用version作为key，当需要自定义时，设置此值
    buildTasks = ['build'] // 构建的task，建议设置为 assembleRelease
    versionPatterns = [
        /(\d+)([^\d]*$)/: { Matcher m, Project p -> m.replaceAll("${(m[0][1] as int) + 1}${m[0][2]}") }
    ] //版本匹配正则
    scmAdapters = [
        net.researchgate.release.GitAdapter,
        net.researchgate.release.SvnAdapter,
        net.researchgate.release.HgAdapter,
        net.researchgate.release.BzrAdapter
    ]
    git {
        requireBranch = 'master' // 构建需要的分支
        pushToRemote = 'origin' // push的远端
        pushToBranchPrefix = ''
        commitVersionFileOnly = false
        signTag = false
    }
    svn {
        username = null
        password = null
        pinExternals = false   // allows to pin the externals when tagging, requires subversion client >= 1.9.0
    }
}
```

### snapshot发布

snapshot发布使用命令

```
gradle :moduleName:uploadSnapshot
```

uploadSnapshot这个task依赖uploadArchives，发布时请确保版本号为SNAPSHOT，如果未追加-SNAPSHOT，则会抛出异常

注意不要使用uploadArchives，此Task被禁用，被uploadRelease和uploadSnapshot取代

## Bintray发布

Bintray只支持release发布，使用命令，发布后需要同步到jcenter的需要到后台进行处理，首次发布需要审核，更多详情见https://bintray.com/

```
gradle :moduleName:uploadBintray
```

注意不要使用bintrayUpload，此Task被禁用

## 自定义发布

如果你需要自定义发布一些文件，需要自己定义publishing，然后使用gradle publis发布，如下


```
publishing {
    publications {
        maven(MavenPublication) {
            artifact "${project.buildDir}/libs/${project.archivesBaseName}-${project.version}.jar"
        }
    }
}
```

除此之外，也可以通过添加 project.artifacts，然后使用uploadSnapshot或者uploadRelease发布

## 自定义多pom发布

```
project.afterEvaluate {
    uploadArchives {
        repositories {
            mavenDeployer {
                addFilter("yourPomName1") { artifact, file ->
                    artifact.name == "yourPomName1"
                }

                addFilter("yourPomName2") { artifact, file ->
                    artifact.name == "yourPomName2"
                }

                pom("yourPomName1") {
                    groupId = 'group1'
                    artifactId = "name1"
                    version = "version1"
                    packaging = 'so'
                }

                pom("yourPomName2") {
                    groupId = 'group2'
                    artifactId = "name2"
                    version = "version2"
                    packaging = 'jar'
                }
            }
        }
    }
    project.artifacts {
         archives file: new File("path/to/yourPomName1.so"), name: "yourPomName1"
         archives file: new File("path/to/yourPomName2.jar"), name: "yourPomName2"
    }
}
```

## 自定义classifier发布

```
project.artifacts {
    archives file: new File("path/to/mainFile")
    //必要时可发布classsifier文件，发布的文件需要group:name:version:classifier才能拉下来
    archives classifier: 'armeabi', file: new File("path/to/classifierFile")
    archives classifier: 'armeabi-v7a', file: new File("path/to/classifierFile")
    archives classifier: 'arm64-v8a', file: new File("path/to/classifierFile")
    archives classifier: 'x86', file: file: new File("path/to/classifierFile")
    archives classifier: 'x86_64',  file: new File("path/to/classifierFile")
    archives classifier: 'mips', file: new File("path/to/classifierFile")
    archives classifier: 'mips64',file: new File("path/to/classifierFile")
}
```

## native动态库依赖

```
apply plugin: 'core.native'
//apply plugin: 'core.publish' //包含了core.native插件

dependencies {
	nativeCompile 'com.snappydb:snappydb-native:0.2.0:armeabi@so'
	nativeCompile 'com.snappydb:snappydb-native:0.2.0:x86@so'
	nativeCompile 'com.snappydb:snappydb-native:0.2.0:mips@so'
	nativeCompile "com.snappydb:snappydb-native:0.2.0:armeabi-v7a@so"
}
```

其中classifier可选，其值为 armeabi, armeabi-v7a, arm64-v8a, x86, x86_64, mips, mips64其中一个，不是这些值会抛异常。
并且依赖中的ext @so是否需要携带取决于发布时默认的文件是否是so，如果存在classifier, 则@so为必选项，默认值为@jar，为了让其寻找so，需要手动指定为@so。
不支持引入所有abi，只支持单个abi逐个引入