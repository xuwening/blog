
# google diff match库介绍

## 前言

全名是google diff match patch，是谷歌根据[Myer's diff algorithm](https://neil.fraser.name/software/diff_match_patch/myers.pdf)论文实现的一套差异比较工具。目前实现了的语言版本有`Java, JavaScript, Dart, C++, C#, Objective C, Lua and Python`。

项目地址是：[google-diff-match-patch](https://code.google.com/p/google-diff-match-patch/)，一看就知道需要翻墙，等你翻墙了发现项目失效无法下载。最后在github上找到一个同步版本：[github](https://github.com/bystep15/google-diff-match-patch)。

下载下来代码，可以先看demo目录下的三个demo，大致了解该库的三个核心功能：

- 差异比较
- 匹配
- 差异补丁

## 差异比较

差异比较就是罗列出从文本1转换到文本2需要的一些列步骤，文本转换基础操作描述：`1、插入；2、删除；3、相等；`，分别用`1 -1 0`表示。如：
diff("Good dog", "Bad dog") => [(-1, "Goo"), (1, "Ba"), (0, "d dog")]。

三种Cleanup模式：

1、No Cleanup，默认方式
2、Semantic Cleanup，符合人的语意方式
3、Efficiency Cleanup，效率方式

举例说明：mouse和sofas的比较，默认方式：[(-1, "m"), (1, "s"), (0, "o"), (-1, "u"), (1, "fa"), (0, "s"), (-1, "e")]。而Semantic方式则是：[(-1, "mouse"), (1, "sofas")]。Efficiency类似Semantic方式，不过对机器处理来说效率比较好。

三中差异比较模式：

1、character模式（默认）
2、Word模式
3、Line模式

即差异比较的最小单元，选取哪种比较模式，根据项目需求而定。

## 匹配

主要是提供模糊匹配，可以调节匹配参数。

## 差异补丁

根据差异比较，生成差异步骤，差异步骤应用到原始文本上，就会推导出目标文本。这个差异步骤，就是差异补丁。

## API使用

1、先创建diff_match_patch对象
2、比较差异，diff_main(text1, text2) => diffs
3、应用补丁，patch_apply(patches, text1) => [text2, results]

针对diffs可以单独操作，比如采用Efficiency模式`diff_cleanupEfficiency(diffs) => null`或Semantic模式`diff_cleanupSemantic(diffs) => null`。

