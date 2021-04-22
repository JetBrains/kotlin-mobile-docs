[//]: # (title: Configure SQLDelight for data storage)
[//]: # (auxiliary-id: Configure_SQLDelight_for_data_storage)

在移动开发领域，数据库通常用于客户端设备上的本地数据存储。在 Kotlin 多平台项目（*KMM*）中，操作<!--
-->数据库的一个选择是<!--
-->使用 [**SQLDelight**](https://cashapp.github.io/sqldelight/) 库。它从各种关系数据库的 SQL 语句中<!--
-->生成类型安全的 Kotlin API。SQLDelight 还提供了 SQLite 驱动程序的多平台实现。
关于 SQLDelight 功能和其他详细信息的描述，请参见 [SQLDelight 文档](https://cashapp.github.io/sqldelight/)。

在本文中，我们将展示如何在 Kotlin 移动多平台（KMM）项目中开始使用 SQLDelight 操作数据库：
* 将 SQLDelight 连接到项目
* 创建数据库驱动程序
* 使用 SQLDelight 生成的 API 执行数据库查询操作


## 连接并配置 SQLDelight

### Gradle 插件

要将 SQLDelight 插件连接到项目，请在项目的构建脚本（root `build.gradle` 或 `build.gradle.kts`）中 apply SQLDelight Gradle 插件：
首先，将插件的 `classpath` 添加到构建系统中：

<tabs>
<tab title="Groovy">
    
```Groovy
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath "com.squareup.sqldelight:gradle-plugin:$sql_delight_version"
    }
}
```

</tab>
<tab title="Kotlin">

```Kotlin
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath("com.squareup.sqldelight:gradle-plugin:$sql_delight_version")
    }
}
```

</tab>
</tabs>

使用项目所需的版本替换 `$sql_delight_version`。

然后通过在共享多平台模块的构建脚本（`build.gradle` 或 `build.gradle.kts`）的开头添加以下行来 apply SQLDelight Gradle 插件：

<tabs>
<tab title="Groovy">

```Groovy
apply plugin: 'com.squareup.sqldelight'
```

</tab>
<tab title="Kotlin">

```Kotlin
plugins {
    id("com.squareup.sqldelight")
}
```

</tab>
</tabs>

### 数据库驱动程序

#### 公共 source set

如需在公共代码中使用数据库驱动程序，可将以下依赖项添加到 `commonMain` source set 中：

<tabs>
<tab title="Groovy">
    
```Groovy
commonMain {
    dependencies {
        implementation "com.squareup.sqldelight:runtime:$sql_delight_version"
    }
}
```
  
</tab>
<tab title="Kotlin">

```Kotlin
val commonMain by getting {
    dependencies {
        implementation("com.squareup.sqldelight:runtime:$sql_delight_version")
    }
}
```

</tab>
</tabs>

####  Android source sets

如需连接 Android 的 SQLite 数据库驱动程序，可在模块的 `build.gradle` 或 `build.gradle.kts` 中将如下内容添加到对应 source set 的 `dependencies` 代码块中：

<tabs>
<tab title="Groovy">
    
```Groovy
androidMain {
    dependencies {
        implementation "com.squareup.sqldelight:android-driver:$sql_delight_version"
    }
}
```

</tab>
<tab title="Kotlin">

```Kotlin
val androidMain by getting {
    dependencies {
        implementation("com.squareup.sqldelight:android-driver:$sql_delight_version")
    }
}
```

</tab>
</tabs>

#### iOS source sets

如需连接 iOS 或其他原生平台的 SQLite 驱动程序，可添加如下依赖项：

<tabs>
<tab title="Groovy">
    
```Groovy
iosMain {
    dependencies {
        implementation "com.squareup.sqldelight:native-driver:$sql_delight_version"
    }
}
```

</tab>
<tab title="Kotlin">

```Kotlin
val iosMain by getting {
    dependencies {
        implementation("com.squareup.sqldelight:native-driver:$sql_delight_version")
    }
}
```

</tab>
</tabs>

### 配置

如需配置 SQLDelight API 生成器，可使用构建脚本的 sqldelight 顶级块。
例如，创建一个名为 `AppDatabase` 的数据库并指定<!--
-->生成的 Kotlin 类的包名为 `com.example.db`，可使用以下配置块：

<tabs>
<tab title="Groovy">

```Groovy
sqldelight {
    AppDatabase {
        packageName = "com.example.db"
    }
}
```

</tab>
<tab title="Kotlin">

```Kotlin
sqldelight {
    database("AppDatabase") {
        packageName = "com.example.db"
    }
}
```

</tab>
</tabs>    

下面列出的所有代码示例都将使用此 SQLDelight 配置。

如需了解在 SQLDelight 中可以配置哪些内容以及如何配置，请参见 [SQLDelight 文档](https://cashapp.github.io/sqldelight/multiplatform_sqlite/gradle/)。

## 创建 SQLite 驱动程序

SQLDelight 提供了 SQLite 驱动程序的多个平台的特定实现，所以应该分别为每个<!--
-->平台创建它。在公共代码中，你可以使用这些通用的 `SqlDriver` 接口来引用这些驱动程序。

可以利用 `expect`/`actual` 机制来创建一个抽象工厂：

```Kotlin
expect class DatabaseDriverFactory {
    fun createDriver(): SqlDriver
}
```

然后为这个期望的类提供实际的实现：

### Android 驱动程序

在 Android 上，SQLite 驱动程序由 `AndroidSqliteDriver` 类实现。当你创建它的实例时，需将数据库信息和指向上下文的链接传递给构造函数。例如，这段代码为名为 `AppDatabase` 的数据库创建了一个 SQLite 驱动程序：

```Kotlin
actual class DatabaseDriverFactory(private val context: Context) {
    actual fun createDriver(): SqlDriver {
        return AndroidSqliteDriver(AppDatabase.Schema, context, "test.db")
    }
}
```

### iOS 驱动程序

在 iOS 上，SQLite 驱动程序的实现是 `NativeSqliteDriver` 类：

```Kotlin
actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver {
        return NativeSqliteDriver(AppDatabase.Schema, "test.db")
    }
}
```

现在你可以在应用程序的代码中创建 `DatabaseDriverFactory` 实例并将其传递给公共的模块。然后创建一个 `AppDatabase` 实例来执行数据库操作：

```Kotlin
val database = AppDatabase(databaseDriverFactory.createDriver())
```

关于完整的示例，请参见[网络和数据存储实践](https://play.kotlinlang.org/hands-on/Networking%20and%20Data%20Storage%20with%20Kotlin%20Multiplatfrom%20Mobile/01_Introduction)。

## 表操作

SQLDelight 生成器的工作原理如下：创建一个扩展名为 `.sq ` 的文件，在该文件中向数据库提供了<!--
-->所需的 SQL 查询。SQLDelight 插件将生成 Kotlin 代码来执行这些查询操作。
这样，SQLDelight 就自动实现了应用程序与数据库的交互。这消除了<!--
-->手动实现实体类和将 Kotlin 类映射到关系数据库模型的代码的需要。

SQLDelight 生成器的语法使你可以实现所有基本的 SQLite 命令，包括级联，索引，
触发器和其他命令。

让我们看一下如何声明和使用基本的数据库操作。

### 创建

通常，用于创建所有必要数据库表的查询语句列在 `.sq` 文件的开头。
如需创建表，可使用 SQL `CREATE TABLE` 命令。例如，此查询创建一个包含两个字段的表：

```sql
CREATE TABLE Language (
    id INTEGER NOT NULL PRIMARY KEY,
    name TEXT NOT NULL
);
```

对于此查询，SQLDelight 生成带有指定字段的 `Language` Kotlin 接口。它将被用于<!--
-->实现了与 `Language` 表操作的函数中。

### 删除

SQL 的 `DELETE` 操作符用于删除数据表中的行。例如，要删除表中的所有记录，
可在 `.sq` 文件中声明如下查询：

```sql
deleteAllLanguages:
DELETE FROM Language;
```

第一行中的 `deleteAllLanguages:` 标签声明了将执行该查询的 Kotlin 函数的名称。

```kotlin
fun deleteAllLanguages()
```

编写以下代码来执行 `deleteAllLanguages` 查询：

```kotlin
val database = AppDatabase(sqlDriver)
val appDatabaseQueries: AppDatabaseQueries = database.appDatabaseQueries

fun deleteAllLanguages() {
    appDatabaseQueries.deleteAllLanguages()
}
```

你可以使用 `WHERE` 操作符来删除表中的某些行，例如：

```sql
deleteLanguageById:
DELETE FROM Language 
WHERE id = ?;
```

SQLDelight 将会生成带有一个参数的 Kotlin 函数：

```kotlin
fun deleteLanguageById(id: Long)
```

若要使用 `deleteLanguageById()` 函数删除一个指定的数据库记录，可在 `AppDatabaseQueries` 对象上调用该函数<!--
-->并传递将要删除的记录的 `id`：

```kotlin
fun deleteLanguageById(id: Long) {
    appDatabaseQueries.deleteLanguageById(id)
}
```

### 插入

如需添加数据记录到表中，请使用 SQL `INSERT` 命令。向 `Language` 表中插入一条记录的查询语句<!--
-->将像这样：

```sql
insertLanguage:
INSERT INTO Language(id, name)
VALUES(?, ?);
```

`insertLanguage:` 这里定义了 SQLDelight 生成的相应 Kotlin 函数的名称:

```kotlin
fun insertLanguage(id: Long?, name: String)
```

该函数接受两个查询中与该表指定字段相匹配的参数。

可以通过以下方法在应用程序代码中将一条新记录插入到表中：

```kotlin
data class SystemLanguage(
    val id: Long,
    val name: String
)

val database = AppDatabase(sqlDriver)
val appDatabaseQueries: AppDatabaseQueries = database.appDatabaseQueries

fun insertLanguage(systemLanguage: SystemLanguage) {
    appDatabaseQueries.insertLanguage(systemLanguage.id, systemLanguage.name)
}
```

### 更新

SQL `UPDATE` 命令将更改表中特定行的给定字段的值。
例如，此查询使用提供的标识符更改记录的名称：

```sql
updateLanguageName:
UPDATE Language
SET name = ?
WHERE id = ?;
```

`updateLanguageName:` 这里定义了 SQLDelight 生成的相应 Kotlin 函数的名称:

```kotlin
fun updateLanguageName(name: String, id: Long)
```

该函数接受与查询相匹配的两个参数。

使用以下方式在应用程序代码中更新表中的记录：

```kotlin
data class SystemLanguage(
    val id: Long,
    val name: String
)

val database = AppDatabase(sqlDriver)
val appDatabaseQueries: AppDatabaseQueries = database.appDatabaseQueries

fun updateLanguageName(id: Long, newName: String) {
    appDatabaseQueries.updateLanguageName(newName, id)
}
```

### 选择

如需从表中选择记录，可使用 `SELECT` 操作符。例如，如果你想从表中选择所有记录，
那么可以将如下查询添加到 `.sq` 文件中：

```sql
selectAllLanguages:
SELECT * FROM Language;
```

对于这个查询，SQlDelight 将创建以下函数：

```kotlin
fun selectAllLanguages(): Query<Language>
fun <T : Any> selectAllLanguages(mapper: (id: Long, name: String) -> T): Query<T>
```

正如你所看到的，第二个 `selectAllLanguages` 函数的第一个参数是一个 `mapper` lambda 表达式，它将<!--
-->来自选择结果的数据转换成任意类型 `T` 对象。例如，如果你需要将查询结果转换为<!--
-->应用程序业务逻辑所需的实体，可编写以下代码：

```kotlin
val database = AppDatabase(sqlDriver)
val appDatabaseQueries: AppDatabaseQueries = database.appDatabaseQueries

data class SystemLanguage(
    val id: Long,
    val name: String
)

fun selectAllLanguages(): List<SystemLanguage> {
    return appDatabaseQueries.selectAllLanguages { id: Long, name: String ->
        SystemLanguage(id, name)
    }.executeAsList()
}
```

大多数查询都包含选择条件。如果要显示带有特定标识符的记录，可在 `.sq` 文件中添加<!--
-->如下请求：

```sql
selectLanguageById:
SELECT * FROM Language
WHERE id = ?;
```

对于这个查询，SQLDelight 将会创建以下函数：

```kotlin
fun selectLanguageById(id: Long): Query<Language>
fun <T : Any> selectLanguageById(id: Long, mapper: (id: Long, name: String) -> T): Query<T>
```

与上述的示例类似，你可以创建一个查询数据库的函数并且将结果转换为<!--
-->所需要的数据类对象：

```kotlin
fun selectById(languageId: Long): SystemLanguage {
    return appDatabaseQueries.selectLanguageById(languageId) { id: Long, name: String ->
        SystemLanguage(id, name)
    }.executeAsOne()
}
```

## 事务

SQLDelight 能够在单个事务中执行多个 SQL 查询。为此，生成的查询 Kotlin
接口提供了 `transaction` 函数用于创建事务。

如需执行一个带有多个查询的数据库事务，可调用 `transaction` 函数并传递<!--
-->带有这些查询的lambda。例如，在单个事务中添加列表里的所有元素的函数如下所示：

```kotlin
fun insertAllLanguages(languages: List<SystemLanguage>) {
    database.appDatabaseQueries.transaction {
        languages.forEach { language ->
            database.appDatabaseQueries.insertLanguage(language.id, language.name)
        }
    }
}
```

## Android Studio 的 SQLDelight 插件

为了简化编写 `.sq` 生成器文件的工作，SQLDelight 提供了一个 Android Studio 插件。该插件增加了语法<!--
-->高亮、代码补全、用法搜索、重构、显示编译时错误等功能。

如需在 Android Studio 里安装这个插件，打开 **Preferences | Plugins | Marketplace** 然后在<!--
-->搜素框里输入 **SQLDelight**。

关于该插件的更多信息，请参见 [SQLDelight 文档](https://cashapp.github.io/sqldelight/multiplatform_sqlite/intellij_plugin/)。

_我们要感谢 [IceRock team](http://icerockdev.com/) 帮助我们撰写本文。_
