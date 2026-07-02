---
layout: default
title: 辅助方法
parent: 使用指南
---

# 辅助方法

辅助方法必须先被引入（include）才能使用：

```ruby
module IconHelper
  def icon(name)
    tag.i data: {feather: name.to_s}
  end
end

class UserComponent < ViewComponent::Base
  include IconHelper

  def profile_icon
    icon :user
  end
end
```

## 代理

Since 1.5.0
{: .label }

或者，通过 `helpers` 代理来访问辅助方法：

```ruby
class UserComponent < ViewComponent::Base
  def profile_icon
    helpers.icon :user
  end
end
```

它可以配合 `delegate` 使用：

```ruby
class UserComponent < ViewComponent::Base
  delegate :icon, to: :helpers

  def profile_icon
    icon :user
  end
end
```

## 嵌套 URL 辅助方法

Rails 的嵌套 URL 辅助方法在某些情况下会隐式依赖当前的 `request`。由于 ViewComponent 的设计初衷是让组件能够在不同上下文中复用，因此应当向嵌套 URL 辅助方法显式传入选项：

```ruby
# bad
edit_user_path # 隐式依赖当前 request 来提供 `user`

# good
edit_user_path(user: current_user)
```

或者使用 `helpers` 代理：

```ruby
helpers.edit_user_path
```
