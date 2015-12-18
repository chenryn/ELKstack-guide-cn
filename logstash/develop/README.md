# 自己写一个插件

前面已经提过在运行 logstash-1.4.2 的时候，可以通过 `--pluginpath` 参数来加载自己写的插件。那么，插件又该怎么写呢？

## 插件格式

一个标准的 logstash 输入插件格式如下：

```ruby
require 'logstash/namespace'
require 'logstash/inputs/base'
class LogStash::Inputs::MyPlugin < LogStash::Inputs::Base
  config_name 'myplugin'
  milestone 1
  config :myoption_key, :validate => :string, :default => 'myoption_value'
  public def register
  end
  public def run(queue)
  end
end
```

其中大多数语句在过滤器和输出阶段是共有的。

* config_name 用来定义该插件写在 logstash 配置文件里的名字；
* milestone 标记该插件的开发里程碑，一般为1，2，3，如果不再维护的，标记为 0；
* config 可以定义很多个，即该插件在 logstash 配置文件中的可配置参数。logstash 很温馨的提供了验证方法，确保接收的数据是你期望的数据类型；
* register logstash 在启动的时候运行的函数，一些需要常驻内存的数据，可以在这一步先完成。比如对象初始化，*filters/ruby* 插件中的 `init` 语句等。

*小贴士*

milestone 级别在 3 以下的，logstash 默认为不足够稳定，会在启动阶段，读取到该插件的时候，输出类似下面这样的一行提示信息，日志级别是 warn。**这不代表运行出错**！只是提示如果用户碰到 bug，欢迎提供线索。

> {:timestamp=>"2015-02-06T10:37:26.312000+0800", :message=>"Using milestone 2 input plugin 'file'. This plugin should be stable, but if you see strange behavior, please let us know! For more information on plugin milestones, see http://logstash.net/docs/1.4.2-modified/plugin-milestones", :level=>:warn}

## 插件的关键方法

输入插件独有的是 run 方法。在 run 方法中，必须实现一个长期运行的程序(最简单的就是 loop 指令)。然后在每次收到数据并处理成 `event` 之后，一定要调用 `queue << event` 语句。一个输入流程就算是完成了。

而如果是过滤器插件，对应修改成：

```ruby
require 'logstash/filters/base'
class LogStash::Filters::MyPlugin < LogStash::Filters::Base
  public def filter(event)
  end
end
```

输出插件则是：

```ruby
require 'logstash/outputs/base'
class LogStash::Outputs::MyPlugin < LogStash::Outputs::Base
  public def receive(event)
  end
end
```

另外，为了在终止进程的时候不遗失数据，建议都实现如下这个方法，只要实现了，logstash 在 shutdown 的时候就会自动调用：

```ruby
public def teardown
end
```

## 推荐阅读

* [Extending logstash](http://logstash.net/docs/1.4.2/extending/)
* [Plugin Milestones](http://logstash.net/docs/1.4.2/plugin-milestones)

# 插件打包

Logstash 从 1.5.0-GA 版开始，对插件规范做了重大变更。放弃了 milestone 定义，去除了 `--pluginpath` 命令行参数。统一改成 `bin/plugin` 管理的 rubygem 包插件。那么，我们自己写的 Logstash 插件，也同样需要适应这个新规则，写完 Ruby 代码，还要打包成 gem 才能使用。

为了我们更方便的完成工作，logstash 针对 4 种插件形态提供了 4 个示例库，可以按照自己所需克隆使用。比如要写一个 logstash-filter-mything 插件：

```
git clone https://github.com/logstash-plugins/logstash-filter-example
cd logstash-filter-example
mv logstash-filter-example.gemspec logstash-filter-mything.gemspec
mv lib/logstash/filters/example.rb lib/logstash/filters/mything.rb
mv spec/filters/example_spec.rb spec/filters/mything_spec.rb
```

然后把代码写在 `lib/logstash/filters/mything.rb` 里即可。

代码部分完成。然后就是定义 gem 打包需要的额外文件和库依赖了。

目录中有两个文件，`Gemfile` 和 `logstash-filter-mything.gemspec`。

`Gemfile` 文件就是标准格式，用来运行 `bundler install` 时下载 rubygems 包的。默认情况下，最基础的内容是：

```
source 'https://rubygems.org' # 国内建议改成 'http://ruby.taobao.org'
gemspec
gem "logstash", :github => "elastic/logstash", :branch => "1.5"
```

`gemspec` 文件则是用来定义软件包本身规范，不单限于 rubygems。示例如下：

```
Gem::Specification.new do |s|
  s.name = 'logstash-filter-mything'
  s.version         = '1.1.0'
  s.licenses = ['Apache License (2.0)']
  s.summary = "This mything filter is just for ELK Stack Guide example"
  s.description = "This gem is a logstash plugin required to be installed on top of the Logstash core pipeline using $LS_HOME/bin/plugin install gemname. This gem is not a stand-alone program"
  s.authors = ["Chenryn"]
  s.email = 'chenlin7@staff.sina.com.cn'
  s.homepage = "http://kibana.logstash.es"
  s.require_paths = ["lib"]

  s.files = `find . -type f ! -wholename '*.svn*'`.split($\)
  s.test_files = s.files.grep(%r{^(test|spec|features)/})

  s.metadata = { "logstash_plugin" => "true", "logstash_group" => "filter" }

  s.requirements << "jar 'org.elasticsearch:elasticsearch', '1.4.0'"
  s.add_runtime_dependency "jar-dependencies"
  s.add_runtime_dependency "logstash-core", '>= 1.4.0', '< 2.0.0'
  s.add_development_dependency 'logstash-devutils'
end
```

其中:

* `s.version` 就是 milestone 的替代品，0.1.x 相当于是 milestone 0；0.9.x 相当于是 milestone 2；1.x.x 相当于是 milestone 3。
* `s.file` 默认写法是 `git ls-files`，因为默认是 git 库，如果你本身采用了 svn，或者 cvs 库，都不要紧，只要命令列出的是你需要打包进去的文件即可。
* `s.metadata` 是 logstash 的 plugin 命令在 install 的时候会提前 verify 的特殊信息，一定要保留。
* `s.add_runtime_dependency` 是定义插件依赖库的指令。如果有 jar 包依赖，则额外再加 `s.requirements`。

好了，全部完毕。下面打包：

```
gem build logstash-filter-mything.gemspec
```

运行完就会生成一个 *logstash-filter-mything-1.1.0.gem* 软件包，可以安装使用了。
