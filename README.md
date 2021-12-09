# 虚拟列表

## 使用场景

在日常开发中，当服务端返回一个列表时，如果列表长度比较短的话可以直接将列表渲染到页面当中，但是对于`长列表`来说，将其渲染到页面中将会花费很长时间，以下面这段代码为例。

```vue
<template>
  <ul class="list">
    <li class="list-item" v-for="item in data.list" :key="item">{{ item }}</li>
  </ul>
</template>

<script setup lang="ts">
import { reactive, onMounted } from "vue";

const data = reactive({
  list: [],
});

onMounted(() => {
  const tmp = [];
  const startTime = Date.now();

  for (let i = 0; i < 10000; i++) {
    tmp.push(i);
  }

  data.list = tmp;

  // js执行时间0.001s
  console.log("js执行时间", (Date.now() - startTime) / 1000);

  setTimeout(() => {
    // 总执行时间1.588s
    console.log("js执行时间 + 渲染时间", (Date.now() - startTime) / 1000);
  }, 0);
});
</script>
```

在以上代码中，我们将 10000 条数据一次性渲染到了页面当中，总用时为**1.588s**，其中`渲染时间`占了**1.587s**。可以得出在渲染列表时，大部分时间都花在了`页面渲染`阶段。究其原因是因为渲染的`dom节点`太多，因此可以通过减少渲染`dom节点`数量来降低`页面渲染`阶段的用时。`虚拟列表`正是解决这个问题的有效方法。

## 什么是虚拟列表

`虚拟列表`是指仅仅渲染`可视区域`内的列表项，超出的不渲染，这样的话每次渲染的节点数将极大的减少，相应的`渲染时间`也会大幅降低。

`虚拟列表`的渲染主要分为两部分。首先是`首次渲染`，直接取列表的前`n`条数据即可。其次是`滚动渲染`，监听`列表容器`的滚动事件，并实时计算当前需要渲染的`子列表[startIndex, endIndex]`。

## 实现

首先实现`虚拟列表`的`Dom结构`。结构如下所示

```vue
<template>
  <!-- 虚拟列表容器 -->
  <div class="virtual-list" ref="virtualListContainer">
    <!-- 占位dom，将其高度设置为itemHeight * list.length，用来撑起容器盒子实现滚动效果 -->
    <div
      class="virtual-list-placeholder"
      :style="{ height: `${itemHeight * list.length}` }"
    ></div>
    <!-- 列表项盒子 -->
    <div
      class="virtual-list-box"
      :style="{ transform: `translate3d(0,${offsetY}px, 0)` }"
    >
      <!-- 列表项 -->
      <div class="virtual-list-box-item"></div>
    </div>
  </div>
</template>

<style lang="less">
* {
  box-sizing: border-box;
}

.virtual-list {
  overflow: auto;
  position: relative;
  max-height: 400px;
  border: 1px solid #ccc;

  &-box {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    /* 需要取消鼠标事件，否则无法进行滚动 */
    pointer-events: none;

    &-item {
      height: 50px;
      text-align: center;
      line-height: 50px;
    }
  }
}
</style>
```

**注意**

1. `virtual-list-box列表盒子`是相对`virtual-list容器盒子`进行定位的，当`容器盒子`滚动时`列表盒子`也会跟着滚动，因此需要给`列表盒子`设置`transform: translate3d(0,offsetY, 0)`属性将其移动到视口区域。

计算前进行以下约定

1. 容器的可视高度为`visibleHeight`
2. 列表项的高度为`itemHeight`
3. 容器可视区域内列表项的个数为`virtualListLength`
4. 容器滚动的距离为`scrollTop`

需要计算的值包括:

1. **容器滚动的距离**：`scrollTop = virtualListContainer.scrollTop`
2. **计算列表开始位置**：`startIndex = Math.floor(scrollTop / itemHeight)`
3. **计算列表结束位置**:`endIndex = startIndex + virtualListLength`
4. **计算列表盒子的偏移量(translateY 的值)**：`offsetY = Math.floor(scrollTop / itemHeight) * itemHeight`

[点我查看完整代码](https://git.n.xiaomi.com/caoshaoshuai/virtual-list)

## 参考

- [「前端进阶」高性能渲染十万条数据(虚拟列表)](https://juejin.cn/post/6844903982742110216)
