# 读取二进制文件(utmp)

*本节作者：NERO*

我们知道Linux系统有些日志不是以可读文本形式存在文件中，而是以二进制内容存放的。比如utmp等。

Files插件在读取二进制文件和文本文件的区别在于没办法一行行处理内容，内容也不是直观可见的。但是在其他处理逻辑是保持一致的，比如多文件支持、断点续传、内容监控等等，区别在于bytes的截断和bytes的转换。
在Files的基础上，新增了个插件utmp，插件实现了对二进制文件的读取和解析，实现类似于文本文件一行行输出内容，以供Filter做进一步分析。


utmp新增了两个配置项：`struct_size` 和 `struct_format`。

`struct_size` 代表结构体的大小，number类型；`struct_format`为结构体的内存分布格式，string类型。

## 配置实例

```
input {
  utmp {
    path => "/var/run/utmp"
    type => "syslog"
    start_position => "beginning"
    struct_size => 384
    struct_format => "s2Ia32a4a32a256s2iI2I4a20"
  }
}
```

我们可以通过 `man utmp` 了解 utmp 的结构体声明。通过结构体声明我们能知道结构体的内存分布情况。我们看
 `struct_format => "s2Ia32a4a32a256s2iI2I4a20"` 中的第一个 **s**，s 代表 short，后面的数字 2 代表 short 元素的个数，那么意思就是结构体的第一个和第二个元素是 short 类型，各占 2 个字节。

以此类推。s2Ia32a4a32a256s2iI2I4a20 总共代表 384 个字节，与 `struct_size` 配置保持一致。s2Ia32a4a32a256s2iI2I4a20 本质上是对应于 Ruby 中的 unpack 函数的参数。其中 s、T、a、i 等等含义参见 Ruby 的 unpack 函数说明。

需要注意的是，你必须了解当前操作系统的字节序是大端模式还是小端模式，对于数字类型的结构体元素很有必要，比如对于 i 和 I 的选择。还有一个就是，结构体内存对齐的问题。实际上 utmp 结构体第一个元素 2 字节，第二个元素是 4 字节。编译器在编译的时候会在第一个 2 字节后插入 2 字节来补齐 4 字节，知道这点尤为重要，所以其实第二个 short 是无意义的，只是为了内存对其才有补的。

## 输出

经过 utmp 插件的 event，message 字段形如 "field1|field2|field3"，以 "|" 分隔每个结构体元素。可用Filter插件进一步去做解析。utmp 本身只负责将二级制内容转换成可读的文本，不对文件编码格式负责。

## 安装

插件地址：https://github.com/txyafx/logstash-input-utmp

```
git clone https://github.com/txyafx/logstash-input-utmp
cd logstash-input-utmp
rm -rf .git
vim Gemfile 修改插件绝对路径
git init
git add .
gem clean
gem build logstash-input-utmp.gemspec
```

用logstash安装本地utmp插件
