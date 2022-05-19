### 1.更新依赖
当我们更新了某个依赖的版本号，并执行同步之后，所依赖的仍然是旧版本，代码仍然是一片爆红 - 找不到xxx方法等等问题的时，可以通过下述步骤进行处理。

1. clean大法
	即先执行clean命令，然后再重新sync
``` shell
./gradlew clean
```
2. Invalidate Caches And Restart
3. 删除缓存
删除缓存之后在下一次Gradle构建时会重新下载依赖。
``` shell
rm -rf $HOME/.gradle/caches/
```
4. 更新依赖
``` shell
./gradlew build  --refresh-dependencies
```

> **Notice：**
> **The --refresh-dependencies option tells Gradle to ignore all cached entries for resolved modules and artifacts.** A fresh resolve will be performed against all configured repositories, with dynamic versions recalculated, modules refreshed, and artifacts downloaded.However, where possible Gradle will check if the previously downloaded artifacts are valid before downloading again. This is done by comparing published SHA1 values in the repository with the SHA1 values for existing downloaded artifacts.
> 
>It’s a common misconception to think that using --refresh-dependencies will force download of dependencies. This is not the case: Gradle will only perform what is strictly required to refresh the dynamic dependencies. This may involve downloading new listing or metadata files, or even artifacts, but if nothing changed, the impact is minimal.
>


### 2.查看依赖
1. 查看目标项目的依赖树并输出到文件中
```groovy
$ ./gradlew :<project>:dependencies > log.txt
```

2. 查看指定库的依赖关系
```groovy
$ ./gradlew :<project>:dependencyInsight --dependency <dependency_library> --configuration compile
```

注意`dependencyInsight`默认只会查看compile阶段的依赖，如果要查看其他阶段可以使用`--configuration`来指定。

### 3.构建时间
使用`--profile`  参数可以产生任务运行时间的报告。该报告存储在build/report/profile目录。
```groovy
$ ./gradlew build --profile
```
 