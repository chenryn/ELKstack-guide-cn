# 自己写一个插件

前面已经提过在运行 logstash 的时候，可以通过 `--pluginpath` 参数来加载自己写的插件。那么，插件又该怎么写呢？

## 插件格式

一个标准的 logstash 输入插件格式如下：

```ruby
require 'logstash/namespace'
require 'logstash/inputs/base'
class LogStash::Inputs::MyPlugin < LogStash::Inputs::Base
  config_name 'myplugin'
  default :codec, "line"
  config :myoption_key, :validate => :string, :default => 'myoption_value'
  public def register
  end
  public def run(queue)
  end
end
```

其中大多数语句在过滤器和输出阶段是共有的。

* config\_name 用来定义该插件写在 logstash 配置文件里的名字；
* config 可以定义很多个，即该插件在 logstash 配置文件中的可配置参数。logstash 很温馨的提供了验证方法，确保接收的数据是你期望的数据类型；
* register logstash 在启动的时候运行的函数，一些需要常驻内存的数据，可以在这一步先完成。比如对象初始化，*filters/ruby* 插件中的 `init` 语句等。

## 插件的关键方法

输入插件独有的是 run 方法。在 run 方法中，必须实现一个长期运行的程序(最简单的就是 loop 指令)。然后在每次收到数据并处理成 `event`，这段示例在上一节演示 Logstash::Event 的生成时已经介绍过。最后，一定要调用 `queue << event` 语句。一个输入流程就算是完成了。

而如果是过滤器插件，对应修改成：

```ruby
require 'logstash/filters/base'
class LogStash::Filters::MyPlugin < LogStash::Filters::Base
  config_name 'myplugin'
  public def register
  end
  public def filter(event)

    filter_matched(event)
  end
end
```

其中，`filter_matched` 是在 filter 函数完成本插件自己的处理逻辑之后一定要调用的。

输出插件则是：

```ruby
require 'logstash/outputs/base'
class LogStash::Outputs::MyPlugin < LogStash::Outputs::Base
  config_name 'myplugin'
  concurrency :single
  public def register
  end
  public def multi_receive(events)
  end
end
```

这里和过去的版本最明显的差别是，处理方法改成了 `multi_receive`，而不是 `receive`。因为新的 pipeline 机制是批量传递数据给输出插件的。不过为了兼容过去的插件，`LogStash::Outputs::Base` 基类中的 `multi_receive` 实现继续迭代调用了 `receive`。

另一个是新出现的配置 `concurrency`，代表着本插件是否 threadsafe，并由此取代了过去的 workers 选项。可选项为：`single` 和 `shared`。

* single 表示本插件是非线程安全的，必须在各 pipeline workers 之间同一时刻只有一个运行。
* shared 表示本插件是线程安全的，每个 pipeline workers 之间可以独立运行，这也就意味着插件作者要自己在 `multi_receive` 里调用 Mutexes。

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
  s.summary = "This mything filter is just for Elastic Stack Guide example"
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
