---
layout: default
title: 模板
parent: 使用指南
---

# 模板

ViewComponent 会包裹一个模板（如果使用 [variants](https://guides.rubyonrails.org/layouts_and_rendering.html#the-variants-option)，则是多个模板），模板可以通过以下几种方式之一来定义：

## 内联

Since 3.0.0
{: .label }

要在组件内部定义模板，请调用 `.TEMPLATE_HANDLER_template` 宏：

```ruby
class InlineErbComponent < ViewComponent::Base
  erb_template <<~ERB
    <h1>Hello, <%= @name %>!</h1>
  ERB

  def initialize(name)
    @name = name
  end
end
```

### 插值

使用 Slim 时，插值必须进行转义，否则它们会在 ViewComponent 类的上下文中被求值。

```ruby
class InlineSlimComponent < ViewComponent::Base
  slim_template <<~SLIM
    p Hello, #{name}!
    p Hello, \#{name}!
  SLIM

  def name
    "World"
  end
end
```

将渲染为：

    <p>Hello InlineSlimComponent!</p>
    <p>Hello World!</p>

## 同级文件

将模板文件放在组件旁边：

```console
app/components
├── ...
├── example_component.rb
├── example_component.html.erb
├── ...
```

## 子目录

Since 2.7.0
{: .label }

作为另一种选择，视图及其他资源文件可以放在与组件同名的子目录中：

```console
app/components
├── ...
├── example_component.rb
├── example_component
|   ├── example_component.html.erb
├── ...
```

要生成带有伴生目录的组件，请使用 `--sidecar` 标志：

```console
bin/rails generate view_component:component Example title --sidecar
  invoke  test_unit
  create  test/components/example_component_test.rb
  create  app/components/example_component.rb
  create  app/components/example_component/example_component.html.erb
```

## `#call`

Since 1.16.0
{: .label }

通过定义 `call` 方法，ViewComponent 可以在没有模板文件的情况下渲染：

```ruby
# app/components/inline_component.rb
class InlineComponent < ViewComponent::Base
  def call
    if active?
      link_to "Cancel integration", integration_path, method: :delete
    else
      link_to "Integrate now!", integration_path
    end
  end
end
```

也可以为 Action Pack 的变体定义专门的方法（本例中为 `phone`）：

```ruby
class InlineVariantComponent < ViewComponent::Base
  def call_phone
    link_to "Phone", phone_path
  end

  def call
    link_to "Default", default_path
  end
end
```

_**注意**：`call_*` 方法必须为 public。_

## 继承

Since 2.19.0
{: .label }

组件子类若未定义自己的模板，则会继承父组件的模板。

```ruby
# 如果 MyLinkComponent 未定义模板，
# 它会回退到 `LinkComponent` 的模板。
class MyLinkComponent < LinkComponent
end
```

### 渲染父类模板

Since 2.55.0
{: .label }

要从子类的模板中渲染父类组件的模板，请使用 `#render_parent`：

```erb
<%# my_link_component.html.erb %>
<div class="base-component-template">
  <%= render_parent %>
</div>
```

如果父类支持当前变体，则会自动渲染该变体。

`#render_parent` 也适用于内联模板：

```ruby
class MyComponent < ViewComponent::Base
  erb_template <<~ERB
    <div>
      <%= render_parent %>
    </div>
  ERB
end
```

请注意，`#render_parent` 不会返回字符串。若需要字符串，例如在 `#call` 方法中，请改调 `#render_parent_to_string`。例如：

```ruby
class MyComponent < ViewComponent::Base
  def call
    content_tag("div") do
      render_parent_to_string
    end
  end
end
```

## 行尾空白

代码编辑器通常会按 Unix 标准在源文件末尾追加一个换行符。组件模板中若包含行尾空白，可能会在 HTML 中产生多余的空白，例如当组件在句末句号之前渲染时。

要去除组件模板的行尾空白，请使用 `strip_trailing_whitespace` 类方法。

```ruby
class MyComponent < ViewComponent::Base
  # 去除空白
  strip_trailing_whitespace

  # 不去除空白
  strip_trailing_whitespace(false)
end
```
