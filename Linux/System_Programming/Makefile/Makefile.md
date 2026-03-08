# Makefile

---

## 什么是 Makefile

Makefile 是一种可以进行自动化构建和管理软件项目的文件。当项目工程足够大，通常一次编译需要涉及到多个源文件，一个个的输入到命令行是不现实的，故 Makefile 的作用即帮你完成这一步骤，让构建过程自动化。这一过程通常需要一个文件和一条指令：`Makefile 文件 或 makefile 文件； make 指令`。

---

## 如何构建 Makefile 文件

仅需使用 `touch Makefile 或 touch makefile` 即可创建 Makefile 文件，此处将以具体的 Makefile 来分析：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p1.png)

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p2.png)

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p3.png)

以上三图分别展示了：已有文件，test.c 内容，makefile 内容。

---

## 如何使用 Makefile

主要分析 makefile 的内容：

```linux
  1 myexe:test.c
  2     gcc test.c -o myexe
  3 
  4 .PHONY:clean
  5 clean:
  6     rm -f myexe   
```

第一行：指明依赖关系 -- `所需要生成的文件(目标文件):生成该文件需要依赖的文件(依赖项)`，依赖文件可以不止一个，若有多个用空格分割。
第二行：表述依赖方法 -- 直接用命令表述需要如何执行（开头一定要是 tab 键）。

此时只需要输入：`make` 即可自动执行上述过程。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p4.png)

当然，可以用 `$@` 表示所有冒号前面的内容，用 `$^` 表示所有冒号后面的内容。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p5.png)

这样可以更高效的完成构建工作。



第五行：clean 目标文件不需要任何依赖项，故冒号后面没有内容。
第六行：表述依赖方法，即删除生成的文件（开头一定要是 tab 键）。

此时想删除生成的文件，只需要：`make clean`

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p6.png)

即可高效完成删除。

第四行：此行表示 clean 指令被定义成伪目标，不对应任何的实际文件，确保是否有 clean 文件存在都可以执行该命令。
如果没有添加此项，并该目录下刚好有一个文件名称为 clean，这样 `make clean` 会认为其任务是创建 clean 目标，但 clean 目标已经存在且为最新，所以会发生 `make: `clean' is up to date.` 的错误。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p7.png)

故添加 `.PHONY:clean` 的目的是指明 clean 伪目标只是触发特定的操作或任务。

---

## make 和 make clean

为何 `make` 指令会直接构建第一个目标，而 clean 需要 `make clean` 才能构建 clean？但其实：`make myexe` 也是正确的，只是 make 默认会构建第一个目标，而 `make target` 可以指定构建的目标。

---

## 重复 make 

如果重复 make，会发现以下现象：

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p8.png)

提示 myexe 已经是最新的，无需再次更新。而事实也是如此，如果没有做出修改，更新 myexe 是不必要的，若修改了 test.c 则再次更新则有效。

而如果为 myexe 添加 `.PHONY` 将其构建为伪目标，则其一直会执行，因为**伪目标通常不需要指定依赖项**，则会无脑执行操作，无需关心依赖项是否被更新。

### 更新的判断

可知，编译一定是需要花费时间的，这就会导致目标文件的更新时间一定会比依赖项要晚，如果任意依赖项的更新时间晚于目标文件（或者说目标文件的更新时间早于任意依赖项时间），则说明依赖项一定在 make 之后更新过了，这就是 make 判断是否需要再次执行的逻辑。而这里的更新时间指的是时间戳，且指的是文件的修改时间（Modify time）。可以通过命令：`stat filename` 来查看文件的 访问时间（Access time）、修改时间（Modify time）、更改时间（Change time）。

![](https://thepale-2022-1313448760.cos.ap-beijing.myqcloud.com/Makefile-p9.png)

访问时间在我书写本文章时更新的频率比较低，不会在每次对文件进行访问时都对其进行更改，这是为了提高性能（因为其更新的频率实在太高，且每次都要写入文件）。而修改时间是指文件内容发生改变的时间，更改时间一般是文件属性发生更改的时间，这两者一是内容而是属性，切勿混淆。文件属性被修改内容不一定会更改，内容更改则文件属性一定会更改（因为文件内容的修改时间也属于文件属性，内容被修改属性一定变化）。

可以使用：`touch -a/-m/-c` 来修改三个时间为最新时间，或再追加：`-t time` 来更改至指定时间。