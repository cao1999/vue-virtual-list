<template>
  <div class="virtual-list-demo">
    <!-- 视口容器 -->
    <div class="virtual-list" @scroll="handleScroll" ref="virtualListContainer">
      <!-- 撑起列表的真实高度容器 -->
      <div
        class="virtual-list-placeholder"
        :style="{ height: `${props.itemSize * props.list.length}px` }"
      ></div>
      <!-- 渲染的列表项 -->
      <div
        class="virtual-list-item-wrapper"
        :style="{ transform: `translate3d(0,${offsetY}px,0)` }"
      >
        <div class="virtual-list-item" v-for="item in virtualList" :key="item">
          {{ item }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch, withDefaults, defineProps } from "vue";

interface ListProps {
  list: Array<any>;
  // 列表项的高度
  itemSize?: number;
}

const props = withDefaults(defineProps<ListProps>(), {
  list: [],
  itemSize: 50,
});

// 虚拟列表
const virtualList = ref<any[]>([]);
// 虚拟列表的容器
const virtualListContainer = ref(null);
// 可视区域展示多少个列表项
const virtualListLength = ref(0);
// 虚拟列表的偏移量
const offsetY = ref(0);

// 计算当前视口可以显示列表项的个数
const calcVirtualListLength = () => {
  // 必须向上取整，保证列表项的个数比可视区域最多展示的个数要多\
  const containerHeight = virtualListContainer.value.clientHeight;
  let visibleCount = Math.ceil(containerHeight / props.itemSize) + 1;

  virtualListLength.value = visibleCount;
};

watch(
  () => props.list,
  () => {
    calcVirtualListLength();
  }
);

onMounted(() => {
  calcVirtualListLength();

  // 首次渲染，取列表的前n个列表项
  virtualList.value = props.list.slice(0, virtualListLength.value);
});

// 监听滚动事件，动态修改virtualList的内容
const handleScroll = () => {
  // 1. 获取scrollTop的值，计算startIndex
  const scrollTop = virtualListContainer.value.scrollTop;
  const startIndex = Math.floor(scrollTop / props.itemSize);
  const endIndex = startIndex + virtualListLength.value;

  offsetY.value = Math.floor(scrollTop / props.itemSize) * props.itemSize;

  // 更新虚拟列表内容
  virtualList.value = props.list.slice(startIndex, endIndex);
};
</script>

<style lang="less">
* {
  box-sizing: border-box;
}

.virtual-list {
  overflow: scroll;
  position: relative;
  max-height: 400px;
  border: 1px solid #ccc;

  &-demo {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate3d(-50%, -50%, 0);
    overflow: hidden;
    width: 200px;
  }

  &-item {
    height: 50px;
    text-align: center;
    line-height: 50px;

    &-wrapper {
      /* 注意：这里是fixed定位 */
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      pointer-events: none;
    }
  }
}
</style>
