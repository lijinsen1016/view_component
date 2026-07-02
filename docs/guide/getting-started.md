---
layout: default
title: 快速入门
parent: 使用指南
nav_order: 1
---

# 快速入门

## 约定

- 组件是 `ViewComponent::Base` 的子类，位于 `app/components` 目录下。常见做法是创建一个继承自 `ViewComponent::Base` 的 `ApplicationComponent`，然后让其他组件继承它。
- 组件名以 -`Component` 结尾。
- 组件的模块名使用复数形式，与控制器和 job 一致：`Users::AvatarComponent`
- 组件的命名应体现它渲染的是什么，而非它接收的是什么。（应使用 `AvatarComponent` 而非 `UserComponent`）

## 安装

在 `Gemfile` 中添加：

```ruby
gem "view_component"
```

## 快速上手

使用组件生成器创建一个新的 ViewComponent。

生成器接受一个组件名和一组参数：

```console
bin/rails generate view_component:component Example title

      invoke  test_unit
      create  test/components/example_component_test.rb
      create  app/components/example_component.rb
      create  app/components/example_component.html.erb
```

用于自定义生成器的可用选项详见 [生成器](/guide/generators.html) 页面。

## 实现

ViewComponent 是一个继承自 `ViewComponent::Base` 的 Ruby 类：

```ruby
class ExampleComponent < ViewComponent::Base
  erb_template <<-ERB
    <span title="<%= @title %>"><%= content %></span>
  ERB

  def initialize(title:)
    @title = title
  end
end
```

以 block 形式传给 ViewComponent 的内容会被捕获并赋给 `content` 访问器。

在视图中渲染：

```erb
<%= render(ExampleComponent.new(title: "my title")) do %>
  Hello, World!
<% end %>
```

返回：

```html
<span title="my title">Hello, World!</span>
```

## `#with_content`

Since 2.31.0
{: .label }

也可以通过调用 `#with_content` 将字符串内容传给 ViewComponent：

```erb
<%= render(ExampleComponent.new(title: "my title").with_content("Hello, World!")) %>
```

## 在控制器中渲染

也可以在控制器中渲染 ViewComponent：

```ruby
def show
  render(ExampleComponent.new(title: "My Title"))
end
```

_注意：在控制器中无法通过 block 向组件传递内容。请改用 `with_content`。_

当配合 [turbo-rails](https://github.com/hotwired/turbo-rails) 使用 turbo frame 时，需将 `content_type` 设为 `text/html`：

```ruby
def create
  render(ExampleComponent.new, content_type: "text/html")
end
```

### 在控制器动作中将 ViewComponent 渲染为字符串

当需要多次渲染同一个组件以便后续复用时，请使用 `render_in`：

```rb
class PagesController < ApplicationController
  def index
    # 不可行：会触发 `AbstractController::DoubleRenderError`
    # @reusable_icon = render IconComponent.new("close")

    # 不可行：会把整个 index 视图渲染为字符串
    # @reusable_icon = render_to_string IconComponent.new("close")

    # 可行：将组件渲染为字符串
    @reusable_icon = IconComponent.new("close").render_in(view_context)
  end
end
```

### 在视图上下文之外渲染 ViewComponent

要在视图上下文之外渲染 ViewComponent（例如在后台 job、markdown 处理器等中），请实例化一个 Rails 控制器：

```ruby
ApplicationController.new.view_context.render(MyComponent.new)
```
