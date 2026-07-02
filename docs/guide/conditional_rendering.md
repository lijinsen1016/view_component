---
layout: default
title: 条件渲染
parent: 使用指南
nav_order: 3
---

# 条件渲染

Since 1.8.0
{: .label }

组件可以实现 `#render?` 方法，该方法会在初始化之后被调用，以决定组件是否应当渲染。

传统上，是否渲染某个视图的逻辑既可以放在组件模板中：

```erb
<% if user.requires_confirmation? %>
  <div class="alert">Please confirm your email address.</div>
<% end %>
```

也可以放在渲染该组件的视图里：

```erb
<% if current_user.requires_confirmation? %>
  <%= render(ConfirmEmailComponent.new(user: current_user)) %>
<% end %>
```

使用 `#render?` 钩子可以简化视图：

```ruby
class ConfirmEmailComponent < ViewComponent::Base
  erb_template <<-ERB
    <div class="banner">
      Please confirm your email address.
    </div>
  ERB

  def initialize(user:)
    @user = user
  end

  def render?
    @user.requires_confirmation?
  end
end
```

```erb
<%= render(ConfirmEmailComponent.new(user: current_user)) %>
```

_要断言组件是否已被渲染，请使用 `ViewComponent::TestHelpers` 中的 `assert_component_rendered` / `refute_component_rendered`。_
