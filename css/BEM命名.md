---
typora-root-url: ..
---

## 什么是block

1. 一个可以被重用的独立的页面组件，在`html`中，一个块由`class`属性来表示
2. 一个block的命名需要表示出它的用途，比如它是什么，而不是描述它的外观

```html
<!-- `header` block -->
<header class="header">
    <!-- Nested `logo` block -->
    <div class="logo"></div>

    <!-- Nested `search-form` block -->
    <form class="search-form"></form>
</header>
```

## 什么是element

1. 它是一个块中的不可分割的一部分，不能被单独使用
2. 一个element的命名需要描述它的用途，不是描述它的外观
3. 它的命名以`__`为前缀
4. 一个element只能是一个block的一部分，不能是一个element的一部分，即不能出现下面的分级结构`block__elem1__elem2`

```html
<!-- `search-form` block -->
<form class="search-form">
    <!-- `input` element in the `search-form` block -->
    <input class="search-form__input">

    <!-- `button` element in the `search-form` block -->
    <button class="search-form__button">Search</button>
</form>
```

```html
<!--
    Correct. The structure of the full element name follows the pattern:
    `block-name__element-name`
-->
<form class="search-form">
    <div class="search-form__content">
        <input class="search-form__input">

        <button class="search-form__button">Search</button>
    </div>
</form>

<!--
    Incorrect. The structure of the full element name doesn't follow the pattern:
    `block-name__element-name`
-->
<form class="search-form">
    <div class="search-form__content">
        <!-- Recommended: `search-form__input` or `search-form__content-input` -->
        <input class="search-form__content__input">

        <!-- Recommended: `search-form__button` or `search-form__content-button` -->
        <button class="search-form__content__button">Search</button>
    </div>
</form>
```

5. 一个block在`dom`树中有以下的文档结构:

```html
<div class="block">
    <div class="block__elem1">
        <div class="block__elem2">
            <div class="block__elem3"></div>
        </div>
    </div>
</div>
```

6. 一个block的结构同样也可以表示成一串列表

```css
.block {}
.block__elem1 {}
.block__elem2 {}
.block__elem3 {}
```

7. 这样子的表示方式可以不需要修改代码就可以改变`dom`树结构

```html
<div class="block">
    <div class="block__elem1">
        <div class="block__elem2"></div>
    </div>

    <div class="block__elem3"></div>
</div>
```

## 什么时候使用block

当这部分代码（不与页面中正在创建的组件相关）需要被重用时

## 什么时候使用element

1. 当这部分代码不能独立于某个block被使用时
2. 另一种情况是，当需要将一个element分成几个小的子element时，但在`bem`的思想中，不能创建`elements`的`subelements`，在这种情形下，不是创建一个`element`而是创建一个`service block`

## 什么是modifier

1. 一个modifier描述的是一个block或element的外观、状态或行为
2. 它的命名由`_`作为前缀

## modifier的类型

### boolean类型

1. 当modifier的出现或者不出现很重要时，那么它所表示的值就不是很必要，比如`disabled`属性，如果该modifier出现时，那么它的值就表示为true
2. 完整的命名模式如下：

```
block-name_modifier-name
block-name__element-name_modifier-name
```

```html
<!-- The `search-form` block has the `focused` Boolean modifier -->
<form class="search-form search-form_focused">
    <input class="search-form__input">

    <!-- The `button` element has the `disabled` Boolean modifier -->
    <button class="search-form__button search-form__button_disabled">Search</button>
</form>
```

### key-value类型

1. 当modifier所表示的值很重要时，
2. 完整的命名模式如下：

```
block-name_modifier-name_modifier-value

block-name__element-name_modifier-name_modifier-value
```

```html
<!-- The `search-form` block has the `theme` modifier with the value `islands` -->
<form class="search-form search-form_theme_islands">
    <input class="search-form__input">

    <!-- The `button` element has the `size` modifier with the value `m` -->
    <button class="search-form__button search-form__button_size_m">Search</button>
</form>

<!-- You can't use two identical modifiers with different values simultaneously -->
<form class="search-form
             search-form_theme_islands
             search-form_theme_lite">

    <input class="search-form__input">

    <button class="search-form__button
                   search-form__button_size_s
                   search-form__button_size_m">
        Search
    </button>
</form>
```



## 文件结构

1. 一个单一的block关联到一个单一的目录中
2. block和其关联的文件有着相同的名字，比如`header`块在`header/`文件中，`menu`块在`menu/`文件中
3. 一个块的功能是由几个文件来实现的，比如`header.js`和`header.css`
4. 一个块的目录是其elements和modifiers的根目录
5. element目录以`__`为前缀进行命名，比如`header/__logo/`和`menu/__item/`
6. modifier目录以`_`为前缀进行命名，比如`header_fixed/`和`menu/_theme_islands`
7. elements和modifiers的功能是由几个文件来实现的，比如`header__input.js`和`header_theme_islands.css`

```
search-form/                           # Directory of the search-form

    __input/                           # Subdirectory of the search-form__input
        search-form__input.css         # CSS implementation of the
                                       # search-form__input element
        search-form__input.js          # JavaScript implementation of the
                                       # search-form__input element

    __button/                          # Subdirectory of the search-form__button
                                       # element
        search-form__button.css
        search-form__button.js

    _theme/                            # Subdirectory of the search-form_theme
                                       # modifier
        search-form_theme_islands.css  # CSS implementation of the search-form block
                                       # that has the theme modifier with the value
                                       # islands
        search-form_theme_lite.css     # CSS implementation of the search-form block
                                       # that has the theme modifier with the value
                                       # lite

    search-form.css                    # CSS implementation of the search-form block
    search-form.js                     # JavaScript implementation of the
                                       # search-form block
```

