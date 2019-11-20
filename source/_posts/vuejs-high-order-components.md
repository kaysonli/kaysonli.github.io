---
title: 你已经是成熟的前端，应该学会自己使用 Vue 高阶组件了
date: 2019-10-26 20:04:54
tags: Vue.js
---

高阶组件(HOC)是一种架构模式，在React中非常常见，但也可以在Vue中使用。它可以被描述为一种在组件之间共享公共功能而不需要重复代码的方法。HOC的目的是增强组件的功能。它允许在项目中实现可重用性和可维护性。

只要你向一个方法传入组件，然后返回一个新的组件，这就是一个HOC。

高阶组件在以下方面非常有用:
* 操作属性。
* 操作数据和数据抽象。
* 代码重用

##  预备知识

在我们开始教程之前，需要了解以下几点：
*   使用Vue框架的经验。
*   知道如何使用[vue-cli](https://cli.vuejs.org/)设置应用程序。
*   JavaScript 和 Vue 的基本知识
*   Node (8)
*   npm (5.2.0)

在开始本教程之前，请确保安装了Node和npm。
<!-- more -->
## Vue中的高阶组件模式

虽然高阶组件通常与React相关联，但是为Vue组件创建高阶组件是很有可能的。在Vue中创建高阶组件的模式如下所示。
```
    // hocomponent.js
    import Vue from 'vue'
    import ComponentExample from '@/components/ComponentExample.vue'

    const HoComponent = (component) => {
      return Vue.component('withSubscription', {
        render(createElement) {
          return createElement(component)
        } 
      }
    }
    const HoComponentEnhanced = HoComponent(ComponentExample);
```

如上面的代码块所示，`HoComponent` 函数接受一个组件作为参数，并创建一个新组件来渲染传进来的组件。
## 一个简单的 HOC 示例

在本教程中，我们将介绍一个使用高阶组件的示例。在介绍高阶组件之前，我们将了解在没有高阶组件的情况下，当前的代码库是如何工作的，然后了解如何进行抽象。
[https://codesandbox.io/embed/llvq04nx4l](https://codesandbox.io/embed/llvq04nx4l)

正如上面的CodeSandbox所示，该应用程序会显示一个纸业公司及其净资产的列表，以及《办公室》(美国)中的人物及其获奖情况。

我们获得应用程序所需的所有数据来源只有一个，那就是`mockData.js` 文件。
```
    // src/components/mockData.js
    const staff = [
        {
          name: "Michael Scott",
          id: 0,
          awards: 2
        },
        {
          name: "Toby Flenderson",
          id: 1,
          awards: 0
        },
        {
          name: "Dwight K. Schrute",
          id: 2,
          awards: 10
        },
        {
          name: "Jim Halpert",
          id: 3,
          awards: 1
        },
        {
          name: "Andy Bernard",
          id: 4,
          awards: 0
        },
        {
          name: "Phyllis Vance",
          id: 5,
          awards: 0
        },
        {
          name: "Stanley Hudson",
          id: 6,
          awards: 0
        }
    ];
    const paperCompanies = [
      {
        id: 0,
        name: "Staples",
        net: 10000000
      },
      {
        id: 1,
        name: "Dundler Mufflin",
        net: 5000000
      },
      {
        id: 2,
        name: "Michael Scott Paper Company",
        net: 300000
      },
      {
        id: 3,
        name: "Prince Family Paper",
        net: 30000
      }
    ];

    export default {
      getStaff() {
        return staff;
      },
      getCompanies() {
        return paperCompanies;
      },
      increaseAward(id) {
        staff[id].awards++;
      },
      decreaseAward(id) {
        staff[id].awards--;
      },
      setNetWorth(id) {
        paperCompanies[id].net = Math.random() * (5000000 - 50000) + 50000;
      }
    };
```

在上面的代码片段中，有几个`const`变量保存了公司和员工列表的信息。我们也导出了一些函数，实现以下功能：
*   返回员工列表
*   返回公司列表
*   增加和减少对特定员工的奖励，最后
*   最后，设定公司的净值

接下来，我们看看 `Staff.vue` 和 `Companies.vue` 组件。

```
    // src/components/Staff.vue
    <template>
      <main>
        <h3>Staff List</h3>
        <div v-for="(staff, i) in staffList" :key="i">
          {{ staff.name }}: {{ staff.awards }} Salesman of the year Award 🎉
          <button @click="increaseAwards(staff.id);">+</button>
          <button @click="decreaseAwards(staff.id);">-</button>
        </div>
      </main>
    </template>
    <script>
      import mockData from "./mockData.js";
      export default {
        data() {
          return {
            staffList: mockData.getStaff()
          };
        },
        methods: {
          increaseAwards(id) {
            mockData.increaseAward(id);
            this.staffList = mockData.getStaff();
          },
          decreaseAwards(id) {
            mockData.decreaseAward(id);
            this.staffList = mockData.getStaff();
          }
        }
      };
    </script>
```

在上面的代码块中，数据实例变量`staffList`被赋值为函数`mockData.getStaff()`返回的内容。我们也有`increaseAwards`和`decreaseAwards`函数，分别调用`mockData.increaseAward` 和 `mockData.decreaseAward`。传递给这些函数的`id` 是从渲染的模板中获得的。
```
    // src/components/Companies.vue
    <template>
      <main>
        <h3>Paper Companies</h3>
        <div v-for="(companies, i) in companies" :key="i">
          {{ companies.name }} - ${{ companies.net
          }}<button @click="setWorth(companies.id);">Set Company Value</button>
        </div>
      </main>
    </template>

    <script>
      import mockData from "./mockData.js";
        export default {
        data() {
          return {
            companies: mockData.getCompanies()
          };
        },
        methods: {
          setWorth(id) {
            mockData.setNetWorth(id);
            this.companies = mockData.getCompanies();
          }
        }
      };
    </script>
```

在上面的代码块中，数据实例变量`companies` 被赋值为函数`mockData.getCompanies()`的返回内容。我们还有`setWorth`函数，它通过将公司的 `id` 传递给`mockData.setNetWorth`来设置一个随机值作为净值。传递给函数的`id`是从渲染的模板中获得的。

现在我们已经看到了这两个组件是如何工作的，我们可以找出它们之间的共同点，并将其抽象如下：
*   从数据源获取数据 (mockData.js)
*   onClick 函数

我们来看看如何将上面的操作放到高阶组件中，以避免代码重复并确保可重用性。

你可以使用[Vue-cli](https://cli.vuejs.org/) 继续创建一个新的Vue项目。Vue CLI是一个用于快速开发Vue.js项目的完整系统，它确保你有一个可用的开发环境，而不需要构建配置。你可以使用下面的命令安装`vue-cli`。
```
    npm install -g @vue/cli

```

安装完成后，你可以使用下面的命令创建一个新项目，其中`vue-hocomponent` 是应用程序的名称。请确保选择默认预设，因为不需要自定义选项。
```
    vue create vue-hocomponent

```

安装完成后，你可以继续创建以下文件，然后使用上面共享的片段内容进行编辑。
*   `src/components` 目录下的`Staff.vue` 文件。
*   `src/components` 目录下的 `Companies.vue` 文件。
*   `src/components` 目录下的`mockData.js` 文件。

或者，你也可以直接 fork  [CodeSandbox](https://codesandbox.io/s/llvq04nx4l)  里的应用跟着本教程操作。

下一步是创建一个用于抽象的高阶组件文件。在`src`文件夹中创建一个名为`HoComponent.js`的文件，编辑以下内容：
```
    // src/components/HoComponent.js
    import Vue from "vue";
    import mockData from "./mockData.js";

    const HoComponent = (component, fetchData) => {
      return Vue.component("HoComponent", {
        render(createElement, context) {
          return createElement(component, {
            props: {
              returnedData: this.returnedData
            }
          });
        },
        data() {
          return {
            returnedData: fetchData(mockData)
          };
        }
      });
    };

    export default HoComponent;
```

在上面的代码中，Vue 和 `mockData` 文件中的数据被导入组件。

`HoComponent` 函数接受两个参数，一个组件和`fetchData`。`fetchData`方法用于确定要在表示组件中显示什么。这意味着无论在哪里使用高阶组件，作为`fetchData` 传递的函数都将被用来从`mockData` 中获取数据。

然后将数据实例`returnedData` 设置为`fetchData`的内容，然后作为`props` 传递给在高阶组件中创建的新组件。

让我们看看新创建的高阶组件如何在应用程序中使用。我们需要编辑`Staff.vue` 和`Companies.vue`。
```
    // src/components/Staff.vue
    <template>
      <main>
        <h3>Staff List</h3>
        <div v-for="(staff, i) in returnedData" :key="i">
          {{ staff.name }}: {{ staff.awards }} Salesman of the year Award 🎉
          <button @click="increaseAwards(staff.id);">+</button>
          <button @click="decreaseAwards(staff.id);">-</button>
        </div>
      </main>
    </template>
    <script>
      export default {
        props: ["returnedData"]
      };
    </script>
```

```
    // src/components/Companies.vue
    <template>
      <main>
        <h3>Paper Companies</h3>
        <div v-for="(companies, i) in returnedData" :key="i">
          {{ companies.name }} - ${{ companies.net
          }}<button @click="setWorth(companies.id);">Set Company Value</button>
        </div>
      </main>
    </template>
    <script>
      export default {
        props: ["returnedData"]
      };
    </script>
```

正如你在上面的代码块中看到的，对于这两个组件，我们去掉了函数和数据实例变量，显示内容所需的所有数据现在都将从这些props中获得。对于删掉的函数，我们将很快会讲到。

在 `App.vue` 组件中，用以下代码编辑`script` 标签中的现有内容：
```
    // src/App.vue
    <script>
      // import the Companies component
      import Companies from "./components/Companies";
      // import the Staff component
      import Staff from "./components/Staff";
      // import the higher order component
      import HoComponent from "./components/HoComponent.js";

      // Create a const variable which contains the Companies component wrapped in the higher order component
      const CompaniesComponent = HoComponent(Companies, mockData => mockData.getCompanies()
      );

      // Create a const variable which contains the Staff component wrapped in the higher order component
      const StaffComponent = HoComponent(Staff, mockData => mockData.getStaff());

      export default {
        name: "App",
        components: {
          CompaniesComponent,
          StaffComponent
        }
      };
    </script>
```

在上面的代码块中，`HoComponent`用于包装 `Staff` 和 `Companies` 组件。每个组件作为`HoComponent` 的第一个参数传入，第二个参数是一个函数，它返回另一个函数，指定应该从`mockData`获取什么数据。这是我们之前创建的高阶组件(HoComponent.js)中的`fetchData` 函数。

如果你现在刷新应用程序，你应该仍然可以看到来自`mockData` 文件的数据像往常一样呈现。唯一的区别是，这些按钮无法工作，因为它们还没有绑定到任何函数。让我们解决这个问题。

我们先修改`Staff.vue` 和 `Companies.vue`这两个文件：
```
    // src/components/Staff.vue
    <template>
      <main>
        <h3>Staff List</h3>
        <div v-for="(staff, i) in returnedData" :key="i">
          {{ staff.name }}: {{ staff.awards }} Salesman of the year Award 🎉
          <button @click="$emit('click', { name: 'increaseAward', id: staff.id });">
          +
          </button>
          <button @click="$emit('click', { name: 'decreaseAward', id: staff.id });">
          -
          </button>
        </div>
      </main>
    </template>
    <script>
      export default {
        props: ["returnedData"]
      };
    </script>
```

```
    // src/components/Companies.vue
    <template>
      <main>
        <h3>Paper Companies</h3>
        <div v-for="(companies, i) in returnedData" :key="i">
          {{ companies.name }} - ${{ companies.net
          }}<button
          @click="$emit('click', { name: 'setNetWorth', id: companies.id });"
          >
          Set Company Value
          </button>
        </div>
      </main>
    </template>
    <script>
      export default {
        props: ["returnedData"]
      };
    </script>
```

在上面的两个代码片段中，我们发送了事件，这些事件将在父组件`App.vue`中使用。我们发送了一个对象，它包含值、与试图执行的操作相关联的函数名以及被单击元素的`id`。别忘了`mockData.js` 文件中定义了`increaseAward`, `decreaseAward` 和`setNetWorth` 函数。

接下来，我们开始编辑父组件`App.vue` ，让其对子组件发送过来的数据进行响应。`App.vue` 更改如下：
```
    // src/App.vue
    <template>
      <div id="app">
        <CompaniesComponent @click="onEventHappen" />
        <StaffComponent @click="onEventHappen" />
      </div>
    </template>

    <script>
      // import the Companies component
      import Companies from "./components/Companies";
      // import the Staff component
      import Staff from "./components/Staff";
      // import the higher order component
      import HoComponent from "./components/HoComponent.js";
      // import the source data from mockData only to be used for event handlers
      import sourceData from "./components/mockData.js";
      // Create a const variable which contains the Companies component wrapped in the higher order component
      const CompaniesComponent = HoComponent(Companies, mockData =>
      mockData.getCompanies()
      );
      // Create a const variable which contains the Staff component wrapped in the higher order component
      const StaffComponent = HoComponent(Staff, mockData => mockData.getStaff());

      export default {
        name: "App",
        components: {
          CompaniesComponent,
          StaffComponent
          },
        methods: {
          onEventHappen(value) {
            // set the variable setFunction to the name of the function that was passed iin the value emitted from child component i.e. if value.name is 'increaseAward', setFunction is set to increaseAward()
            let setFunction = sourceData[value.name];
            // call the corresponding function with the id passed as an argument.
            setFunction(value.id);
          }
        }
      };
    </script>

    <style>
    #app {
      font-family: "Avenir", Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      margin-top: 60px;
    }
    </style>
```
在上面的代码块中，我们在`App.vue`组件中添加了一个事件监听器。vue的组件。每当`Staff.vue` 或`Companies.vue` 组件被点击时，`onEventHappen` 方法将会被调用。

在`onEventHappen`方法中，我们将变量`setFunction` 设置为从子组件发出的值中传递的函数名，也就是说，如果`value.name`是'increaseAward'，那么`setFunction`设置为`increaseAward()`。`setFunction` 将以id作为参数执行。

最后，为了将事件监听器传递给封装在高阶组件中的组件，我们需要在 `HoComponent.js`文件中添加下面这行代码。
```
    props: {
    returnedData: this.returnedData
    },
    on: { ...this.$listeners }
```
你现在可以刷新应用程序，所有的按钮都可以正常工作。
## vue-hoc

或者，您可以使用[vue-hoc](https://github.com/jackmellis/vue-hoc)库来帮助创建高阶组件。vue-hoc帮助你轻松地创建高阶组件，你要做的就是传递基本组件、应用于HOC的一系列组件选项和在渲染阶段传递给组件的数据属性。

vue-hoc 可用如下命令安装：
```
    npm install --save vue-hoc

```

vue-hoc插件有一些例子可以让你开始创建更高阶的组件，你可以[查看这里](https://github.com/jackmellis/vue-hoc/blob/master/packages/vue-hoc/README.md).
## 总结

在本教程中我们了解到，高阶组件的主要用途是增强应用程序中表现类组件的可重用性和逻辑。
另外还了解到，高阶组件在以下方面有用处：

* 操作属性。
* 操作数据和数据抽象。
* 代码重用

然后我们看了一个如何在 Vue 中创建和使用高阶组件的例子。例子的源码可在[GitHub](https://github.com/yomete/vue-hocomponent)上查看。

[原文](https://pusher.com/tutorials/higher-order-components-vue)

