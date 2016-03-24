# 发送邮件(Email)

## 配置示例

```
output {
    email {
        to => "admin@website.com,root@website.com"
        cc => "other@website.com"
        via => "smtp"
        subject => "Warning: %{title}"
        options => {
            smtpIporHost       => "localhost",
            port               => 25,
            domain             => 'localhost.localdomain',
            userName           => nil,
            password           => nil,
            authenticationType => nil, # (plain, login and cram_md5)
            starttls           => true
        }
        htmlbody => ""
        body => ""
        attachments => ["/path/to/filename"]
    }
}
```

*注意：以上示例适用于 `Logstash 1.5` ，`options =>` 的参数配置在 `Logstash 2.0` 之后的版本已被移除，（126 邮箱发送到 qq 邮箱）示例如下：*

```
output {
    email {
		port           =>    "25"
		address        =>    "smtp.126.com"
		username       =>    "test@126.com"
		password       =>    ""
		authentication =>    "plain"
		use_tls        =>    true
		from           =>    "test@126.com"
		subject        =>    "Warning: %{title}"
		to             =>    "test@qq.com"
		via            =>    "smtp"
		body           =>    "%{message}"
    }
}
```

## 解释

*outputs/email* 插件支持 SMTP 协议和 sendmail 两种方式，通过 `via` 参数设置。SMTP 方式有较多的 options 参数可配置。sendmail 只能利用本机上的 sendmail 服务来完成 —— 文档上描述了 Mail 库支持的 sendmail 配置参数，但实际代码中没有相关处理，不要被迷惑了。。。

