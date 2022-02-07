# 笔记

## 确认页面url

### 页面分类

+ money 记账/默认页面
+ labels 标签
+ statistics 统计
+ 404	

初始化组件： Money. Vue/ Labels.vue/ Statistics.ue
将 router 传给 new Vue（）

router/ index.ts添加 router,配置4个路径对应组件

在App组件里用< router-view/>给出 router渲染区域

新增导航栏Nav组件，需要在每个组件中添加Nav，为了方便在main.ts中全局引入

``` vue
Vue.component("Nav", Nav);
```

### 404页面

Vue-router中写道，常规参数只会匹配被 / 分隔的 URL 片段中的字符。如果想匹任意路径，可以使用通配符 (*)：

```js
 {
    //匹配所有不存在的页面
    path: "/*",
    component: NotFound,
  }
```

### 布局Layout组件

需要在每个页面设置同样的页面布局，可以抽离成Layout组件，再利用插槽在其它页面组件中使用

```vue
 <div class="layout-wrapper">
    <div class="content">
      <slot />
    </div>
 </div>
<script lang="ts">
export default {
  name: "Layout"
};
</script>
<style lang="scss" scoped>
.layout-wrapper {
  display: flex;
  flex-direction: column;
  height: 100vh;
}
.content {
  overflow: auto;
  flex: 1;
}
</style>
```

### meta viewport

```html
<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
```

## 使用svg-sprite-loader引入icon

svg格式文件不能直接引用，使用svg-sprite-loader

### 安装

```
npm install svg-sprite-loader -D
# via yarn
yarn add svg-sprite-loader -D
```

### 在vue.config.js中添加配置

```js
module.exports = {
  chainWebpack: config => {
    const dir = path.resolve(__dirname, 'src/assets/icons')
    config.module
      .rule('svg-sprite')
      .test(/\.svg$/)
      .include.add(dir).end() //只包含icons目录
      .use('svg-sprite-loader').loader('svg-sprite-loader').options({ extract: false }).end()
      .use('svgo-loader').loader('svgo-loader')
      .tap(options => ({ ...options, plugins: [{ removeAttrs: { attrs: 'fill' } }] })).end()
    config.plugin('svg-sprite').use(require('svg-sprite-loader/plugin'), [{ plainSprite: true }])
    config.module.rule('svg').exclude.add(dir) //其它svg loader排除icons目录
  }
}
```

添加配置后，就可以引入并使用`<svg><use xlink:href="#name"></use></svg>`来展示svg图片

### 封装Icon组件

在添加icon时需要多次引入和写多个svg标签，可将Icon分装成全局组件，减少重复代码

```vue
<template>
  <svg class="icon" @click="$emit('click', $event)">
    <use :xlink:href="'#' + name" />
  </svg>
</template>

<script lang="ts">
let importAll = (requireContext: __WebpackModuleApi.RequireContext) =>
  requireContext.keys().forEach(requireContext);
try {
  importAll(require.context("../assets/icons", true, /\.svg$/));
} catch {
  console.log(Error);
}
export default {
  props: ["name"],
  name: "Icon",
};
</script>

<style lang="scss">
.icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

## 初始化导航栏Nav组件，添加样式

```vue
<template>
  <nav>
    <router-link to="/money" class="item" active-class="selected"
      ><Icon name="money"
    /></router-link>
    <router-link to="/labels" class="item" active-class="selected"
      ><Icon name="label"
    /></router-link>
    <router-link to="/statistics" class="item" active-class="selected"
      ><Icon name="statistics"
    /></router-link>
  </nav>
</template>

<script lang="ts">
export default {};
</script>

<style lang="scss" scoped>
nav {
  display: flex;
  flex-direction: row;
  font-size: 12px;
  background: #3f3a3a;
  color: rgb(165, 160, 160);
  > .item {
    padding: 10px 0;
    width: 33.333333%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    .icon {
      margin: 7px;
      width: 32px;
      height: 32px;
    }
  }
  .item.selected {
    color: white;
  }
}
</style>
```

## Money页面

### 初始化Money页面

```vue
<template>
  <Layout>
    <p>money</p>
    <div class="tags">
      <ul class="current">
        <li>衣</li>
        <li>食</li>
        <li>住</li>
        <li>行</li>
      </ul>
      <div class="new">
        <button>新增</button>
      </div>
    </div>
    <label class="notes">
      <span>备注</span>
      <input id="xxx" type="text" />
    </label>
    <div>
      <ul class="types">
        <li class="selected">支出</li>
        <li>收入</li>
      </ul>
    </div>
    <div class="numberPad">
      <div class="output">number</div>
      <div class="buttons">
        <button>1</button>
        <button>2</button>
        <button>3</button>
        <button>删除</button>
        <button>4</button>
        <button>5</button>
        <button>6</button>
        <button>清空</button>
        <button>7</button>
        <button>8</button>
        <button>9</button>
        <button>ok</button>
        <button>0</button>
        <button>.</button>
      </div>
    </div>
  </Layout>
</template>
```

### 模块化Money页面，抽离组件

#### Money组件

```vue
<template>
  <Layout>
    <NumberPad />
    <Types />
    <Tags />
    <Notes />
  </Layout>
</template>

<script>
import NumberPad from "@/components/Money/NumberPad.vue";
import Tags from "@/components/Money/Tags.vue";
import Types from "@/components/Money/Types.vue";
import Notes from "@/components/Money/Notes.vue";
export default {
  components: { NumberPad, Tags, Notes, Types },
  name: "Money",
};
</script>
```

#### Notes模块

```vue
<template>
  <label class="notes">
    <span class="name">备注</span>
    <input id="xxx" type="text" placeholder="输入备注" />
  </label>
</template>

<script lang="ts">
export default {
  name: "Notes",
};
</script>

<style lang="scss" scoped>
.notes {
  font-size: 14px;
  background: #f5f5f5;
  display: flex;
  padding-left: 16px;
  align-items: center;
  .name {
    padding-right: 16px;
  }
  input {
    border: none;
    background: transparent;
    height: 64px;
    flex-grow: 1;
    padding-right: 16px;
  }
}
</style> 
```

#### Types模块

```vue
<template>
  <ul class="types">
    <li class="selected">支出</li>
    <li>收入</li>
  </ul>
</template>

<script lang="ts">
export default {};
</script>

<style lang="scss" scoped>
.types {
  display: flex;
  background: #c4c4c4;
  font-size: 24px;
  > li {
    width: 50%;
    height: 64px;
    line-height: 64px;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    &.selected {
      &::after {
        content: "";
        position: absolute;
        bottom: 0;
        left: 0;
        height: 4px;
        width: 100%;
        background: #333;
      }
    }
  }
}
</style>
```



#### NumberPad模块

```vue
<template>
  <div class="numberPad">
    <div class="output">number</div>
    <div class="buttons">
      <button>1</button>
      <button>2</button>
      <button>3</button>
      <button>删除</button>
      <button>4</button>
      <button>5</button>
      <button>6</button>
      <button>清空</button>
      <button>7</button>
      <button>8</button>
      <button>9</button>
      <button class="ok">ok</button>
      <button class="zero">0</button>
      <button>.</button>
    </div>
  </div>
</template>

<script lang="ts">
export default {};
</script>

<style lang="scss" scoped>
@import "~@/assets/style/helper.scss";
.numberPad {
  .output {
    @extend %clearfix;
    @extend %innerShadow;
    font-size: 36px;
    font-family: Consolas, monospace;
    padding: 9px 16px;
    text-align: right;
  }
  .buttons {
    &::after {
      @extend %clearfix;
    }
    > button {
      width: 25%;
      height: 64px;
      float: left;
      background: transparent;
      border: none;
      &.ok {
        height: 64px * 2;
        float: right;
      }
      &.zero {
        width: 50%;
      }
      $bg: #f2f2f2;
      &:nth-child(1) {
        background: $bg;
      }
      &:nth-child(2),
      &:nth-child(5) {
        background: darken($color: $bg, $amount: 4%);
      }
      &:nth-child(3),
      &:nth-child(6),
      &:nth-child(9) {
        background: darken($color: $bg, $amount: 8%);
      }
      &:nth-child(4),
      &:nth-child(7),
      &:nth-child(10) {
        background: darken($color: $bg, $amount: 12%);
      }
      &:nth-child(8),
      &:nth-child(11),
      &:nth-child(13) {
        background: darken($color: $bg, $amount: 16%);
      }
      &:nth-child(14) {
        background: darken($color: $bg, $amount: 20%);
      }
      &:nth-child(12) {
        background: darken($color: $bg, $amount: 24%);
      }
    }
  }
}
</style> 
```

#### Tags模块

```vue
<template>
  <div class="tags">
    <div class="new">
      <button>新增</button>
    </div>
    <ul class="current">
      <li>衣</li>
      <li>食</li>
      <li>住</li>
      <li>行</li>
    </ul>
  </div>
</template>

<script lang='ts'>
export default {};
</script>

<style lang="scss" scoped>
.tags {
  flex-grow: 1;
  font-size: 14px;
  padding: 16px;
  display: flex;
  flex-direction: column-reverse;
  > .current {
    display: flex;
    > li {
      background: rgb(192, 185, 185);
      height: 24px;
      line-height: 24px;
      border-radius: 12px;
      padding: 0 16px;
      margin-right: 16px;
    }
  }
  > .new {
    padding: 16px 0;
    button {
      background: transparent;
      border: none;
      color: #999;
      border-bottom: 1px solid;
      padding: 0 4px;
    }
  }
}
</style> 
```

### Types组件

#### JS组件

```vue
<template>
  <ul class="types">
    <li :class="type === '-' && 'selected'" @click="selectType('-')">支出</li>
    <li :class="type === '+' && 'selected'" @click="selectType('+')">收入</li>
  </ul>
</template>

<script>
export default {
  name: "Types",
  data() {
    return {
      type: "-", //'-'表示支出 '+'表示收入
    };
  },
  mounted() {
    console.log(this.xxx);
  },
  methods: {
    selectType(type) {
      //type只能是'-'或'+'
      if (type !== "-" && type !== "+") {
        throw new Error("type is unknown");
      }
    },
  },
};
</script>
...
```

#### TS组件

+ 有别于JS组件不使用构造选项,使用装饰器，从第三方库vue-property-decorator引入Component和Prop，[文档](https://github.com/kaorun343/vue-property-decorator)

```vue
<template>
  <ul class="types">
    <li :class="type === '-' && 'selected'" @click="selectType('-')">支出</li>
    <li :class="type === '+' && 'selected'" @click="selectType('+')">收入</li>
  </ul>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator"; 
export default class Types extends Vue {
  type = "-"; //data， "-"表示指出 "+表示收入"
  @Prop(Number) propA: number | undefined; //Prop装饰器
  selectType(type: string) {
    //methods
    if (type !== "-" && type !== "+") {
      throw new Error("type is unknown");
    }
    this.type = type;
  }
}
</script>
...
```

+ TS本质 `JS：类型` 检查JS代码

  + 类型提示：更智能的提示
  + 编译时报错：未运行代码就知道错误
  + 类型检查：无法点出错误的属性

+ 写Vue组件三种方式

  + ```vue
    //用JS对象
    export default { data,props,methods,created,... }
    ```

  + ```vue
    //用TS类 <script lang='ts'>
    export default class xxx extends Vue { 
    	xxx:string='hi'
    }
    ```

  + ```vue
    //用JS类
    export default class xxx extends Vue { 
    	xxx='hi'
    }
    ```

### NumberPad组件

```vue
<template>
  <div class="numberPad">
    <div class="output">{{ output || "0" }}</div>
    <div class="buttons">
      <button @click="inputContent">1</button>
      <button @click="inputContent">2</button>
      <button @click="inputContent">3</button>
      <button @click="remove">删除</button>
      <button @click="inputContent">4</button>
      <button @click="inputContent">5</button>
      <button @click="inputContent">6</button>
      <button @click="clear">清空</button>
      <button @click="inputContent">7</button>
      <button @click="inputContent">8</button>
      <button @click="inputContent">9</button>
      <button class="ok" @click="ok">ok</button>
      <button class="zero" @click="inputContent">0</button>
      <button @click="inputContent">.</button>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
@Component
export default class NumberPad extends Vue {
  output: string = "0";
  inputContent(event: MouseEvent) {
    //鼠标事件
    const button = event.target as HTMLButtonElement; //强制指定为Button
    const input = button.textContent!;
    if (this.output.length === 16) {
      return;
    }
    if (this.output === "0") {
      if ("0123456789".indexOf(input) >= 0) {
        this.output = input;
      } else {
        this.output += input;
      }
      return;
    }
    if (this.output.indexOf(".") >= 0 && input === ".") {
      //控制只有一个'.'
      return;
    }
    this.output += input;
  }
  remove() {
    this.output = this.output.slice(0, -1);
  }
  clear() {}
  ok() {}
}
</script>
...
```

### Notes组件

```vue
<template>
  <div>
    {{ value }}
    <label class="notes">
      <span class="name">备注</span>
      <input v-model="value" type="text" placeholder="输入备注" />
    </label>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
@Component
export default class Notes extends Vue {
  value = "";
}
</script>
...
```

### Tags组件

```vue
<template>
  <div class="tags">
    <div class="new">
      <button @click="create">新增</button>
    </div>
    <ul class="current">
      <li 
        v-for="tag in dataSource"
        :key="tag"
        :class="{ selected: selectedTags.indexOf(tag) >= 0 }"
        @click="toggle(tag)"
      >
        {{ tag }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
@Component
export default class Tags extends Vue {
  @Prop(Array) dataSource: string[] | undefined;
  selectedTags: string[] = [];
   //标签的开关控制
  toggle(tag: string) {
    const index = this.selectedTags.indexOf(tag);
    if (index >= 0) {
      this.selectedTags.splice(index, 1);
    } else {
      this.selectedTags.push(tag);
    }
  }
  create() {
    const name = window.prompt("输入标签名");
    if (name === "") {
      window.alert("标签名不能为空");
    } else if (this.dataSource) { //如果填了一个name不为空，就把更新dataSource的请求告诉外部
      this.$emit("update:dataSource", [...this.dataSource, name]);
    }
  }
}
</script>
...


```

+ Money组件中使用.sync修饰符接受更新dataSource的请求

```vue
<template>
  <Layout class-prefix="layout">
    <NumberPad />
    <Types />
    <Notes />
    <Tags :data-source.sync="tags" />
  </Layout>
</template>

<script>
...
export default {
  components: { NumberPad, Tags, Notes, Types },
  name: "Money",
  data() {
      tags: ["衣", "食", "住", "行"],
    };
  },
};
</script>
```



### 收集四个组件数据

#### Tags

```vue
...
<script lang='ts'>
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
@Component
export default class Tags extends Vue {
  @Prop(Array) dataSource: string[] | undefined;
  selectedTags: string[] = [];
  toggle(tag: string) {
    const index = this.selectedTags.indexOf(tag);
    if (index >= 0) {
      this.selectedTags.splice(index, 1);
    } else {
      this.selectedTags.push(tag);
    }
      //将‘update:value’事件传出去
    this.$emit("update:value", this.selectedTags);
  }
  create() {
    const name = window.prompt("输入标签名");
    if (name === "") {
      window.alert("标签名不能为空");
    } else if (this.dataSource) {
      this.$emit("update:value", [...this.dataSource, name]);
    }
  }
}
</script>
```

+ Money组件中

```vue
<script lang="ts">...type Record = {  tags: string[];  notes: string;  type: string;  amount: number;};@Component({ components: { NumberPad, Tags, Notes, Types } })export default class Money extends Vue {  tags = ["衣", "食", "住", "行"];  record: Record = { tags: [], notes: "", type: "-", amount: 0 };  onUpdateTags(value: string[]) {    this.record.tags = value;  }  onUpdateNotes(value: string) {    this.record.notes = value;  }  onUpdateAmount(value: string) {    this.record.amount = parseFloat(value);  }}</script>
```



#### Notes

```vue
...<script lang="ts">import Vue from "vue";import { Component, Watch } from "vue-property-decorator";@Componentexport default class Notes extends Vue {  value = "";    //watch监听value  @Watch('value')  onValueChange(value:string){    this.$emit('update:value',value)  }}</script>
```

#### Types

```vue
<template>  <ul class="types">    <li :class="value === '-' && 'selected'" @click="selectType('-')">支出</li>    <li :class="value === '+' && 'selected'" @click="selectType('+')">收入</li>  </ul></template><script lang="ts">import Vue from "vue";import { Component, Prop, Watch } from "vue-property-decorator";@Componentexport default class Types extends Vue {  @Prop() readonly value!: string;  selectType(type: string) {    if (type !== "-" && type !== "+") {      throw new Error("type is unknown");    }    this.$emit("update:value", type);  }}</script>
```

#### NumberPads

```vue
<template>  <div class="numberPad">    <div class="output">{{ output || "0" }}</div>        ...    <script lang="ts">import Vue from "vue";import { Component } from "vue-property-decorator";	@@ -29,28 +29,32 @@  output: string = "0";  inputContent(event: MouseEvent) {    const button = event.target as HTMLButtonElement;    const input = button.textContent!;    if (this.output.length === 16) {      return;    }    if (this.output === "0") {      if ("0123456789".indexOf(input) >= 0) {        this.output = input;      } else {        this.output += input;      }      return;    }    if (this.output.indexOf(".") >= 0 && input === ".") {      return;    }    this.output += input;  }  remove() {    this.output = this.output.slice(0, -1);  }  clear() {    this.output = "0";  }  ok() {    this.$emit("update:value", this.output);  }}</script>
```

#### Money组件

```vue
<template>  <Layout class-prefix="layout">    <NumberPad @update:value="onUpdateAmount" />    <Types :value.sync="record.type" />    <Notes @update:value="onUpdateNotes" />    <Tags :data-source.sync="tags" @update:value="onUpdateTags" />  </Layout></template><script lang="ts">...//类型声明type Record = {  tags: string[];  notes: string;  type: string;  amount: number;  createAt?： Date;};@Component({ components: { NumberPad, Tags, Notes, Types } })export default class Money extends Vue {  tags = ["衣", "食", "住", "行"];    //record赋予初始值  record: Record = { tags: [], notes: "", type: "-", amount: 0 };  onUpdateTags(value: string[]) {    this.record.tags = value;  }  onUpdateNotes(value: string) {    this.record.notes = value;  }  onUpdateAmount(value: string) {    this.record.amount = parseFloat(value);  }}</script>
```

### 将record保存到localstorage

```vue
//money组件	...<script lang="ts">	...recordList: Record[] = JSON.parse( //list初始化时从localStorage读取    window.localStorage.getItem("recordList") || "[]");	...saveRecord(amount) {  this.recordList.push(this.record)}@Watch('recordList')onRecordListChange() {  window.localStorage.setItem('recordList',JSON.stringify(this.recordList))}</script>
```

先输入1，第一次得到record `{a:1}`，使用`list.push(record)`

在输入2，`record.amout=2` ，push的是一个引用，会将记录覆盖

所以得先深拷贝record

```js
saveRecord() {
    const deepCloneRecord = JSON.parse(JSON.stringify(this.record));
    this.recordList.push(deepCloneRecord);//deepCloneRecord会拥有record一样的属性，但不是一样的对象
  }
  @Watch("recordList")
  onRecordListChange() {
    window.localStorage.setItem("recordList", JSON.stringify(this.recordList));
    console.log(this.recordList);
  }
```

###  抽离操纵数据的模块model

类型声明抽离到`custom.d.ts`TS全局声明文件

```vue
type Record = {
  tags: string[];
  notes: string;
  type: string;
  amount: number;
  createAt?： Date;
};
```

+ 新建models目录，其中添加`recordListModel.ts`

```tsx
const localStorageKeyName = "recordList";
const recordListModel = {
  clone(data: RecordItem[] | RecordItem) {
    return JSON.parse(JSON.stringify(data));
  },
  fetch() {
    return JSON.parse(
      window.localStorage.getItem("localStorageKeyName ") || "[]"
    ) as RecordItem[];
  },
  save(data: RecordItem[]) {
    window.localStorage.setItem(localStorageKeyName, JSON.stringify(data));
  },
};

export default recordListModel;
```

### 展示和创建标签

+ 把标签也存于models目录中，tagListModel.ts

```typescript
const localStorageKeyName = "tagList";
type Tag = {
  id: string;
  name: string;
};
type TagListModel = {
  data: Tag[];
  fetch: () => Tag[];
  create: (name: string) => "success" | "duplicated"; //'success'表示成功 'duplicated'表示重复
  save: () => void;
};
const tagListModel: TagListModel = {
  data: [],
  fetch() {
    this.data = JSON.parse(
      window.localStorage.getItem(localStorageKeyName) || "[]"
    );
    return this.data;
  },
  save() {
    window.localStorage.setItem(localStorageKeyName, JSON.stringify(this.data));
  },
  create(name: string) {
  const names = this.data.map((item) => item.name);
    if (names.indexOf(name) >= 0) {
      return "duplicated";
    }
    this.data.push({ id: name, name: name });
    this.save();
    return "success";
  },
};

export default tagListModel;
```

+ Lables.vue

```vue
<template>
  <Layout>
    <ol class="tags">
      <li v-for="tag in tags" :key="tag.id">
        <span>{{ tag.name }}</span> <Icon name="right" />
      </li>
    </ol>
    <div class="createTag-wrapper">
      <button class="createTag" @click="createTag">新建标签</button>
    </div>
  </Layout>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
import tagListModel from "@/models/tagListModel";
tagListModel.fetch();
@Component
export default class Labels extends Vue {
  tags = tagListModel.data;
  createTag() {
    const name = window.prompt("请输入标签");
    if (name) {
      const message = tagListModel.create(name);
      if (message === "duplicated") {
        window.alert("标签重复");
      } else if (message === "success") {
        window.alert("添加成功");
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.tags {
  background: white;
  font-size: 16px;
  padding-left: 16px;
  > li {
    min-height: 44px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    border-bottom: 1px solid #e6e6e6;
    svg {
      width: 20px;
      height: 20px;
      margin-right: 16px;
    }
  }
}
.createTag {
  background: #767676;
  color: white;
  border-radius: 4px;
  border: none;
  height: 40px;
  padding: 0 16px;
  &-wrapper {
    padding: 16px;
    text-align: center;
    margin-top: 28px;
  }
}
</style>
```

+ Money.vue

```vue
...
<script lang="ts">
...
import recordListModel from "@/models/recordListModel";
import tagListModel from "@/models/tagListModel";
const recordList = recordListModel.fetch();
const tagList = tagListModel.fetch();
@Component({ components: { NumberPad, Tags, Notes, Types } })
export default class Money extends Vue {
  tags = tagList;
  recordList: RecordItem[] = JSON.parse(
    window.localStorage.getItem("recordList") || "[]"
  );
    ...
const deepCloneRecord: RecordItem = recordListModel.clone(this.record);
    deepCloneRecord.createdAt = new Date();
    this.recordList.push(deepCloneRecord);
  }
  @Watch("recordList")
  onRecordListChange() {
    recordListModel.save(this.recordList);
  }
}
</script>
```

#### 编辑标签

+ EditLabel.vue

```vue
<template>
  <Layout>标签</Layout>
</template>

<script lang="ts">
import tagListModel from "@/models/tagListModel";
import Vue from "vue";
import { Component } from "vue-property-decorator";
@Component
export default class EditLabel extends Vue {
  created() {
    const id = this.$route.params.id;
    tagListModel.fetch();
    const tags = tagListModel.data;
    const tag = tags.filter((t) => t.id === id)[0];
    if (tag) {
      console.log(tag);
    } else {
      this.$router.replace("/404");
    }
  }
}
</script>
```

+ $route用来获取路由信息 $router是路由器

+ 在router中添加`EditLabel` 

```vue
{
   path: "/labels/edit/:id",
   component: EditLabel,
}
```

#### 标签页路由

+ Labels.vue

```vue
<template>
  <Layout>
    <div class="tags">
      <router-link class="tag" :to="`/labels/edit/${tag.id}`" v-for="tag in tags" :key="tag.id">
        <span>{{ tag.name }}</span> <Icon name="right" />
      </router-link>
    </div>
    ...
```

#### 改造Notes 使其成为通用组件

+ Notes.vue更名为FromItem.vue

```vue
<template>
  <div>
    {{ value }}
    <label class="notes">
      <span class="name">{{ this.fieldName }}</span>
      <input v-model="value" type="text" :placeholder="this.placeholder" />
    </label>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop, Watch } from "vue-property-decorator";
@Component
export default class Notes extends Vue {
  value = "";
  @Prop({ required: true }) fieldName!: string;
  @Prop() placeholder?: string;
  @Watch("value")
  onValueChange(value: string) {
    this.$emit("update:value", value);
  }
}
</script>
```

+ 编辑EditLabel.vue和Money.vue

```vue
<template>
  <Layout
    ><div><Icon class="left" name="right" /><span>编辑标签</span></div>
    <FromItem field-name="标签名" placeholder="请输入标签名"
  /></Layout>
</template>

<script lang="ts">
import tagListModel from "@/models/tagListModel";
import Vue from "vue";
import { Component } from "vue-property-decorator";
import FromItem from "../components/Money/FromItem.vue";
@Component({ components: { Notes } })
export default class EditLabel extends Vue {
  created() {
    const id = this.$route.params.id;
    tagListModel.fetch();
    const tags = tagListModel.data;
    const tag = tags.filter((t) => t.id === id)[0];
    if (tag) {
    } else {
      this.$router.replace("/404");
    }
  }
}
</script>

<style lang="scss" scoped>
.left {
  margin-top: 10px;
  transform: rotate(180deg);
  width: 20px;
  height: 20px;
  margin-right: 16px;
}
</style>
```

```vue
//Money.vue
...
 <Types :value.sync="record.type" />
    <FromItem
      @update:value="onUpdateNotes"
      fieldName="备注"
      placeholder="在此输入备注"
    />
    <Tags :data-source.sync="tags" @update:value="onUpdateTags" />
  </Layout>
</template>
```

#### 修复value被赋值的问题

```vue
...
<div>
    <label class="notes">
      <span class="name">{{ this.fieldName }}</span>
      <input
        //未直接赋值 而是通知外部要进行变更
        @input="onValueChange($event.target.value)"
        type="text"
        :placeholder="this.placeholder"
      />
    </label>
  </div>
</template>
...
@Prop({ default: "" }) readonly value!: string;
...
```

#### 完成编辑和删除标签

+ tagListModel.vue

```tsx
const localStorageKeyName = "tagList";
type Tag = {
  id: string;
  name: string;
};
type TagListModel = {
  data: Tag[];
  fetch: () => Tag[];
  create: (name: string) => "success" | "duplicated"; //'success'表示成功 'duplicated'表示重复
  save: () => void;
  update: (id: string, name: string) => "success" | "not-found" | "duplicated";
  remove: (id: string) => boolean;
};
const tagListModel: TagListModel = {
  data: [],
  fetch() {
    this.data = JSON.parse(
      window.localStorage.getItem(localStorageKeyName) || "[]"
    );
    return this.data;
  },
  update(id: string, name: string) {
    const idList = this.data.map((item) => item.id);
    if (idList.indexOf(id) >= 0) {
      const names = this.data.map((item) => item.name);
      if (names.indexOf(name) >= 0) {
        return "duplicated";
      } else {
        const tag = this.data.filter((item) => item.id == id)[0];
        tag.name = tag.id = name;
        this.save();
        return "success";
      }
    } else {
      return "not-found";
    }
  },
  save() {
    window.localStorage.setItem(localStorageKeyName, JSON.stringify(this.data));
  },
  remove(id: string) {
    let index = -1;
    for (let i = 0; i < this.data.length; i++) {
      if (this.data[i].id === id) {
        index = i;
        break;
      }
    }
    this.data.splice(index, 1);
    this.save();
    return true;
  },
  create(name: string) {
    const names = this.data.map((item) => item.name);
    if (names.indexOf(name) >= 0) {
```

+ EditLabels.vue

```vue
<template>
  <Layout
    ><div class="navBar">
      <Icon class="left" name="right" @click="goBack" /><span class="title"
        >编辑标签</span
      ><span class="right"></span>
    </div>
    <div class="notes-wrapper">
      <FromItem
        :value="tag.name"
        @update:value="update"
        field-name="标签名"
        placeholder="请输入标签名"
      />
    </div>
    <div class="button-wrapper">
      <Button @click="remove">删除标签</Button>
    </div>
  </Layout>
</template>
<script lang="ts">
import tagListModel from "@/models/tagListModel";
import Vue from "vue";
import { Component } from "vue-property-decorator";
import Notes from "../components/Money/Notes.vue";
import Button from "../components/Button.vue";
@Component({ components: { Notes, Button } })
export default class EditLabel extends Vue {
  tag?: { id: string; name: string } = undefined;
  created() {
    const id = this.$route.params.id;
    tagListModel.fetch();
    const tags = tagListModel.data;
    const tag = tags.filter((t) => t.id === id)[0];
    if (tag) {
      this.tag = tag;
    } else {
      this.$router.replace("/404");
    }
  }
  update(name: string) {
    if (this.tag) {
      tagListModel.update(this.tag.id, name);
    }
  }
  remove() {
    if (this.tag) {
      tagListModel.remove(this.tag.id);
    }
  }
  goBack() {
    this.$router.back();
  }
}
</script>
```

+ Icon组件添加click事件

```vue
<template>
  <svg class="icon" @click="$emit('click', $event)">
    <use :xlink:href="'#' + name" />
  </svg>
</template>
```

#### ID生成器

+ 会出现id复用的bug，创建一个id生成器

```typescript
let id: number = parseInt(window.localStorage.getItem("_idMax") || "0") || 0;
function createId() {
  id++;
  window.localStorage.setItem("_idMax", id.toString());
  return id;
}

export default createId;
```

+ tagListModel.ts中引入id生成器

```typescript
create(name: string) {
    const names = this.data.map((item) => item.name);
    if (names.indexOf(name) >= 0) {
      return "duplicated";
    }
    const id = createId().toString();
    this.data.push({ id, name: name });
    this.save();
    return "success";
  },
```

+ EditLabel.vue中删除成功时路由回退

```javascript
 ...
 remove() {
    if (this.tag) {
      if (tagListModel.remove(this.tag.id)) {
        this.$router.back();
      }
    }
  }
 ...
```

### 全局数据管理

数据在不同组件内调用时都产生一个新的对象，导致数据不同步

+ 重新封装recordListModel

```typescript
const localStorageKeyName = "recordList";
const recordListModel = {
  data: [] as RecordItem[],
  clone(data: RecordItem[] | RecordItem) {
    return JSON.parse(JSON.stringify(data));
  },
  fetch() {
    this.data = JSON.parse(
      window.localStorage.getItem("localStorageKeyName ") || "[]"
    ) as RecordItem[];
    return this.data;
  },
  save(data: RecordItem[]) {
    window.localStorage.setItem(localStorageKeyName, JSON.stringify(this.data));
  },
};

export default recordListModel;
```

+ 将create封装到recordListModel

```typescript
import clone from "@/lib/clone";
const localStorageKeyName = "recordList";
const recordListModel = {
  data: [] as RecordItem[],
  create(record: RecordItem) {
    const deepCloneRecord: RecordItem = clone(record);
    deepCloneRecord.createdAt = new Date();
    this.data.push(deepCloneRecord);
  },
    ...
```

+ 使用全局tagList

```typescript
//custom.d.ts
...
type Tag = {
  id: string;
  name: string;
};
type TagListModel = {
  data: Tag[];
  fetch: () => Tag[];
  create: (name: string) => "success" | "duplicated"; //'success'表示成功 'duplicated'表示重复
  save: () => void;
  update: (id: string, name: string) => "success" | "not-found" | "duplicated";
  remove: (id: string) => boolean;
};
interface Window {
  tagList: Tag[];
}
```

+ 在入口文件`main.ts`向window中保存tagList

```typescript
import Vue from "vue";
import App from "./App.vue";
import "./registerServiceWorker";
import router from "./router";
import store from "./store";
import Nav from "@/components/Nav.vue";
import Layout from "@/components/Layout.vue";
import Icon from "@/components/Icon.vue";
import tagListModel from "./models/tagListModel";

Vue.config.productionTip = false;
Vue.component("Nav", Nav);
Vue.component("Layout", Layout);
Vue.component("Icon", Icon);

window.tagList = tagListModel.fetch();

new Vue({
  router,
  store,
  render: (h) => h(App),
}).$mount("#app");
```

#### 封装taglist的增删改查

```typescript
//custom.d.ts
...
interface Window {
  tagList: Tag[];
  createTag: (name: string) => void;
  removeTag: (id: string) => boolean;
  updateTag: (
    id: string,
    name: string
  ) => "success" | "not-found" | "duplicated";
  findTag: (id: string) => Tag;
}
//main.ts tag-store
...
window.createTag = (name: string) => {
  const message = tagListModel.create(name);
  if (message === "duplicated") {
    window.alert("标签重复");
  } else if (message === "success") {
    window.alert("添加成功");
  }
};
window.findTag = (id: string) => {
  return window.tagList.filter((t) => t.id === id)[0];
};
window.removeTag = (id: string) => {
  return tagListModel.remove(id);
};
window.updateTag = (id: string, name: string) => {
  return tagListModel.update(id, name);
};

new Vue({
  router,
  store,
  render: (h) => h(App),
}).$mount("#app");
```

#### 封装record操作

```typescript
  //custom.d.ts
  ...
  recordList: RecordItem[];
  createRecord: (record: RecordItem) => void;
}
//main.ts record-store
window.recordList = recordListModel.fetch();
window.createRecord = (record: RecordItem) => {
  return recordListModel.create(record);
};
```

#### 模块化

```typescript
//在src目录下新建store文件夹添加tagStore.ts模块
import createId from "@/lib/createId";

const localStorageKeyName = "tagList";

const tagStore = {
  tagList: [] as Tag[],
  fetchTags() {
    this.tagList = JSON.parse(
      window.localStorage.getItem(localStorageKeyName) || "[]"
    );
    return this.tagList;
  },
  findTag(id: string) {
    return this.tagList.filter((t) => t.id === id)[0];
  },
  createTag(name: string) {
    const names = this.tagList.map((item) => item.name);
    if (names.indexOf(name) >= 0) {
      window.alert("标签名重复了");
      return "duplicated";
    }
    const id = createId().toString();
    this.tagList.push({ id, name: name });
    this.saveTags();
    window.alert("添加成功");
    return "success";
  },
  removeTag(id: string) {
    let index = -1;
    for (let i = 0; i < this.tagList.length; i++) {
      if (this.tagList[i].id === id) {
        index = i;
        break;
      }
    }
    this.tagList.splice(index, 1);
    this.saveTags();
    return true;
  },
  updateTag(id: string, name: string) {
    const idList = this.tagList.map((item) => item.id);
    if (idList.indexOf(id) >= 0) {
      const names = this.tagList.map((item) => item.name);
      if (names.indexOf(name) >= 0) {
        return "duplicated";
      } else {
        const tag = this.tagList.filter((item) => item.id === id)[0];
        tag.name = name;
        this.saveTags();
        return "success";
      }
    } else {
      return "not found";
    }
  },
  saveTags() {
    window.localStorage.setItem(
      localStorageKeyName,
      JSON.stringify(this.tagList)
    );
  },
};

tagStore.fetchTags();

export default tagStore;
```

```typescript
//在src目录下新建store文件夹添加recordStore.ts模块
import clone from "@/lib/clone";

const localStorageKeyName = "recordList";

const recordStore = {
  recordList: [] as RecordItem[],
  fetchRecords() {
    this.recordList = JSON.parse(
      window.localStorage.getItem(localStorageKeyName) || "[]"
    ) as RecordItem[];
    return this.recordList;
  },
  saveRecords() {
    window.localStorage.setItem(
      localStorageKeyName,
      JSON.stringify(this.recordList)
    );
  },
  createRecord(record: RecordItem) {
    const record2: RecordItem = clone(record);
    record2.createdAt = new Date();
    this.recordList && this.recordList.push(record2);
    recordStore.saveRecords();
  },
};

recordStore.fetchRecords();

export default recordStore;
```

+ 封装到src/store/index2.ts

```typescript
import recordStore from "@/store/recordStore";
import tagStore from "@/store/tagStore";

const store = {
  ...recordStore,
  ...tagStore,
};

console.log(store);

export default store;
```

+ 删除全局声明文件中的interface Window 和入口文件的 tagListModel和recordListModel 

#### 修复Tags组件

新增标签时，标签不显示内容

```typescript
//Tags组件中
create() {
    const name = window.prompt("输入标签名");
    if (name === "") {
      window.alert("标签名不能为空");
    } else if (this.dataSource) {
        //update事件每次触发新传一个数组，不能赋值给之前的
      this.$emit("update:value", [...this.dataSource, name]);
    }
  }
}
//修改
  const name = window.prompt('请输入标签名');
      if (!name) { return window.alert('标签名不能为空'); }
	//利用全局数据管理直接更新
      store.createTag(name);
```

#### 使用Vuex

```tsx
//src/store/index.ts
import clone from "@/lib/clone";
import createId from "@/lib/createId";
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

const store = new Vuex.Store({
  state: {
    recordList: [] as RecordItem[],
    tagList: [] as Tag[],
  },
  mutations: {
    fetchRecords(state) {
      state.recordList = JSON.parse(
        window.localStorage.getItem("recordList") || "[]"
      ) as RecordItem[];
    },
    createRecord(state, record) {
      const record2: RecordItem = clone(record);
      record2.createdAt = new Date();
      state.recordList.push(record2);
      store.commit("saveRecords");
    },
    saveRecords(state) {
      window.localStorage.setItem(
        "recordList",
        JSON.stringify(state.recordList)
      );
    },
    fetchTags(state) {
      return (state.tagList = JSON.parse(
        window.localStorage.getItem("tagList") || "[]"
      ));
    },
    createTag(state, name: string) {
      const names = state.tagList.map((item) => item.name);
      if (names.indexOf(name) >= 0) {
        window.alert("标签名重复了");
        return "duplicated";
      }
      const id = createId().toString();
      state.tagList.push({ id, name: name });
      store.commit("saveTags");
      window.alert("添加成功");
      return "success";
    },
    saveTags(state) {
      window.localStorage.setItem("tagList", JSON.stringify(state.tagList));
    },
  },
});

export default store;
```

+ 调用

```tsx
//Tags.vue
...
@Component({
  computed: {
    tagList() {
      return this.$store.state.tagList;
    },
  },
})
...
created() {
    this.$store.commit("fetchTags");
  }
...
 create() {
    const name = window.prompt("请输入标签名");
    if (!name) {
      return window.alert("标签名不能为空");
    }
    this.$store.commit("createTag", name);
  }
}
//Money.vue
...
@Component({
  components: { NumberPad, Tags, Notes, Types },
  computed: {
    recordList() {
      return store.recordList;
    },
  },
})
...
```

#### 抽离 TagHelper 作为 mixin

```typescript
//src/mixins/TagHelper.tsimport Vue from "vue";import Component from "vue-class-component";@Componentexport class TagHelper extends Vue {  createTag() {    const name = window.prompt("请输入标签名");    if (!name) {      return window.alert("标签名不能为空");    }    this.$store.commit("createTag", name);  }}export default TagHelper;
```

+ Labels.vue

```vue
...
<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
import Button from "@/components/Button.vue";
import store from "@/store/index2";
import { mixins } from "vue-class-component";
import TagHelper from "@/mixins/TagHelper";
@Component({
  components: { Button },
  computed: {
    tags() {
      return this.$store.state.tagList;
    },
  },
})
export default class Labels extends mixins(TagHelper) {
  beforeCreate() {
    this.$store.commit("fetchTags");
  }
}
</script>
```

+ TS 里使用 computed 要用 getter 语法

#### 完成vuex的使用

```typescript
//src/store/index.ts
...
etCurrentTag(state, id: string) {
      state.currentTag = state.tagList.filter((t) => t.id === id)[0];
    },
    updateTag(state, payload: { id: string; name: string }) {
      const { id, name } = payload;
      const idList = state.tagList.map((item) => item.id);
      if (idList.indexOf(id) >= 0) {
        const names = state.tagList.map((item) => item.name);
        if (names.indexOf(name) >= 0) {
          window.alert("标签名重复了");
        } else {
          const tag = state.tagList.filter((item) => item.id === id)[0];
          tag.name = name;
          store.commit("saveTags");
        }
      }
    },
    removeTag(state, id: string) {
      let index = -1;
      for (let i = 0; i < state.tagList.length; i++) {
        if (state.tagList[i].id === id) {
          index = i;
          break;
        }
      }
      if (index >= 0) {
        state.tagList.splice(index, 1);
        store.commit("saveTags");
        router.back();
      } else {
        window.alert("删除失败");
      }
    },
    createRecord(state, record) {
      const record2: RecordItem = clone(record);
      record2.createdAt = new Date();
      ...
      
//src/views/EditLabel.vue 
...
  created() {
    const id = this.$route.params.id;
    this.$store.commit("fetchTags");
    this.$store.commit("setCurrentTag", id);
    if (!this.tag) {
      this.$router.replace("/404");
    } else {
      console.log("has tag");
    }
  }
  update(name: string) {
    this.$store.commit("updateTag", {
      id: this.tag.id,
      name,
    });
  }
  remove() {
    if (this.tag) {
      this.$store.commit("removeTag", this.tag.id);
    }
  }
  ...
  
//src/views/Money.vue
<template>
  <Layout class-prefix="layout">
    <NumberPad :value.sync="record.amount" @submit="saveRecord" />
    <Types :value.sync="record.type" />
    <div class="notes">
      <Notes
	@@ -9,7 +9,7 @@
        placeholder="在此输入备注"
      />
    </div>
    <Tags />
  </Layout>
</template>
```

### 列表展示

#### 抽离Tabs在Statistics中使用

+ src/components/Money/Types.vue

```vue
<template>
  <ul class="types">
    <li
        //使用deep语法
      :class="{ [classPrefix + '-item']: classPrefix, selected: value === '-' }"
    >
      支出
    </li>
    <li
      :class="{ [classPrefix + '-item']: classPrefix, selected: value === '+' }"
    >
      收入
    </li>
  </ul>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop, Watch } from "vue-property-decorator";
@Component
export default class Types extends Vue {
  @Prop(String) readonly value!: string;
  @Prop(String) classPrefix?: string;
  selectType(type: string) {
    if (type !== "-" && type !== "+") {
      throw new Error("type is unknown");
	@@ -47,4 +56,4 @@ export default class Types extends Vue {
    }
  }
}
</style>
```

+ src/components/Tabs.vue 

```vue
<template>
  <ul class="tabs">
    <li
      v-for="item in dataSource"
      :key="item.value"
      :class="liClass(item)"
      @click="select(item)"
    >
      {{ item.text }}
    </li>
  </ul>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
type DataSourceItem = { text: string; value: string };
@Component
export default class Tabs extends Vue {
  @Prop({ required: true, type: Array })
  dataSource!: DataSourceItem[];
  @Prop(String)
  readonly value!: string;
  @Prop(String)
  classPrefix?: string;
  liClass(item: DataSourceItem) {
    return {
      [this.classPrefix + "-tabs-item"]: this.classPrefix,
      selected: item.value === this.value,
    };
  }
  select(item: DataSourceItem) {
    this.$emit("update:value", item.value);
  }
}
</script>

<style lang="scss" scoped>
.tabs {
  background: #c4c4c4;
  display: flex;
  text-align: center;
  font-size: 24px;
  > li {
    width: 50%;
    height: 64px;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    &.selected::after {
      content: "";
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 4px;
      background: #333;
    }
  }
}
</style>
```

+ src/views/Statistics.vue 

```vue
<template>
  <Layout>
    <Tabs class-prefix="type" :data-source="typeList" :value.sync="type" />
    <Tabs
      class-prefix="interval"
      :data-source="intervalList"
      :value.sync="interval"
    />
    <div>
      type: {{ type }}
      <br />
      interval: {{ interval }}
    </div>
  </Layout>
</template>

<script lang="ts">
import Types from "@/components/Money/Types.vue";
import Vue from "vue";
import { Component } from "vue-property-decorator";
import Tabs from "@/components/Tabs.vue";
@Component({
  components: { Tabs, Types },
})
export default class Statistics extends Vue {
  type = "-";
  interval = "day";
  intervalList = [
    { text: "按天", value: "day" },
    { text: "按周", value: "week" },
    { text: "按月", value: "month" },
  ];
  typeList = [
    { text: "支出", value: "-" },
    { text: "收入", value: "+" },
  ];
}
</script>

<style lang="scss" scoped>
::v-deep .type-tabs-item {
  background: white;
  &.selected {
    background: #c4c4c4;
    &::after {
      display: none;
    }
  }
}
</style>
```

#### 在 Money.vue 使用 Tabs 

+ 抽离intervalList.ts和recordTypeList.ts

```tsx
//src/constants/intervalList.ts
export default Object.freeze([
  { text: "按天", value: "day" },
  { text: "按周", value: "week" },
  { text: "按月", value: "month" },
]);
//src/constants/recordTypeList.ts
export default Object.freeze([
  { text: "支出", value: "-" },
  { text: "收入", value: "+" },
]);
```

+ Money

```vue
<template>
  <Layout class-prefix="layout">
    <NumberPad :value.sync="record.amount" @submit="saveRecord" />
    <Tabs :data-source="recordTypeList" :value.sync="record.type" />
    <div class="notes">
      <Notes
        @update:value="onUpdateNotes"
        ...
import Types from "@/components/Money/Types.vue";
import Notes from "@/components/Money/Notes.vue";
import { Component } from "vue-property-decorator";
import Tabs from "@/components/Tabs.vue";
import recordTypeList from "@/constants/recordTypeList";

@Component({
  components: { NumberPad, Tags, Notes, Types, Tabs },
})
export default class Money extends Vue {
  get recordList() {
    return this.$store.state.recordList;
  }
  recordTypeList = recordTypeList;
  record: RecordItem = {
    tags: [],
    notes: "",
    ...
```

#### 删除Types

#### 统计页面

+ src/views/Statistics.vue

```vue
<template>
  <Layout>
    <Tabs
      class-prefix="type"
      :data-source="recordTypeList"
      :value.sync="type"
    />
    <Tabs
      class-prefix="interval"
      :data-source="intervalList"
      :value.sync="interval"
    />
    <div>
      type: {{ type }}
      <br />
      interval: {{ interval }}
    </div>
    <ol>
      <li v-for="(group, index) in result" :key="index">
        <h3 class="title">{{ group.title }}</h3>
        <ol>
          <li v-for="item in group.items" :key="item.id" class="record">
            <span>{{ tagString(item.tags) }}</span>
            <span class="notes">{{ item.notes }}</span>
            <span>￥{{ item.amount }} </span>
          </li>
        </ol>
      </li>
    </ol>
  </Layout>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
import Tabs from "@/components/Tabs.vue";
import intervalList from "@/constants/intervalList";
import recordTypeList from "@/constants/recordTypeList";
@Component({
  components: { Tabs },
})
export default class Statistics extends Vue {
  tagString(tags: Tag[]) {
    return tags.length === 0 ? "无" : tags.join(",");
  }
  get recordList() {
    return (this.$store.state as RootState).recordList;
  }
  get result() {
    const { recordList } = this;
    type HashTableValue = { title: string; items: RecordItem[] };
    const hashTable: { [key: string]: HashTableValue } = {};
    for (let i = 0; i < recordList.length; i++) {
      const [date, time] = recordList[i].createdAt!.split("T");
      hashTable[date] = hashTable[date] || { title: date, items: [] };
      hashTable[date].items.push(recordList[i]);
    }
    return hashTable;
  }
  beforeCreate() {
    this.$store.commit("fetchRecords");
  }
  type = "-";
  interval = "day";
  intervalList = intervalList;
  recordTypeList = recordTypeList;
}
</script>

<style lang="scss" scoped>
::v-deep {
  .type-tabs-item {
    background: white;
    &.selected {
      background: #c4c4c4;
      &::after {
        display: none;
      }
    }
  }
  .interval-tabs-item {
    height: 48px;
  }
}
%item {
  padding: 8px 16px;
  line-height: 24px;
  display: flex;
  justify-content: space-between;
  align-content: center;
}
.title {
  @extend %item;
}
.record {
  background: white;
  @extend %item;
}
.notes {
  margin-right: auto;
  margin-left: 16px;
  color: #999;
}
</style>
```

#### ISO8601和day.js

+ 列表需要通过时间来划分列表，这里使用第三方库day.js

```vue
<template>
  <Layout>
    <Tabs
      class-prefix="type"
      :data-source="recordTypeList"
      :value.sync="type"
    />
    <ol v-if="groupedList.length > 0">
      <li v-for="(group, index) in groupedList" :key="index">
        <h3 class="title">
          {{ beautify(group.title) }} <span>￥{{ group.total }}</span>
        </h3>
        <ol>
          <li v-for="item in group.items" :key="item.id" class="record">
            <span>{{ tagString(item.tags) }}</span>
            <span class="notes">{{ item.notes }}</span>
            <span>￥{{ item.amount }} </span>
          </li>
        </ol>
      </li>
    </ol>
  <div v-else>
  无
  </div>
  </Layout>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
import Tabs from "@/components/Tabs.vue";
import recordTypeList from "@/constants/recordTypeList";
import dayjs from "dayjs";
import clone from "@/lib/clone";
@Component({
  components: { Tabs },
})
export default class Statistics extends Vue {
  tagString(tags: Tag[]) {
    return tags.length === 0 ? "无" : tags.map((t) => t.name).join("，");
  }
  beautify(string: string) {
    const day = dayjs(string);
    const now = dayjs();
    if (day.isSame(now, "day")) {
      return "今天";
    } else if (day.isSame(now.subtract(1, "day"), "day")) {
      return "昨天";
    } else if (day.isSame(now.subtract(2, "day"), "day")) {
      return "前天";
    } else if (day.isSame(now, "year")) {
      return day.format("M月D日");
    } else {
      return day.format("YYYY年M月D日");
    }
  }
  get recordList() {
    return (this.$store.state as RootState).recordList;
  }
  //将时间排序（数据排序）
  get groupedList() {
    const { recordList } = this;
    if (recordList.length === 0) {
      return [];
    }
    const newList = clone(recordList)
      .filter((r) => r.type === this.type)
      .sort(
        (a, b) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf()
      );
    if (newList.length === 0) {
      return [] as Result;
    }
    type Result = { title: string; total?: number; items: RecordItem[] }[];
    const result: Result = [
      {
        title: dayjs(newList[0].createdAt).format("YYYY-MM-DD"),
        items: [newList[0]],
      },
    ];
    for (let i = 1; i < newList.length; i++) {
      const current = newList[i];
      const last = result[result.length - 1];
      if (dayjs(last.title).isSame(dayjs(current.createdAt), "day")) {
        last.items.push(current);
      } else {
        result.push({
          title: dayjs(current.createdAt).format("YYYY-MM-DD"),
          items: [current],
        });
      }
    }
    result.map((group) => {
      group.total = group.items.reduce((sum, item) => {
        return sum + item.amount;
      }, 0);
    });
    return result;
  }
  beforeCreate() {
    this.$store.commit("fetchRecords");
  }
  type = "-";
  recordTypeList = recordTypeList;
}
</script>
```

#### Money页面添加createdAt输入框

+ Notes.vue

```vue
<template>
  <div>
    <label class="notes">
      <span class="name">{{ fieldName }}</span>
      <template v-if="type === 'date'"
        ><input
          :value="x(value)"
          @input="onValueChange($event.target.value)"
          :type="type || 'text'"
          :placeholder="placeholder"
      /></template>
      <template v-else
        ><input
          :value="value"
          @input="onValueChange($event.target.value)"
          :type="type || 'text'"
          :placeholder="placeholder"
      /></template>
    </label>
  </div>
</template>

<script lang="ts">
import dayjs from "dayjs";
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
@Component
export default class Notes extends Vue {
  @Prop({ default: "" }) readonly value!: string;
  @Prop({ required: true }) fieldName!: string;
  @Prop() placeholder?: string;
  @Prop() type?: string;
  onValueChange(value: string) {
    this.$emit("update:value", value);
  }
  x(isoString: string) {
    return dayjs(isoString).format("YYYY-MM-DD");
  }
}
</script>
```

+ Money.vue

```vue
<template>
  <Layout class-prefix="layout">
    <NumberPad :value.sync="record.amount" @submit="saveRecord" />
    <Tabs
      class="tabs"
      :data-source="recordTypeList"
      :value.sync="record.type"
    />
    <div class="createAt">
      <Notes field-name="日期" :value.sync="record.createdAt" type="date" />
    </div>
    <div class="notes">
      <Notes
        field-name="备注"
...
    notes: "",
    type: "-",
    amount: 0,
    createdAt: new Date().toISOString(),
  };
  created() {
    this.$store.commit("fetchRecords");
...
```

#### 使用echarts

+ src/components/Chart.vue

```vue
<template>
  <div id="main" ref="main"></div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop } from "vue-property-decorator";
import * as echarts from "echarts";
@Component
export default class Chart extends Vue {
  @Prop() option?: echarts.EChartsOption;
  mounted() {
    if (this.option === undefined) {
      return console.error("option 为空");
    }
    const myChart = echarts.init(this.$refs.main as HTMLDivElement);
    myChart.setOption(this.option);
  }
}
</script>
<style lang="scss">
#main {
  height: 200px;
}
</style>
```

+ src/views/Statistics.vue

```vue
<template>
  <Layout class="layout">
    <NavTop
      ><Tabs
        class-prefix="type"
        :data-source="recordTypeList"
        :value.sync="type"
    /></NavTop>
    <div class="chart-wrapper" ref="chartWrapper">
      <Chart class="chart" :option="chartOption" />
    </div>
    <ol v-if="groupedList.length > 0">
      <li v-for="(group, index) in groupedList" :key="index">
        <h3 class="title">
...
import recordTypeList from "@/constants/recordTypeList";
import dayjs from "dayjs";
import clone from "@/lib/clone";
import Chart from "../components/Chart.vue";
@Component({
  components: { Tabs, Chart },
})
export default class Statistics extends Vue {
  tagString(tags: Tag[]) {
...
 return day.format("YYYY年M月D日");
    }
  }
  mounted() {
    const div = this.$refs.chartWrapper as HTMLDivElement;
    div.scrollLeft = div.scrollWidth;
  }
  get recordList() {
    return (this.$store.state as RootState).recordList;
  }
...
type = "-";
  recordTypeList = recordTypeList;
  get chartOption() {
    return {
      grid: {
        left: 0,
        right: 0,
      },
      tooltip: {
        trigger: "axis",
        formatter: "{c}",
        axisPointer: {
          animation: true,
        },
      },
      xAxis: {
        type: "category",
        splitLine: {
          show: false,
        },
        axisTick: { alignWithLabel: true },
        data: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
      },
      yAxis: {
        type: "value",
        splitLine: {
          show: true,
        },
      },
      series: [
        {
          itemStyle: { color: "#acd1c0" },
          type: "line",
          data: [1222, 321, 124, 121, 213, 123, 3214, 5, 512, 53],
        },
      ],
    };
  }
}
</script>
          
...
.chart {
  width: 200%;
}
.layout {
  overflow: hidden;
}
.chart {
  width: 150%;
  &-wrapper {
    overflow: auto;
    &::-webkit-scrollbar {
      display: none;
    }
  }
}
</style>
```

#### 自定义 完善echarts 

+ src/components/Chart.vue

```vue
<template>
  <div id="main" ref="main"></div>
</template>

<script lang="ts">
import Vue from "vue";
import { Component, Prop, Watch } from "vue-property-decorator";
import * as echarts from "echarts";
import { EChartsOption, ECharts } from "echarts";
@Component
export default class Chart extends Vue {
  @Prop() option?: EChartsOption;
  myChart?: ECharts;
  mounted() {
    if (this.option === undefined) {
      return console.error("option 为空");
    }
    this.myChart = echarts.init(this.$refs.main as HTMLDivElement);
    this.myChart.setOption(this.option);
  }
  @Watch("option")
  onOptionsChange(newValue: EChartsOption) {
    this.myChart!.setOption(newValue);
  }
}
</script>
...
```

+ src/views/Statistics.vue

```vue
...
<script lang='ts'>
...
import _ from "lodash";
import day from "dayjs";
...
type = "-";
  recordTypeList = recordTypeList;
    //通过桶排序展示三十天数据
  get keyValueList() {
    const today = new Date();
    const array = [];
    for (let i = 0; i <= 30; i++) {
      const dateString = day(today).subtract(i, "day").format("YYYY-MM-DD");
      const found = _.find(this.groupedList, {
        title: dateString,
      });
      array.push({
        key: dateString,
        value: found ? found.total : 0,
      });
    }
    array.sort((a, b) => {
      if (a.key > b.key) {
        return 1;
      } else if (a.key === b.key) {
        return 0;
      } else {
        return -1;
      }
    });
    return array;
  }
  get chartOption() {
    const keys = this.keyValueList.map((item) => item.key);
    const values = this.keyValueList.map((item) => item.value);
    return {
      grid: {
        left: 0,
...
          show: false,
        },
        axisTick: { alignWithLabel: true },
        axisLabel: {
          formatter: function (value: string, index: number) {
            return value.substring(5);
          },
        },
        data: keys,
      },
      yAxis: {
        type: "value",
...
      },
      yAxis: {
        type: "value",
        splitLine: {
          show: true,
        },
      },
      series: [
        {
          itemStyle: { color: "#acd1c0" },
          type: "line",
          data: values,
        },
      ],
    };
</script>
```

