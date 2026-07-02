---
layout: default
title: 配置
parent: 使用指南
nav_order: 4
---

# 配置

要配置 ViewComponent，请在 `config/ENVIRONMENT.rb` 中设置相关选项：

```ruby
MyApplication.configure do
  config.view_component.instrumentation_enabled = true
  config.view_component.generate.path = "app/custom_components"
  config.view_component.previews.controller = "MyPreviewController"
end
```

可用选项列表请参见 [/api](/api#configuration)。
