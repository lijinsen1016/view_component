---
layout: default
title: 测试
parent: 使用指南
---

# 测试

使用 `render_inline` 测试辅助方法对组件进行单元测试，并对渲染输出进行断言：

```ruby
require "test_helper"

class ExampleComponentTest < ViewComponent::TestCase
  def test_render_component
    render_inline(ExampleComponent.new(title: "my title")) { "Hello, World!" }

    assert_component_rendered

    assert_selector("span[title='my title']", text: "Hello, World!")
    # 或者只断言文本：
    assert_text("Hello, World!")
  end
end
```

（若安装了 capybara gem，则可使用 Capybara 的匹配器）

_注意：`assert_selector` 默认只匹配可见元素。要不论可见性都进行匹配，请添加 `visible: false`。更多细节请参见 [Capybara 文档](https://rubydoc.info/github/jnicklas/capybara/Capybara/Node/Matchers)。_

出于调试目的，`rendered_content` 测试辅助方法会输出渲染后的 HTML。

## 配合 RSpec 使用

要在 RSpec 中启用 ViewComponent 测试辅助方法，请添加：

```ruby
# spec/rails_helper.rb
require "view_component/test_helpers"

RSpec.configure do |config|
  # ...

  config.include ViewComponent::TestHelpers, type: :component
  config.include ViewComponent::SystemSpecHelpers, type: :feature
  config.include ViewComponent::SystemSpecHelpers, type: :system
end
```

## 测试插槽

```ruby
def test_render_component
  component = ListComponent.new(title: "Fruits").tap do |c|
    c.with_item { "Apple" }
    c.with_item { "Orange" }
    c.with_extra { "<div><span>rendered html</span></div>".html_safe }
  end

  render_inline(component)

  assert_selector("ul")
  assert_selector("li", text: "Apple")
  assert_selector("li", text: "Orange")
end
```

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

## 测试带交互行为的组件

要测试带交互行为的 ViewComponent，可在系统测试中访问某个预览：

```ruby
class MyComponentSystemTest < ActionDispatch::SystemTestCase
  def test_default_preview
    visit("/rails/view_components/my_component/default")

    click_on("Open dialog")

    assert_text("Test Dialog")
  end
end
```

## 最佳实践

优先对渲染输出进行测试，而非对单个方法：

```ruby
# Good
assert_selector(".Label", text: "My label")

# Bad
assert_equal MyComponent.new.label, "My label"
```

## 不使用 `capybara`

在没有 `capybara` 的情况下，可对 `render_inline` 的返回值进行断言，它是 `Nokogiri::HTML::DocumentFragment` 的实例：

```ruby
def test_render_component
  result = render_inline(ExampleComponent.new(title: "my title")) { "Hello, World!" }

  assert_includes result.css("span[title='my title']").to_html, "Hello, World!"
end
```

## 插槽

要测试使用插槽的组件：

```ruby
def test_renders_slots_with_content
  render_inline(SlotsComponent.new(footer: "Bye!")) do |component|
    component.with_title { "Hello!" }
    component.with_body { "Have a nice day." }
  end

  assert_selector(".title", text: "Hello!")
  assert_selector(".body", text: "Have a nice day.")
end
```

## Action Pack 变体

使用 `with_variant` 辅助方法测试特定变体：

```ruby
def test_render_component_for_tablet
  with_variant :tablet do
    render_inline(ExampleComponent.new(title: "my title")) { "Hello, tablets!" }

    assert_selector("span[title='my title']", text: "Hello, tablets!")
  end
end
```

## 请求格式

使用 `with_format` 辅助方法测试特定请求格式：

```ruby
def test_render_component_as_json
  with_format :json do
    render_inline(MultipleFormatsComponent.new)

    assert_equal(rendered_json["hello"], "world")
  end
end
```

## 配置测试中使用的控制器

Since 2.27.0
{: .label }

组件测试会假定存在 `ApplicationController` 类。要为某个测试文件设置控制器，请定义 `vc_test_controller_class`：

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def vc_test_controller_class
    PublicController
  end

  def test_component_in_public_controller
    render_inline ExampleComponent.new

    assert_text "foo"
  end
end
```

要为某个测试用例配置所使用的控制器，请使用 `ViewComponent::TestHelpers` 中的 `with_controller_class`。

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def test_component_in_public_controller
    with_controller_class PublicController do
      render_inline ExampleComponent.new

      assert_text "foo"
    end
  end

  def test_component_in_authenticated_controller
    with_controller_class AuthenticatedController do
      render_inline ExampleComponent.new

      assert_text "bar"
    end
  end
end
```

## 设置 `request.path_parameters`

Since 2.31.0
{: .label }

某些 Rails 辅助方法在 `request.path_parameters` 未正确设置时无法工作，会导致 `ActionController::UrlGenerationError`。

要为某个测试用例设置 `request.path_parameters`，请使用 `ViewComponent::TestHelpers` 中的 `with_request_url`：

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def test_with_request_url
    with_request_url "/products/42" do
      render_inline ExampleComponent.new # 其中包含例如 `link_to "French", url_for(locale: "fr")`
      assert_link "French", href: "/products/42?locale=fr"
    end
  end
end
```

## 设置 `request.host`

Since 3.3.0
{: .label }

带有子域名约束的 Rails 路由要求 `request.host` 被正确设置。

要为某个测试用例设置 `request.host`，请使用 `ViewComponent::TestHelpers` 中的 `with_request_url`：

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def test_with_request_url
    with_request_url "/products/42", host: "app.example.com" do
      render_inline ExampleComponent.new # 其中包含例如受 'app' 子域名约束的 `products_path`
      assert_link "Products", href: "/products"
    end
  end
end
```

### 查询参数

Since 2.41.0
{: .label }

也可以设置查询参数：

```ruby
class ExampleComponentTest < ViewComponent::TestCase
  def test_with_request_url
    with_request_url "/products/42?locale=en" do
      render_inline ExampleComponent.new # 其中包含例如 `link_to "Recent", url_for(request.query_parameters.merge(filter: "recent"))`
      assert_link "Recent", href: "/?locale=en&filter=recent"
    end
  end
end
```

## RSpec 配置

要使用 RSpec，请添加如下内容：

```ruby
# spec/rails_helper.rb
require "view_component/test_helpers"
require "view_component/system_test_helpers"
require "capybara/rspec"

RSpec.configure do |config|
  config.include ViewComponent::TestHelpers, type: :component
  config.include ViewComponent::SystemTestHelpers, type: :component
  config.include Capybara::RSpecMatchers, type: :component
end
```

要在测试中访问 Devise 的控制器辅助方法，请添加如下内容：

```ruby
RSpec.configure do |config|
  config.include Devise::Test::ControllerHelpers, type: :component

  config.before(:each, type: :component) do
    @request = vc_test_controller.request
  end
end
```

由生成器创建的 spec 可以使用诸如 `render_inline` 之类的测试辅助方法。例如：

```ruby
require "rails_helper"

RSpec.describe ExampleComponent, type: :component do
  it "renders component" do
    render_inline(described_class.new(title: "my title")) { "Hello, World!" }

    expect(page).to have_css "span[title='my title']", text: "Hello, World!"
    # 或者只断言文本
    expect(page).to have_text "Hello, World!"
  end
end
```

要使用组件预览：

```ruby
# config/application.rb
config.view_component.previews.paths << "#{Rails.root}/spec/components/previews"
```

## 组件系统测试

将 `with_rendered_component_path` 与 `render_inline` 配合使用，对组件进行系统测试：

```rb
class ViewComponentSystemTest < ViewComponent::SystemTestCase
  def test_simple_js_interaction_in_browser_without_layout
    with_rendered_component_path(render_inline(SimpleJavascriptInteractionWithJsIncludedComponent.new)) do |path|
      visit(path)

      assert(find("[data-hidden-field]", visible: false))
      find("[data-button]", text: "Click Me To Reveal Something Cool").click
      assert(find("[data-hidden-field]", visible: true))
    end
  end
end
```

对于依赖布局的组件，请提供 `layout` 参数：

```rb
class ViewComponentSystemTest < ViewComponent::SystemTestCase
  def test_simple_js_interaction_in_browser_with_layout
    with_rendered_component_path(render_inline(SimpleJavascriptInteractionWithoutJsIncludedComponent.new), layout: "application") do |path|
      # ...
    end
  end
end
```
