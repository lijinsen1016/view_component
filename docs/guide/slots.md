---
layout: default
title: 插槽
parent: 使用指南
---

# 插槽

Since 2.12.0
{: .label }

除了 `content` 访问器之外，ViewComponent 还可以通过插槽（slots）接收内容。可以把插槽理解为一种渲染多块内容（包括其他组件）的方式。

插槽通过 `renders_one` 和 `renders_many` 来定义：

- `renders_one` 定义每个组件最多渲染一次的插槽：`renders_one :header`
- `renders_many` 定义每个组件可多次渲染的插槽：`renders_many :posts`

如果这两个方法没有提供第二个参数，则会注册一个**透传插槽（passthrough slot）**。传入的任何内容都可以在这些插槽中无限制地渲染。

例如：

```ruby
# blog_component.rb
class BlogComponent < ViewComponent::Base
  renders_one :header
  renders_many :posts
end
```

要渲染 `renders_one` 插槽，调用该插槽的名称即可。

要渲染 `renders_many` 插槽，对该插槽的名称进行迭代：

```erb
<%# blog_component.html.erb %>
<h1><%= header %></h1>

<% posts.each do |post| %>
  <%= post %>
<% end %>
```

```erb
<%# index.html.erb %>
<%= render BlogComponent.new do |component| %>
  <% component.with_header do %>
    <%= link_to "My blog", root_path %>
  <% end %>

  <% BlogPost.all.each do |blog_post| %>
    <% component.with_post do %>
      <%= link_to blog_post.name, blog_post.url %>
    <% end %>
  <% end %>
<% end %>
```

返回：

```erb
<h1><a href="/">My blog</a></h1>

<a href="/blog/first-post">First post</a>
<a href="/blog/second-post">Second post</a>
```

## 谓词方法

Since 2.50.0
{: .label }

要判断某个插槽是否已被传入组件，请使用提供的 `#{slot_name}?` 方法。

```erb
<%# blog_component.html.erb %>
<% if header? %>
  <h1><%= header %></h1>
<% end %>

<% if posts? %>
  <div class="posts">
    <% posts.each do |post| %>
      <%= post %>
    <% end %>
  </div>
<% else %>
  <p>No post yet.</p>
<% end %>
```

## 组件插槽

插槽也可以渲染其他组件。将组件名作为第二个参数传入，即可定义组件插槽。

调用组件插槽时传入的参数会用于初始化该组件并将其渲染。还可以传入一个 block 来设置组件的内容。

```ruby
# blog_component.rb
class BlogComponent < ViewComponent::Base
  # 由于 `HeaderComponent` 嵌套在本组件内部，
  # 我们必须以字符串形式引用它，而不是使用类名。
  renders_one :header, "HeaderComponent"

  # `PostComponent` 定义在另一个文件中，因此可以用类名引用它。
  renders_many :posts, PostComponent

  class HeaderComponent < ViewComponent::Base
    attr_reader :classes

    def initialize(classes:)
      @classes = classes
    end

    def call
      content_tag :h1, content, {class: classes}
    end
  end
end
```

```erb
<%# blog_component.html.erb %>
<%= header %>

<% posts.each do |post| %>
  <%= post %>
<% end %>
```

```erb
<%# index.html.erb %>
<%= render BlogComponent.new do |component| %>
  <% component.with_header(classes: "") do %>
    <%= link_to "My Site", root_path %>
  <% end %>

  <% component.with_post(title: "My blog post") do %>
    Really interesting stuff.
  <% end %>

  <% component.with_post(title: "Another post!") do %>
    Blog every day.
  <% end %>
<% end %>
```

## 引用插槽

由于传给插槽的内容是在组件初始化之后才注册的，因此不能在初始化方法中引用插槽内容。引用插槽内容的一种方式是使用 `before_render` [生命周期方法](/guide/lifecycle)：

```ruby
# blog_component.rb
class BlogComponent < ViewComponent::Base
  renders_one :image
  renders_many :posts

  def before_render
    @post_container_classes = "PostContainer--hasImage" if image.present?
  end
end
```

```erb
<%# blog_component.html.erb %>
<% posts.each do |post| %>
  <div class="<%= @post_container_classes %>">
    <%= image if image? %>
    <%= post %>
  </div>
<% end %>
```

## Lambda 插槽

也可以将插槽定义为一个返回待渲染内容（字符串或 ViewComponent 实例）的 lambda。在编写独立组件可能并不必要时，lambda 插槽会很有用，例如配合 `content_tag` 这类辅助方法使用，或用作带特定默认值的另一个 ViewComponent 的包装器：

```ruby
class BlogComponent < ViewComponent::Base
  renders_one :header, ->(classes:) do
    # 这部分逻辑目前还不够复杂，不足以成为独立组件，
    # 所以我们使用 lambda 插槽。如果它变得更复杂，
    # 就应该被抽取为独立的 ViewComponent，并通过组件插槽在此渲染。
    content_tag :h1 do
      link_to title, root_path, {class: classes}
    end
  end

  # 也可以返回一个带有预设默认值的另一个 ViewComponent：
  renders_many :posts, ->(title:, classes:) do
    PostComponent.new(title: title, classes: "my-default-class " + classes)
  end
end
```

lambda 插槽能够访问父级 ViewComponent 的状态：

```ruby
class TableComponent < ViewComponent::Base
  renders_one :header, -> do
    HeaderComponent.new(selectable: @selectable)
  end

  def initialize(selectable: false)
    @selectable = selectable
  end
end
```

要通过 block 为 lambda 插槽提供内容，请添加一个 block 参数。通过调用该 block 的 `call` 方法来渲染内容，或将该 block 直接传给 `content_tag`：

```ruby
class BlogComponent < ViewComponent::Base
  renders_one :header, ->(classes:, &block) do
    content_tag :h1, class: classes, &block
  end
end
```

_注意：虽然 lambda 会在调用 `with_*` 方法时被调用，但返回的组件直到首次使用时才会被渲染。_

## 渲染集合

Since 2.23.0
{: .label }

`renders_many` 插槽还可以使用复数形式的 setter（本例中为 `links`）传入一个集合：

```ruby
# navigation_component.rb
class NavigationComponent < ViewComponent::Base
  renders_many :links, "LinkComponent"

  class LinkComponent < ViewComponent::Base
    def initialize(name:, href:)
      @name = name
      @href = href
    end
  end
end
```

```erb
<%# navigation_component.html.erb %>
<% links.each do |link| %>
  <%= link %>
<% end %>
```

```erb
<%# index.html.erb %>
<%= render(NavigationComponent.new) do |component| %>
  <% component.with_links([
    { name: "Home", href: "/" },
    { name: "Pricing", href: "/pricing" },
    { name: "Sign Up", href: "/sign-up" },
  ]) %>
<% end %>
```

## `#with_SLOT_NAME_content`

Since 3.0.0
{: .label }

如果不需要向插槽传递参数，可以使用 `#with_SLOT_NAME_content` 来设置插槽内容：

```erb
<%= render(BlogComponent.new.with_header_content("My blog")) %>
```

## `#with_content`

Since 2.31.0
{: .label }

也可以使用 `#with_content` 来设置插槽内容：

```erb
<%= render BlogComponent.new do |component| %>
  <% component.with_header(classes: "title").with_content("My blog") %>
<% end %>
```

## 多态插槽

Since 2.42.0
{: .label }

多态插槽可以渲染若干可能插槽中的一种。

例如，考虑这样一个列表项组件，它可以带图标或头像作为可视化元素来渲染。`visual` 插槽接收一个将类型映射到插槽定义的哈希：

```ruby
class ListItemComponent < ViewComponent::Base
  renders_one :visual, types: {
    icon: IconComponent,
    avatar: lambda { |**system_arguments|
      AvatarComponent.new(size: 16, **system_arguments)
    }
  }
end
```

**注意**：`types` 哈希的值可以是任何合法的插槽定义，包括组件类、字符串或 lambda。

填充 `visual` 插槽通过调用相应的插槽方法来完成：

```erb
<%= render ListItemComponent.new do |component| %>
  <% component.with_visual_avatar(src: "http://some-site.com/my_avatar.jpg", alt: "username") do %>
    Profile
  <% end >
<% end %>
<%= render ListItemComponent.new do |component| %>
  <% component.with_visual_icon(icon: :key) do %>
    Security Settings
  <% end >
<% end %>
```

要判断某个多态插槽是否已被传入组件，请使用 `#{slot_name}?` 方法。

```erb
<% if visual? %>
  <%= visual %>
<% else %>
  <span class="visual-placeholder">N/A</span>
<% end %>
```

### 自定义多态插槽 setter

Since 3.1.0
{: .label }

通过为 `type` 的值指定一个嵌套哈希，可以自定义插槽 setter：

```ruby
class ListItemComponent < ViewComponent::Base
  renders_one :visual, types: {
    icon: {renders: IconComponent, as: :icon_visual},
    avatar: {
      renders: lambda { |**system_arguments| AvatarComponent.new(size: 16, **system_arguments) },
      as: :avatar_visual
    }
  }
end
```

此时 setter 变为 `#with_icon_visual` 和 `#with_avatar_visual`，而非默认的 `#with_visual_icon` 和 `#with_visual_avatar`。插槽的 getter 仍为 `#visual`。

## `#default_SLOT_NAME`

Since 4.0.0
{: .label }

要为插槽提供默认值，请定义 `default_SLOT_NAME` 方法：

```ruby
class SlotableDefaultComponent < ViewComponent::Base
  renders_one :header

  def default_header
    "Hello, World!"
  end
end
```

`default_SLOT_NAME` 也可以返回一个待渲染的组件实例：

```ruby
class SlotableDefaultInstanceComponent < ViewComponent::Base
  renders_one :header

  def default_header
    MyComponent.new
  end
end
```
