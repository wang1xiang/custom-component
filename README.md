#### 处理组件边界情况

[官方网站](https://cn.vuejs.org/v2/guide/components-edge-cases.html)

- 访问根实例`$root`

  小型应用中可以在 vue 根实例里存储共享数据，组件中可以通过 `$root` 访问根实例，不过这个模式扩展到中大型应用来说就不然了

  ```html
  <!-- 01-root.vue -->
  <div>
      <!--
      小型应用中可以在 vue 根实例里存储共享数据
      组件中可以通过 $root 访问根实例
      -->
      $root.title：{{ $root.title }}
      <br>
      <button @click="$root.handle">获取 title</button>&nbsp;&nbsp;
      <button @click="$root.title = 'Hello $root'">改变 title</button>
  </div>
  ```

  ```js
  // main.js
  new Vue({
    render: (h) => h(App),
    data: {
      title: '根实例 - Root',
    },
    methods: {
      handle () {
        console.log(this.title)
      }
    }
  }).$mount('#app')
  ```

- 访问父组件实例

  和 `$root` 类似，`$parent` 可以用来从一个子组件访问父组件的实例，触达父级组件会使得你的应用更难调试和理解

  ```vue
  <!-- parent.vue -->
  <script>
  export default {
    data () {
      return {
        title: '获取父组件实例'
      }
    },
    methods: {
      handle () {
        console.log(this.title)
      }
    }
  }
  </script>
  <!-- child.vue -->
  <template>
    <div class="child">
      child<br>
      $parent.title：{{ $parent.title }}<br>
      <button @click="$parent.handle">获取 $parent.title</button>
      <button @click="$parent.title = 'Hello $parent.title'">改变 $parent.title</button>
    
      <grandson></grandson>
    </div>
  </template>
  <!-- grandson.vue -->
  <template>
    <div class="grandson">
      grandson<br>
      $parent.$parent.title：{{ $parent.$parent.title }}<br>
      <button @click="$parent.$parent.handle">获取 $parent.$parent.title</button>
      <button @click="$parent.$parent.title = 'Hello $parent.$parent.title'">改变 $parent.$parent.title</button>
    </div>
  </template>
  ```

  在更底层组件中，可以使用`$parent.$parent`获取更高级组件实例

- 访问子组件实例或子元素

  可以通过使用`ref`为子组件赋予一个ID，可以在JavaScript中直接访问子组件或元素

  - 通过`ref`获取子组件

    ```vue
    <template>
      <div>
        <myinput ref="mytxt"></myinput>
        <button @click="focus">获取焦点</button>
      </div>
    </template>
    
    <script>
    import myinput from './02-myinput'
    export default {
      components: {
        myinput
      },
      methods: {
        focus () {
          this.$refs.mytxt.focus()
        }
      }
    }
    </script>
    ```

  - 通过`ref`获取DOM元素

    ```vue
    <template>
      <div>
        <input v-model="value" type="text" ref="txt">
      </div>
    </template>
    
    <script>
    export default {
      data () {
        return {
          value: 'default'
        }
      },
      methods: {
        focus () {
          this.$refs.txt.focus()
        }
      }
    }
    </script>
    ```

- 依赖注入`provide&inject`

  使用 `$parent` 无法很好的扩展到更深层级的嵌套组件上，所以就需要使用依赖注入：`provide` 和 `inject`。

  `provide` 选项允许我们指定我们想要提供给后代组件的数据/方法

  ```vue
  <!-- parent.vue -->
  <template>
    ...
  </template>
  
  <script>
  export default {
    provide () {
      return {
        title: this.title,
        handle: this.handle
      }
    },
    data () {
      return {
        title: '父组件 provide'
      }
    },
    methods: {
      handle () {
        console.log(this.title)
      }
    }
  }
  </script>
  ```

  然后再任何后代组件里，可以使用 `inject` 选项来接收指定 property

  ```js
  inject: ['title', 'handle']
  ```

- [$attrs/$listeners](https://cn.vuejs.org/v2/api/#vm-attrs)

  `$attrs`：把父组件中非prop属性绑定到内部组件

  `$listeners`：把父组件中的DOM对象的原生事件绑定到内部组件

  如果不希望组件的根元素继承 attribute，在组件的选项中设置 `inheritAttrs: false`

  ```vue
  <!-- parent.vue -->
  <template>
    <div>
      <myinput
        required
        placeholder="Enter your username"
        class="theme-dark"
        @focus="onFocus"
        @input="onInput"
        data-test="test">
      </myinput>
      <button @click="handle">按钮</button>
    </div>
  </template>
  <script>
  import myinput from './02-myinput'
  export default {
    components: {
      myinput
    },
    methods: {
      handle () {
        console.log(this.value)
      },
      onFocus (e) {
        console.log(e)
      },
      onInput (e) {
        console.log(e.target.value)
      }
    }
  }
  </script>
  <!-- child.vue -->
  <template>
    <!--
      1. 从父组件传给自定义子组件的属性，如果没有 prop 接收
         会自动设置到子组件内部的最外层标签上
         如果是 class 和 style 的话，会合并最外层标签的 class 和 style 
    -->
    <!-- <input type="text" class="form-control" :placeholder="placeholder"> -->
  
    <!--
      2. 如果子组件中不想继承父组件传入的非 prop 属性，可以使用 inheritAttrs 禁用继承
         然后通过 v-bind="$attrs" 把外部传入的非 prop 属性设置给希望的标签上
  
         但是这不会改变 class 和 style
    -->
    <!-- <div>
      <input type="text" v-bind="$attrs" class="form-control">
    </div> -->
  
  
    <!--
      3. 注册事件
    -->
  
    <!-- <div>
      <input
        type="text"
        v-bind="$attrs"
        class="form-control"
        @focus="$emit('focus', $event)"
        @input="$emit('input', $event)"
      >
    </div> -->
  
  
    <!--
      4. $listeners 父组件传过来原生的dom事件
    -->
  
    <div>
      <input
        type="text"
        v-bind="$attrs"
        class="form-control"
        v-on="$listeners"
      >
    </div>
  </template>
  
  <script>
  export default {
    // props: ['placeholder', 'style', 'class']
    // props: ['placeholder']
    inheritAttrs: false
  }
  </script>
  ```

#### 快速原型开发

VueCli提供一个插件可以进行[原型快速开发](https://cli.vuejs.org/zh/guide/prototyping.html)

- 安装

  ```bash
  npm install -g @vue/cli-service-global
  ```

- 使用 `vue serve`

  ```text
  在开发环境模式下零配置为 .js 或 .vue 文件启动一个服务器
  
  Options:
  
    -o, --open  打开浏览器
    -c, --copy  将本地 URL 复制到剪切板
    -h, --help  输出用法信息
  ```

- 创建App.vue文件

  ```vue
  <template>
    <h1>Hello!</h1>
  </template>
  ```

- 运行

  ```bash
  vue serve
  ```

  vue serve会在当前目录自动推导入口文件——入口可以是 `main.js`、`index.js`、`App.vue` 或 `app.vue` 中的一个

  可以显式地指定入口文件：

  ```bash
  vue serve MyComponent.vue
  ```

##### 安装ElementUI

- 创建文件，初始化package.json

  ```bash
  mkdir custom-component
  cd custom-component
  yarn init -y
  ```

- 安装ElementUI

  ```bash
  vue add element
  ```

- 创建main.js，加载ElementUI，使用Vue.use()安装插件

  ```js
  import Vue from 'vue'
  import ElementUI from 'element-ui'
  import 'element-ui/lib/theme-chalk/index.css'
  import Login from './src/Login.vue'
  
  Vue.use(ElementUI)
  
  new Vue({
    el: '#app',
    render: h => h(Login)
  })
  ```

- 创建src/Login.vue，并使用vue serve运行

  ```vue
  <template>
    <el-form class="form" ref="form" :model="user" :rules="rules">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="user.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input type="password" v-model="user.password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">登 录</el-button>
      </el-form-item>
    </el-form>
  </template>
  
  <script>
  export default {
    name: 'Login',
    data() {
      return {
        user: {
          username: '',
          password: '',
        },
        rules: {
          username: [
            {
              required: true,
              message: '请输入用户名',
            },
          ],
          password: [
            {
              required: true,
              message: '请输入密码',
            },
            {
              min: 6,
              max: 12,
              message: '请输入6-12位密码',
            },
          ],
        },
      }
    },
    methods: {
      login() {
        this.$refs.form.validate((valid) => {
          if (valid) {
            alert('验证成功')
          } else {
            alert('验证失败')
            return false
          }
        })
      },
    },
  }
  </script>
  
  <style>
  .form {
    width: 30%;
    margin: 150px auto;
  }
  </style>
  ```

##### 组件开发

##### 步骤条组件

- 创建Steps.vue

  ```vue
  <template>
    <div class="lg-steps">
      <div class="lg-steps-line"></div>
      <div
        class="lg-step"
        v-for="index in count"
        :key="index"
        :style="{ color: active >= index ? activeColor : defaultColor }"
      >
        {{ index }}
      </div>
    </div>
  </template>
  
  <script>
  import './steps.css'
  export default {
    name: 'LgSteps',
    props: {
      count: {
        type: Number,
        default: 3,
      },
      active: {
        type: Number,
        default: 0,
      },
      activeColor: {
        type: String,
        default: 'red',
      },
      defaultColor: {
        type: String,
        default: 'green',
      },
    },
  }
  </script>
  <style></style>
  ```

  使用`vue serve ./Steps.vue`运行，打开页面访问此组件

- 创建Steps-test.vue组件，进行测试

  ```vue
  <template>
    <div>
      <steps :count="count" :active="active"></steps>
      <button @click="next">下一步</button>
    </div>
  </template>
  
  <script>
  import Steps from './Steps.vue'
  export default {
    components: {
      Steps,
    },
    data() {
      return {
        count: 4,
        active: 0,
      }
    },
    methods: {
      next() {
        if (this.active < this.count) {
          this.active++
        }
      },
    },
  }
  </script>
  <style></style>
  ```

##### 表单组件

- 模仿Element的Form组件，在src下创建form文件夹

- 创建Form.vue、FormItem.vue、Input.vue以及Button.vue组件

  ```vue
  <!-- Form.vue -->
  <template>
    <form>
      <slot></slot>
    </form>
  </template>
  
  <script>
  export default {
    name: 'wangForm',
    props: {
      model: {
        type: Object
      },
      rules: {
        type: Object
      }
    }
  }
  
  </script>
  <style>
  </style>
  <!-- FormItem.vue -->
  <template>
    <div>
      <label>{{ label }}</label>
      <div>
        <slot></slot>
        <p v-if="errMessage">{{ errMessage }}</p>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'wangFormItem',
    props: {
      label: {
        type: String
      },
      prop: {
        type: String
      }
    },
    data () {
      return {
        errMessage: ''
      }
    }
  }
  </script>
  
  <style>
  </style>
  <!-- Input.vue -->
  <template>
    <div>
      <input v-bind="$attrs" :type="type" :value="value" @input="handleInput">
    </div>
  </template>
  
  <script>
  export default {
    name: 'wangInput',
    // 禁用父组件默认属性
    inheritAttrs: false,
    props: {
      value: {
        type: String
      },
      type: {
        type: String,
        default: 'text'
      }
    },
    methods: {
      handleInput (evt) {
        this.$emit('input', evt.target.value)
      }
    }
  }
  </script>
  <style>
  </style>
  <!-- Button.vue -->
  <template>
    <div>
      <button @click="handleClick"><slot></slot></button>
    </div>
  </template>
  
  <script>
  export default {
    name: 'wangButton',
    methods: {
      handleClick (evt) {
        this.$emit('click', evt)
        evt.preventDefault()
      }
    }
  }
  </script>
  <style></style>
  ```

- 创建Form-test.vue测试组件

  ```vue
  <template>
    <wang-form class="form" ref="form" :model="user" :rules="rules">
      <wang-form-item label="用户名" prop="username">
        <!-- <wang-input v-model="user.username"></wang-input> -->
        <wang-input :value="user.username" @input="user.username=$event" placeholder="请输入用户名"></wang-input>
      </wang-form-item>
      <wang-form-item label="密码" prop="password">
        <wang-input type="password" v-model="user.password"></wang-input>
      </wang-form-item>
      <wang-form-item>
        <wang-button type="primary" @click="login">登 录</wang-button>
      </wang-form-item>
    </wang-form>
  </template>
  
  <script>
  import WangForm from './form/Form'
  import WangFormItem from './form/FormItem'
  import WangInput from './form/Input'
  import WangButton from './form/Button'
  export default {
    components: {
      WangForm,
      WangFormItem,
      WangInput,
      WangButton
    },
    data () {
      return {
        user: {
          username: '',
          password: ''
        },
        rules: {
          username: [
            {
              required: true,
              message: '请输入用户名'
            }
          ],
          password: [
            {
              required: true,
              message: '请输入密码'
            },
            {
              min: 6,
              max: 12,
              message: '请输入6-12位密码'
            }
          ]
        }
      }
    },
    methods: {
      login () {
        console.log('button')
        // this.$refs.form.validate(valid => {
        //   if (valid) {
        //     alert('验证成功')
        //   } else {
        //     alert('验证失败')
        //     return false
        //   }
        // })
      }
    }
  }
  </script>
  
  <style>
    .form {
      width: 30%;
      margin: 150px auto;
    }
  </style>
  
  ```

  `vue serve ./src/Form-test.vue`运行

- 设置表单验证

  表单验证原则：input组件中触发自定义事件validate、FormItem渲染完毕注册自定义事件validate

  修改Input.vue

  ```vue
  ...
  <script>
  export default {
    ...
    methods: {
      // input组件中触发自定义事件validate
      // FormItem渲染完毕注册自定义事件validate
      handleInput (evt) {
        this.$emit('input', evt.target.value)
        // 找input的父组件
        const findParent = parent => {
          while (parent) {
            if (parent.$options.name === 'LgFormItem') {
              break
            } else {
              parent = parent.$parent
            }
          }
          return parent
        }
        const parent = findParent(this.$parent)
        // 如果找到就触发自定义事件
        if (parent) {
          parent.$emit('validate')
        }
      }
    }
  }
  </script>
  ```

  需要在Fome.vue中将Form组件实例注册依赖

  ```vue
  provide () {
      return {
        form: this
      }
    }
  ```

  修改FormItem.vue，使用[async-validator](https://www.npmjs.com/package/async-validator)进行验证

  ```vue
  ...
  <script>
  import AsyncValidator from 'async-validator'
  export default {
    name: 'WangFormItem',
    inject: ['form'],
    ...
    mounted () {
      this.$on('validate', () => {
        this.validate()
      })
    },
    methods: {
      validate () {
        // 如果没有prop 就不需要验证
        if (!this.prop) return
        const value = this.form.model[this.prop]
        const rules = this.form.rules[this.prop]
  
        const descriptor = { [this.prop]: rules }
        const validator = new AsyncValidator(descriptor)
        return validator.validate({ [this.prop]: value }, errors => {
          if (errors) {
            this.errMessage = errors[0].message
          } else {
            this.errMessage = ''
          }
        })
      }
    }
  }
  </script>
  
  <style>
  </style>
  
  ```

  修改Form.vue，提供validate方法

  ```js
  methods: {
      validate (cb) {
        // 过滤出含有prop验证的节点 执行validate方法 返回promise对象
        const tasks = this.$children
          .filter(child => child.prop)
          .map(child => child.validate())
  
        Promise.all(tasks)
          .then(() => cb(true))
          .catch(() => cb(false))
      }
  }
  ```

- 此时重新运行，开启表单验证