# plugin的安装

从 logstash 1.5.0 版本开始，logstash 将所有的插件都独立拆分成 gem 包。这样，每个插件都可以独立更新，不用等待 logstash 自身做整体更新的时候才能使用了。

为了达到这个目标，logstash 配置了专门的 plugins 管理命令。

## plugin 用法说明

```
Usage:
    bin/plugin [OPTIONS] SUBCOMMAND [ARG] ...

Parameters:
    SUBCOMMAND                    subcommand
    [ARG] ...                     subcommand arguments

Subcommands:
    install                       Install a plugin
    uninstall                     Uninstall a plugin
    update                        Install a plugin
    list                          List all installed plugins

Options:
    -h, --help                    print help
```

## 示例

首先，你可以通过 `bin/plugin list` 查看本机现在有多少插件可用。(其实就在 vendor/bundle/jruby/1.9/gems/ 目录下)

然后，假如你看到 `https://github.com/logstash-plugins/` 下新发布了一个 `logstash-output-webhdfs` 模块(当然目前还没有)。打算试试，就只需要运行：

```
bin/plugin install logstash-output-webhdfs
```

就可以了。

同样，假如是升级，只需要运行：

```
bin/plugin update logstash-input-tcp
```

即可。
