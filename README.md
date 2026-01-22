# Gradle Git Repo 插件

这是一个 Gradle 插件，允许你将 Git 仓库作为 Maven 仓库使用。即使是私有的 Git 仓库也可以正常工作（类似于 CocoaPods 的工作方式）。

通过将 GitHub 仓库用作 Maven 仓库，你可以快速简便地托管 Maven JAR 包。传统的私有 Maven 仓库通常需要复杂的 HTTP 认证配置。该插件通过在后台克隆仓库并将其作为本地仓库使用，只要你有权限克隆该仓库，就可以访问其中的依赖。

## 核心特性

*   **支持 Gradle 8.x/9.x**：已适配最新的 Gradle 版本。
*   **支持私有仓库**：基于 Git SSH/HTTP 认证，无需额外配置 Maven 认证。
*   **简单易用**：直接在 `repositories` 块中声明。

---

## 如何使用

### 1. 引入插件

在你的项目 `settings.gradle` 或根目录 `build.gradle` 中配置插件仓库：

```groovy
pluginManagement {
    repositories {
        maven { url "https://github.com/blazeqin/gradle-git-repo-plugin/raw/master/releases" }
        gradlePluginPortal()
    }
}
```

在你的 `build.gradle` 中应用插件：

```groovy
plugins {
    id 'com.github.amistad012oss.gradle.git-repo-plugin' version '2.0.0'
}
```

### 2. 添加 Git 依赖仓库

插件会为 `repositories` 块添加 `github` 方法：

```groovy
repositories {
    // 参数说明: 组织名, 仓库名, 分支(可选, 默认 master), 类型(可选, 默认 releases)
    github("blazeqin", "maven-private", "master", "releases")
}
```

添加后，你就可以像使用普通 Maven 仓库一样添加依赖了：

```groovy
dependencies {
    implementation 'your.group:artifact:1.0.0'
}
```

---

## 如何发布

如果你想将自己的项目发布到 Git 仓库中，请按照以下步骤操作：

### 1. 配置发布任务

在你的项目的 `build.gradle` 中设置 `maven-publish`：

```groovy
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            // 这里指定本地克隆仓库的地址，插件会自动处理推送
            url "file://${System.env.HOME}/.gitRepos/${property("org")}/${property("repo")}/releases"
        }
    }
}
```

### 2. 执行发布命令

使用以下命令将构件发布并推送到 GitHub：

```bash
./gradlew -Porg=你的组织名 -Prepo=你的仓库名 publishToGithub
```

该任务会自动执行以下操作：
1. 编译并打包项目。
2. 将构件发布到本地缓存的 Git 目录。
3. 执行 `git add`, `git commit` 和 `git push` 将更改推送到远程仓库。

---

## 配置项

支持以下可选参数（通过 `-P` 传递）：

*   `gitRepoHome`：本地克隆仓库的存放位置，默认为 `~/.gitRepos`。
*   `org`：(发布必填) 目标 GitHub 组织或用户名。
*   `repo`：(发布必填) 目标 GitHub 仓库名。
*   `publishTask`：(可选) 指定发布任务名称，默认为 `publish`。

---

## 许可证

本项目采用 Apache 2 许可证。详细信息请参阅 LICENSE 文件。
