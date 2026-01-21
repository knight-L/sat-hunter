<template>
  <div class="relative w-full" ref="containerRef">
    <div
      class="relative flex items-center w-full bg-white border rounded cursor-text transition-all duration-200"
      :class="[
        isOpen
          ? 'border-blue-500 ring-2 ring-blue-100'
          : 'border-gray-300 hover:border-blue-400',
        disabled ? 'bg-gray-100 cursor-not-allowed' : '',
      ]"
      @click="triggerFocus">
      <input
        ref="inputRef"
        type="text"
        :value="displayValue"
        :placeholder="selectedLabel ? '' : placeholder"
        :disabled="disabled"
        class="w-full px-3 py-2 bg-transparent outline-none text-sm text-gray-700 placeholder-gray-400"
        @input="handleInput"
        @focus="handleFocus"
        @blur="handleBlur" />

      <span
        class="absolute right-3 text-gray-400 pointer-events-none transition-transform duration-200"
        :class="{ 'rotate-180': isOpen }">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="12"
          height="12"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round">
          <polyline points="6 9 12 15 18 9"></polyline>
        </svg>
      </span>
    </div>

    <div
      v-show="isOpen"
      class="absolute z-50 w-full mt-1 bg-white border border-gray-200 rounded shadow-lg overflow-hidden">
      <div
        v-if="filteredOptions.length === 0"
        class="py-4 text-center text-gray-400 text-sm">
        暂无数据
      </div>

      <div
        v-else
        ref="dropdownRef"
        class="overflow-y-auto custom-scrollbar"
        :style="{ height: `${dropdownHeight}px` }"
        @scroll="handleScroll"
        @mousedown.prevent>
        <div :style="{ height: `${totalHeight}px`, position: 'relative' }">
          <div :style="{ transform: `translateY(${offsetY}px)` }">
            <div
              v-for="item in visibleOptions"
              :key="item.adcode"
              class="px-3 flex items-center text-sm cursor-pointer transition-colors duration-150 truncate"
              :style="{ height: `${itemHeight}px` }"
              :class="[
                modelValue === item.adcode
                  ? 'bg-blue-50 font-medium text-blue-600'
                  : 'text-gray-700 hover:bg-gray-100',
              ]"
              @click="handleSelect(item)">
              {{ item.name }}
              <span class="ml-2 text-[10px] text-gray-400 uppercase">
                {{ item.level }}
              </span>
              <span
                v-if="modelValue === item.adcode"
                class="ml-auto text-blue-600">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="16"
                  height="16"
                  viewBox="0 0 24 24"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round">
                  <polyline points="20 6 9 17 4 12"></polyline>
                </svg>
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { ref, computed, watch, onMounted, onUnmounted, nextTick } from "vue";

  interface Option {
    name: string;
    adcode: string | number;
    level?: string;
  }

  interface Props {
    modelValue?: string | number | null;
    options?: Option[];
    placeholder?: string;
    itemHeight?: number;
    dropdownHeight?: number;
    disabled?: boolean;
  }

  const props = withDefaults(defineProps<Props>(), {
    options: () => [],
    placeholder: "请选择",
    itemHeight: 34,
    dropdownHeight: 256,
    disabled: false,
  });

  const emits = defineEmits(["update:modelValue"]);

  const isOpen = ref(false);
  const inputRef = ref<HTMLInputElement | null>(null);
  const containerRef = ref<HTMLDivElement | null>(null);
  const dropdownRef = ref<HTMLDivElement | null>(null);

  const searchQuery = ref(""); // 搜索关键字
  const scrollTop = ref(0); // 滚动条位置

  // 获取当前选中的 label
  const selectedLabel = computed(() => {
    const target = props.options.find((op) => op.adcode === props.modelValue);
    return target ? target.name : "";
  });

  // 输入框显示的文字
  // 1. 如果打开且正在输入，显示用户的输入值
  // 2. 如果关闭，显示选中的 label
  const displayValue = computed(() => {
    if (isOpen.value) {
      return searchQuery.value;
    }
    return selectedLabel.value;
  });

  // 过滤后的列表
  const filteredOptions = computed(() => {
    if (!searchQuery.value) return props.options;
    const query = searchQuery.value.toLowerCase();
    return props.options.filter((op) =>
      String(op.name).toLowerCase().includes(query)
    );
  });

  // 1. 列表总高度 (撑开滚动条)
  const totalHeight = computed(
    () => filteredOptions.value.length * props.itemHeight
  );

  // 2. 可视区域能显示的条数 (向上取整 + 缓冲区)
  const visibleCount = computed(
    () => Math.ceil(props.dropdownHeight / props.itemHeight) + 4
  );

  // 3. 计算可视区域的开始索引
  const startIndex = computed(() =>
    Math.floor(scrollTop.value / props.itemHeight)
  );

  // 4. 计算可视区域的结束索引
  const endIndex = computed(() =>
    Math.min(
      startIndex.value + visibleCount.value,
      filteredOptions.value.length
    )
  );

  // 5. 当前实际渲染的数据切片
  const visibleOptions = computed(() => {
    return filteredOptions.value.slice(startIndex.value, endIndex.value);
  });

  // 6. 偏移量 (让可视区域始终在视口内)
  const offsetY = computed(() => startIndex.value * props.itemHeight);

  // 滚动事件处理
  const handleScroll = (e: Event) => {
    const target = e.target as HTMLElement;
    scrollTop.value = target.scrollTop;
  };

  const triggerFocus = () => {
    if (props.disabled) return;
    inputRef.value?.focus();
  };

  const handleFocus = () => {
    isOpen.value = true;
    // 这里为了搜索体验，聚焦时如果之前有值，我们把 searchQuery 设为当前 label，方便用户修改，也可以设为空
    searchQuery.value = "";

    // 重置滚动条
    nextTick(() => {
      scrollTop.value = 0;
      if (dropdownRef.value) dropdownRef.value.scrollTop = 0;
    });
  };

  const handleInput = (e: Event) => {
    searchQuery.value = (e.target as HTMLInputElement).value;
    // 搜索时重置滚动条到顶部
    scrollTop.value = 0;
    if (dropdownRef.value) dropdownRef.value.scrollTop = 0;
  };

  const handleSelect = (item: Option) => {
    emits("update:modelValue", item.adcode);
    isOpen.value = false;
    searchQuery.value = "";
    inputRef.value?.blur();
  };

  const handleBlur = () => {
    // 使用 setTimeout 稍微延迟，以便处理点击下拉项的事件
    // 因为 blur 优先于 click 触发
    setTimeout(() => {
      isOpen.value = false;
      searchQuery.value = "";
    }, 150);
  };
</script>

<style scoped>
  .custom-scrollbar::-webkit-scrollbar {
    width: 6px;
  }

  .custom-scrollbar::-webkit-scrollbar-track {
    background: transparent;
  }

  .custom-scrollbar::-webkit-scrollbar-thumb {
    background-color: #d1d5db;
    border-radius: 3px;
  }

  .custom-scrollbar::-webkit-scrollbar-thumb:hover {
    background-color: #9ca3af;
  }
</style>
