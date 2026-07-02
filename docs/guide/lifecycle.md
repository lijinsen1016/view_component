---
layout: default
title: 生命周期
parent: 使用指南
nav_order: 8
---

# 生命周期

## `#before_render`

Since 2.8.0
{: .label }

定义 `before_render` 方法，它会在组件渲染之前被调用，此时可以使用 `helpers`：

```ruby
# app/components/example_component.rb
class ExampleComponent < ViewComponent::Base
  def before_render
    @my_icon = helpers.star_icon
  end
end
```

## `#around_render`

Since 4.0.0.rc2
{: .label }

定义 `around_render` 方法，让它包覆组件的渲染过程：

```ruby
# app/components/example_component.rb
class ExampleComponent < ViewComponent::Base
  def around_render
    MyInstrumenter.instrument do
      yield
    end
  end
end
```
