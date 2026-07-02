---
layout: default
title: 插桩
parent: 使用指南
nav_order: 7
---

# 插桩

Since 2.34.0
{: .label }

要启用 ActiveSupport notifications，请使用 `instrumentation_enabled` 选项：

```ruby
# config/application.rb
# 为所有 ViewComponent 启用 ActiveSupport notifications
config.view_component.instrumentation_enabled = true
```

订阅该事件：

```ruby
ActiveSupport::Notifications.subscribe("render.view_component") do |event| # 或 !render.view_component
  event.name    # => "render.view_component"
  event.payload # => { name: "MyComponent", identifier: "/Users/mona/project/app/components/my_component.rb", view_identifier: "/Users/mona/project/app/components/my_component.html.erb" }
end
```

_注意：启用插桩会降低 ViewComponent 的性能。_

## 编译插桩

Since 4.8.0
{: .label }

ViewComponent 还会通过 `compile.view_component` 事件对启动时的预编译进行插桩。当 `config.eager_load` 为 `true` 时，该事件总是会发送（无需任何配置）：

```ruby
ActiveSupport::Notifications.subscribe("compile.view_component") do |event|
  event.name     # => "compile.view_component"
  event.duration # => 123.45（毫秒）
end
```

## 在浏览器开发者工具中查看插桩汇总

在 Rails 7 中，当 `render.view_component` 配合 `config.server_timing = true`（开发环境默认开启）使用时，浏览器开发者工具会在 Network > Timing 下以 `render.view_component` 为键显示总耗时信息。

![Browser showing the Server Timing data in the browser dev tools](../images/viewing_instrumentation_sums_in_browser_dev_tools.png "Server Timing data in the browser dev tools")

## 在 rack-mini-profiler 中查看插桩细分

[rack-mini-profiler gem](https://rubygems.org/gems/rack-mini-profiler) 是用于对基于 rack 的 Ruby 应用进行性能分析的常用工具。

要对 ViewComponent 渲染与视图及局部模板一同进行性能分析：

```ruby
# config/environments/development.rb
# 对 ViewComponent 的渲染进行性能分析
Rack::MiniProfilerRails.subscribe("render.view_component") do |_name, start, finish, _id, payload|
  Rack::MiniProfilerRails.render_notification_handler(
    Rack::MiniProfilerRails.shorten_identifier(payload[:identifier]),
    finish,
    start
  )
end
```
