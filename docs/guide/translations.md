---
layout: default
title: 翻译
parent: 使用指南
---

# 翻译

要使用 `I18n` 翻译，请添加一个伴生 YAML 文件：

```yml
# app/components/example_component.yml
en:
  hello: "Hello world!"
```

翻译也可以定义在按语言拆分的独立文件中：

```yml
# app/components/example_component.en.yml
en:
  hello: "Hello world!"

# app/components/example_component.fr.yml
fr:
  hello: "Bonjour le monde !"
```

这些文件可以在指定 `--locale` 标志时由组件生成器自动生成。

通过前导点来访问组件本地的翻译：

```erb
<%# app/components/example_component.html.erb %>
<%= t(".hello") %>
```

全局的 Rails 翻译同样可用：

```erb
<%# app/components/example_component.html.erb %>
<%= t("my.global.translation") %>
```

也可以包含命名空间在组件名之下的翻译：

```yml
# config/locales/en.yml
en:
  my_module:
    example_component:
      hello: "Hello world!"
```

```erb
<%# app/components/my_module/example_component.html.erb %>
<%= t(".hello") %>
```

通过 `helpers` 或 `I18n` 访问全局翻译：

```erb
<%# app/components/example_component.html.erb %>
<%= helpers.t("hello") %>
<%= I18n.t("hello") %>
```

## 继承

翻译会从组件的父类继承。假设父组件有一个翻译文件：

```yml
# app/components/parent_component.yml
en:
  hello: "Hello world!"
  greeting: "Cheers!"
```

该翻译在 `ParentComponent` 的子类中可用，允许直接使用或被子类覆盖：

```yml
# app/components/child_component.yml
en:
  greeting: "Howdy!"
```

```rb
# app/components/child_component.rb
class ChildComponent < ParentComponent
  def call
    t(".hello") # => "Hello world!"（继承）
    t(".greeting") # => "Howdy!"    （覆盖）
  end
end
```
