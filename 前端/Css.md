# Css

## 导入方式

- ### 行内样式

  ```html
  <h1 style="color: pink">hello</h1>
  ```

- ### 内部样式

  ```html
  <style>
      h1 {
          color: aqua;
      }
  </style>
  ```

- ### 外部样式

  ```html
  <link rel="stylesheet" href="../css/style.css">
  ```

## 选择器

### 基本选择器

- #### 标签选择器

  ```css
  标签名{
      color: lightcoral;
  }
  ```

  

- #### 类选择器

  ```css
  .类名{
      color: lightcoral;
  }
  ```

  

- #### id选择器

  ```css
  #id{
      color: lightcoral;
  }
  ```

### 层次选择器

```html
<p id="p1">p1</p>
<p>p2</p>
<p>p3</p>

<ul>
    <li>
        <p>p4</p>
    </li>
    <li>
        <p>p5</p>
    </li>
    <li>
        <p>p6</p>
    </li>
</ul>
```

- ##### 后代选择器

  ```css
  body p{
      color: lightcoral;
  }
  ```

- ##### 子选择器

  ```css
  body>p{
      color: lightcoral;
  }
  ```

- ##### 相邻兄弟选择器（向下）

  ```css
  #p1 + p{
      color: lightcoral;
  }
  ```

- ##### 通用选择器（向下的所有元素）

  ```css
  #p1 ~ p{
      color: lightcoral;
  }
  ```

### 结构伪类选择器

```css
ul的最后一个li元素
ul li:last-child{
    color: lightcoral;
}
```

### 属性选择器

```css
p[id = p1]{
    background: lightcoral;
}
```

