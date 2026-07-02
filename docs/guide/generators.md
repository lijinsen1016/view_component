---
layout: default
title: 生成器
parent: 使用指南
nav_order: 5
---

# 生成器

生成器接受一个组件名和一组参数。

要创建一个带有 `title` 和 `content` 属性的 `ExampleComponent`：

```console
bin/rails generate view_component:component Example title content

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  erb
      create    app/components/example_component.html.erb
```

## 生成命名空间下的组件

要生成一个命名空间下的 `Sections::ExampleComponent`：

```console
bin/rails generate view_component:component Sections::Example title content

      create  app/components/sections/example_component.rb
      invoke  test_unit
      create    test/components/sections/example_component_test.rb
      invoke  erb
      create    app/components/sections/example_component.html.erb
```

## 选项

运行生成器时可以指定选项。要全局修改默认值，请定义 [API 文档](/api.html#configuration)中所述的配置项。

生成的 ViewComponent 默认会被添加到 `app/components`。设置 `config.view_component.generate.path` 可使用其他路径。

```ruby
# config/application.rb
config.view_component.generate.path = "app/views/components"
config.eager_load_paths << Rails.root.join("app/views/components")
```

### 覆盖模板引擎

ViewComponent 内置了 `erb`、`haml` 和 `slim` 三种模板引擎的生成器，默认使用 `config.generators.template_engine` 所指定的模板引擎。

```console
bin/rails generate view_component:component Example title --template-engine slim

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  slim
      create    app/components/example_component.html.slim
```

### 覆盖测试框架

默认使用 `config.generators.test_framework`。

```console
bin/rails generate view_component:component Example title --test-framework rspec

      create  app/components/example_component.rb
      invoke  rspec
      create    spec/components/example_component_spec.rb
      invoke  erb
      create    app/components/example_component.html.erb
```

### 生成 [预览](/guide/previews.html)

Since 2.25.0
{: .label }

```console
bin/rails generate view_component:component Example title --preview

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  preview
      create    test/components/previews/example_component_preview.rb
      invoke  erb
      create    app/components/example_component.html.erb
```

### 生成 [Stimulus 控制器](/guide/javascript_and_css.html#stimulus)

Since 2.38.0
{: .label }

```console
bin/rails generate view_component:component Example title --stimulus

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  stimulus
      create    app/components/example_component_controller.js
      invoke  erb
      create    app/components/example_component.html.erb
```

要总是生成 Stimulus 控制器，请设置 `config.view_component.generate.stimulus_controller = true`。

要生成 TypeScript 控制器而非 JavaScript 控制器，可以：

- 传入 `--typescript` 选项
- 设置 `config.view_component.generate.typescript = true`

### 生成 [本地化文件](/guide/translations.html)

Since 2.47.0
{: .label }

```console
bin/rails generate view_component:component Example title --locale

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  locale
      create    app/components/example_component.yml
      invoke  erb
      create    app/components/example_component.html.erb
```

要总是生成本地化文件，请设置 `config.view_component.generate.locale = true`。

若要生成分立的本地化文件，请设置 `config.view_component.generate.distinct_locale_files = true`，会按 `I18n.available_locales` 中配置的每种语言各生成一个文件。

### 将视图放入伴生目录

Since 2.16.0
{: .label }

```console
bin/rails generate view_component:component Example title --sidecar

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  erb
      create    app/components/example_component/example_component.html.erb
```

要总是在伴生目录中生成，请设置 `config.view_component.generate.sidecar = true`。

### 使用 [内联模板](/guide/templates.html#inline)（无模板文件）

```console
bin/rails generate view_component:component Example title --inline

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  erb
```

### 使用 [call 方法](/guide/templates.html#call)（无模板文件）

```console
bin/rails generate view_component:component Example title --call

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  erb
```

### 指定父类

Since 2.41.0
{: .label }

默认情况下，若定义了 `ApplicationComponent` 则使用之，否则使用 `ViewComponent::Base`。

```console
bin/rails generate view_component:component Example title content --parent MyBaseComponent

      create  app/components/example_component.rb
      invoke  test_unit
      create    test/components/example_component_test.rb
      invoke  erb
      create    app/components/example_component.html.erb
```

要始终使用特定的父类，请设置 `config.view_component.parent_class = "MyBaseComponent"`。

### 跳过命名冲突检查

生成器会防止与已有组件重名而发生冲突。要跳过该检查并强制运行生成器，请使用 `--skip-collision-check` 或 `--force` 选项。
