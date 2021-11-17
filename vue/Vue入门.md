# Vue入门

## 概述：

MVVM模式的实现者

1. `Model:模型层，这里标识JavaScript对象`
2. `View:视图层，在这里表示DOM（HTML操作的元素）`
3. `ViewModel:连接视图和数据的中间件，Vue.js就是MVVM中的ViewModel层的实现者`

## 快速开始

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
<!--    导入vue-->
    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.min.js"></script>
</head>
<body>
hello
<div id="app">
    {{message}}
</div>

<script>
    var vm = new Vue({
        el: "#app",
        data:{
            message:"hello vue!"
        }
    });
</script>
</body>
</html>
```

我们发现只要 vm.message发生改变，即使不刷新页面，我们的视图也会发生变化。

