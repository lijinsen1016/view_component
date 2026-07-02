---
layout: default
title: 集合
parent: 使用指南
---

# 集合

Since 2.1.0
{: .label }

与 [Rails 局部模板](https://guides.rubyonrails.org/layouts_and_rendering.html#rendering-collections)类似，可以使用 `with_collection` 通过 ViewComponent 渲染集合：

```erb
<%= render(ProductComponent.with_collection(@products)) %>
```

```ruby
class ProductComponent < ViewComponent::Base
  def initialize(product:)
    @product = product
  end
end
```

[默认情况下](https://github.com/viewcomponent/view_component/blob/89f8fab4609c1ef2467cf434d283864b3c754473/lib/view_component/base.rb#L249)，会以组件名来命名从集合传入组件的参数。

## `with_collection_parameter`

使用 `with_collection_parameter` 可更改集合参数的名称：

```ruby
class ProductComponent < ViewComponent::Base
  with_collection_parameter :item

  def initialize(item:)
    @item = item
  end
end
```

## 额外参数

除集合之外的额外参数会被传递给每个组件实例：

```erb
<%= render(ProductComponent.with_collection(@products, notice: "hi")) %>
```

```ruby
class ProductComponent < ViewComponent::Base
  with_collection_parameter :item

  erb_template <<-ERB
    <li>
      <h2><%= @item.name %></h2>
      <span><%= @notice %></span>
    </li>
  ERB

  def initialize(item:, notice:)
    @item = item
    @notice = notice
  end
end
```

## 集合计数器

Since 2.5.0
{: .label }

ViewComponent 会定义一个与上面参数名同名并后缀 `_counter` 的计数器变量。要访问该变量，请在 `initialize` 中将其作为参数添加：

```ruby
class ProductComponent < ViewComponent::Base
  erb_template <<-ERB
    <li>
      <%= @counter %> <%= @product.name %>
    </li>
  ERB

  def initialize(product:, product_counter:)
    @product = product
    @counter = product_counter
  end
end
```

## 集合迭代上下文

Since 2.33.0
{: .label }

ViewComponent 会定义一个与上面参数名同名并后缀 `_iteration` 的迭代变量。它会为集合中的组件提供关于迭代的上下文信息（`#size`、`#index`、`#first?` 和 `#last?`）。

要访问该变量，请在 `initialize` 中将其作为参数添加：

```ruby
class ProductComponent < ViewComponent::Base
  erb_template <<-ERB
    <li class="<%= "featured" if @iteration.first? %>">
      <%= @product.name %>
    </li>
  ERB

  def initialize(product:, product_iteration:)
    @product = product
    @iteration = product_iteration
  end
end
```

## 间隔组件

Since 3.20.0
{: .label }

将 `:spacer_component` 设置为一个已实例化的组件，它会在各项之间被渲染：

```erb
<%= render(ProductComponent.with_collection(@products, spacer_component: SpacerComponent.new)) %>
```

这会在各 `ProductComponent` 之间渲染 SpacerComponent 组件。
