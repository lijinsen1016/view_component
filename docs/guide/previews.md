---
layout: default
title: 预览
parent: 使用指南
nav_order: 9
---

# 预览

`ViewComponent::Preview` 与 `ActionMailer::Preview` 类似，提供了一种以隔离方式快速预览组件的途径：

```ruby
# test/components/previews/example_component_preview.rb
class ExampleComponentPreview < ViewComponent::Preview
  def with_default_title
    render(ExampleComponent.new(title: "Example component default"))
  end

  def with_content_block
    render(ExampleComponent.new(title: "This component accepts a block of content")) do
      tag.div do
        content_tag(:span, "Hello")
      end
    end
  end
end
```

然后通过以下地址访问生成的预览：

* `/rails/view_components/example_component/with_default_title`
* `/rails/view_components/example_component/with_content_block`

_若希望获得更具交互性的体验，可考虑使用 [Lookbook](https://lookbook.build) 或 [ViewComponent::Storybook](https://github.com/jonspalmer/view_component_storybook)。_

## 传参

通过将动态值设为参数，即可从 URL 参数中获取它们：

```ruby
# test/components/previews/example_component_preview.rb
class ExampleComponentPreview < ViewComponent::Preview
  def with_dynamic_title(title: "Example component default")
    render(ExampleComponent.new(title: title))
  end
end
```

然后传入一个值：`/rails/view_components/example_component/with_dynamic_title?title=Custom+title`。

## 将预览作为测试用例

Since 2.56.0
{: .label }

使用 `render_preview(name)` 可在 ViewComponent 单元测试中渲染预览：

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def test_render_preview
    render_preview(:with_default_title)

    assert_text("Example component default")
  end
end
```

也可以传入参数：

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def test_render_preview
    render_preview(:with_default_title, params: {message: "Hello, world!"})

    assert_text("Hello, world!")
  end
end
```

默认情况下，预览类会根据当前测试文件的名称推断。使用 `from` 可显式指定：

```ruby
class ExampleTest < ViewComponent::TestCase
  def test_render_preview
    render_preview(:with_default_title, from: ExampleComponentPreview)

    assert_text("Hello, world!")
  end
end
```

## 辅助方法

`ViewComponent::Preview` 基类包含了
[`ActionView::Helpers::TagHelper`](https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html)，它提供了 [`tag`](https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html#method-i-tag)
和 [`content_tag`](https://api.rubyonrails.org/classes/ActionView/Helpers/TagHelper.html#method-i-content_tag) 视图辅助方法。

## 布局

预览默认使用应用布局渲染，但可以通过 `layout` 选项使用特定布局：

```ruby
# test/components/previews/example_component_preview.rb
class ExampleComponentPreview < ViewComponent::Preview
  layout "admin"
end
```

要为单个预览及预览索引页设置自定义布局，请设置 `default_preview_layout`：

```ruby
# config/application.rb
# 将默认布局设为 app/views/layouts/component_preview.html.erb
config.view_component.previews.default_layout = "component_preview"
```

## 预览路径

预览类位于 `test/components/previews`，可使用 `previews.paths` 选项配置：

```ruby
# config/application.rb
config.view_component.previews.paths << "#{Rails.root}/lib/component_previews"
```

## 预览路由

预览默认由 `/rails/view_components` 提供。要使用其他端点，请设置 `previews.route` 选项：

```ruby
# config/application.rb
config.view_component.previews.route = "/previews"
```

## 预览模板

对于预览 `test/components/previews/cell_component_preview.rb`，模板文件可定义在 `test/components/previews/cell_component_preview/` 下：

```ruby
# test/components/previews/cell_component_preview.rb
class CellComponentPreview < ViewComponent::Preview
  def default
  end
end
```

```erb
<%# test/components/previews/cell_component_preview/default.html.erb %>
<table class="table">
  <tbody>
    <tr>
      <%= render CellComponent.new %>
    </tr>
  </tbody>
</div>
```

要为预览模板使用其他位置，请传入 `template` 参数：
（路径应相对于 `config.view_component.previews.paths`）

```ruby
# test/components/previews/cell_component_preview.rb
class CellComponentPreview < ViewComponent::Preview
  def default
    render_with_template(template: "custom_cell_component_preview/my_preview_template")
  end
end
```

来自 `params` 的值可通过 `locals` 访问：

```ruby
# test/components/previews/cell_component_preview.rb
class CellComponentPreview < ViewComponent::Preview
  def default(title: "Default title", subtitle: "A subtitle")
    render_with_template(locals: {
      title: title,
      subtitle: subtitle
    })
  end
end
```

这样即可传入值：`/rails/view_components/cell_component/default?title=Custom+title&subtitle=Another+subtitle`。

## 配置预览控制器

使用 `previews.controller` 选项扩展预览，以添加认证、授权、before action 等：

```ruby
# config/application.rb
config.view_component.previews.controller = "MyPreviewController"
```

然后在控制器中引入 `PreviewActions`：

```ruby
class MyPreviewController < ActionController::Base
  include ViewComponent::PreviewActions
end
```

## 启用预览

预览在测试和开发环境中默认启用。要启用或禁用预览，请使用 `previews.enabled` 选项：

```ruby
# config/environments/test.rb
config.view_component.previews.enabled = false
```

## 配合 RSpec 使用

在 RSpec 中使用预览时，请将 `test/components` 替换为 `spec/components`，并更新 `previews.paths`：

```ruby
# config/application.rb
config.view_component.previews.paths << "#{Rails.root}/spec/components/previews"
```
