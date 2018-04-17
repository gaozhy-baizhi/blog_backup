---
title: Maven下载依赖不完整解决方案
date: 2018-04-02 10:56:47
tags:
	- 项目管理
	- 项目构建
categories: Big Data
---

**问题**：使用Maven管理项目，下载依赖的时候由于网络等问题，导致依赖下载不完整，无法再次下载，大大的影响开发效率。  
{% asset_img  2018-04-02_110239.jpg 图片%}
**原因**：不完整的依赖会在本地仓库依赖目录中产生一个 `*.lastUpdated` 文件(如图所示)。如果有此文件，Maven不会再次下载。
![](2018-04-02_111501.jpg)  
<!-- more -->
**解决方案**：删除Maven本地仓库中所有依赖目录中的`*.lastUpdated`文件，再次刷新项目下载即可。
```java
import java.io.File;
import java.util.Scanner;

/**
 * 递归删除maven仓库中的lastUpdate文件
 *
 * @author gaozhy
 * @date 2018/3/22.16:25
 */
public class RecursionDeleteLastUpdatedFile {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        // 如 E:\\maven_repository
        System.out.print("请您输入Maven本地仓库路径 [注意:路径中不能有空格!] >>  ");
        String mavenRepository = scanner.next();
        File file = new File(mavenRepository);
        m1(file);
    }

    /**
     * 递归删除文件方法
     */
    public static void m1(File file) {
        File[] files = file.listFiles();
        for (File f : files) {
            if (f.isDirectory()) {
                m1(f);
            }
            String extensionName = f.getName().substring(f.getName().lastIndexOf(".") + 1);
            if ("lastUpdated".equals(extensionName)){
                System.out.println("删除文件：" + f.getAbsolutePath());
                f.delete();
            }
        }
    }
}
```
![](2018-04-02_111927.jpg)