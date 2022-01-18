---
title: vue3 jsx 入门
---

# what is jsx

```jsx
const element = <div>Hello world!</div>;
```

这就是 jsx  
jsx:Javascript & XML，是一个 js 语法的拓展，就是在 js 中书写 xml 以声明 UI 界面，jsx 乍一看很像是类似于 ejs 之类的模板语言，但是它具有完整的 js 能力，可以说它本身就是在写 js，只不过把 UI 声明的能力整合到了 js 中。  
JSX 最早是由 facebook 起草的一个规范，后面的这个 X 可以理解为它是 JavaScript 的语法扩展，感兴趣的同学可以从这个链接进去看看里面的具体内容。由于各个前端框架的实现不一样，所以它不会由引擎或浏览器实现，需要 Transform 之后转成常规的 JS 之后，才能在浏览器里面运行。JSX 其实也和模板语言类似，但它具有 JavaScript 的全部功能，但是由于在模板中的一些限制，用模板写出来的代码性能要比 JSX 好得多。

<!-- Benchmark: 340.9560546875 ms -->
<!-- BenchmarkSFC: 195.54296875 ms -->

# why jsx

## 一个文件写多个组件

一个 `.vue` 文件里面只能写一个组件，这个说实话在一些场景下还是不太方便，很多时候我们写一个页面的时候其实可能会需要把一些小的节点片段拆分到小组件里面进行复用，这些小组件其实写个简单的函数组件就能搞定了。比如我们需要封装一个 Input 组件，针对不同的 type，我们希望可以直接导出不同的组件，这时如果使用 sfc，你需要书写多个组件，而在 jsx 中，你可以直接导出多个组件，如下

```jsx
const Input = (props) => <input {...props} />;
export const Textarea = (props) => <Input type="textarea" {...props} />;
export const Password = (props) => <Input type="password" {...props} />;
export default Input;
```

## 有完全的 js 编程能力

我们可以利用 js 的各种 api 对整个 UI 进行操作，如下

```jsx
const ReverseList = ({ needReverse }) => {
  const domArr = [<div>DomA</div>, <div>DomB</div>];
  if (needReverse) domArr.reverse();
  return <div>{domArr}</div>;
};
```

## 性能对比

找到下图

![性能对比](./jsximg/713F690C-008E-4DEF-90DA-D322AA6DA3C7.png)

左右两个 demo 里面，整了两万个节点，奇数节点里面 class 是动态的，偶数节点的 textContent 是动态的，点击 shuffle。在这个例子里面，用模板写的代码 比用 JSX 写的要快十几毫秒。

# how to use

vue3&vue-cli 中，由于 cli 直接包括了`@vue/babel-plugin-jsx`这个包，所以可以直接使用 jsx 。  
语法如下

```jsx
import { defineComponent } from "vue";

const HelloWorld = defineComponent({
  name: "hello-world",
  setup() {
    return {};
  },
  render() {
    return <div>Hello world!</div>;
  },
});

export default HelloWorld;
```

vue3&vite 中，需要在 vite.config.js 中引入

```jsx
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import vueJsx from "@vitejs/plugin-vue-jsx";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), vueJsx()],
});
```

引入后，即可使用 jsx 书写组件。

## 常见语法糖对应写法

v-if

```html
<div v-if="show">content</div>
```

```jsx
<>{show && <div>content</div>}</>
```

v-if & v-else

```html
<div v-if="show">content A</div>
<div v-else>content B</div>
```

```jsx
<>{show ? <div>content A</div> : <div>content B</div>}</>
```

v-for
{% raw %}

```html
<div v-for="item in arr" :key="item.id">{{item.content}}</div>
```

{% endraw %}

```jsx
{
  arr.map((item) => <div key={item.id}>{item.content}</div>);
}
```

v-show

```html
<div v-show="show">content</div>
```

{% raw %}

```jsx
<div style={{ display: show ? "block" : "none" }}></div>
```

{% endraw %}

slot

声明

```html
<template>
  <div>
    <slot name="pre"></slot>
    <div>main</div>
    <slot></slot>
    <slot name="next"></slot>
  </div>
</template>

<script>
  export default {
    setup() {},
  };
</script>
```

```jsx
import { defineComponent } from "vue";

const child = defineComponent({
  setup(props, { slots }) {
    return {};
  },
  render() {
    return (
      <>
        {this.$slots.pre?.()}
        <div>childMain</div>
        {this.$slots.default?.()}
        {this.$slots.next?.()}
      </>
    );
  },
});

export default child;
```

v-slot
使用

```html
<template>
  <div>
    <child>
      <template v-slot:pre><div>before</div></template>
      <div>default slot</div>
      <template v-slot:next><div>after</div></template>
    </child>
  </div>
</template>

<script>
  import child from "./child";
  export default {
    components: { child },
    setup() {},
  };
</script>
```

{% raw %}

```jsx
import { defineComponent } from "vue";
import Child from "./child";

const TestSlotJsx = defineComponent({
  name: "test-slot-jsx",
  setup() {
    return {};
  },
  render() {
    return (
      <div>
        <Child
          v-slots={{
            pre: () => <div>childPre</div>,
            next: () => <div>childNext</div>,
          }}
        >
          default slot
        </Child>
      </div>
    );
  },
});

export default TestSlotJsx;
```

{% endrow %}
