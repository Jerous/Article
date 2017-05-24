# Vuex modules 使用说明
## 索引
- [Modules - 基础使用](#modules)
- [Split - 拆分模块](#split)
- [局部状态](#局部状态)
- [命名空间](#命名空间)
- [使用规范](#使用规范)

## Modules
##### [返回索引](#索引)
**基础**
> 在未拆分的情况下，使用 modules 属性

在 `./main.js` 中定义modules
``` javascript
import Vue from 'vue'
import Vuex from 'vuex'
import App from './App'

Vue.use(Vuex)

// defined modules
const ModuleA = {
    state: {
        name: '木子七'
    }
}
const ModuleB = {
    state: {
        name: '王丫丫'
    }
}

// use modules
const store = new Vuex.Store({
    state: {
        rootState: 'i am rootState'
    },
    modules: {
        a: ModuleA,
        b: ModuleB
    }
})

// init
new Vue({
  el: '#app',
  router,
  store,
  template: '<App/>',
  components: { App }
})
```
<br/>

如果一切运行正常，在 `vue-devtools` 中可以看到：

![vue-devtools](./images/modules-state.png)

🦊 可以看到，modules 中的 state 与根节点的 state 唯一的区别就是，modules 中的 state 被封装到一个对象里面，只要我们知道该对象的 key，就能访问到它。
``` javascript
import { mapState } from 'vuex'
export default {
    computed: {
        ...mapState([
            'rootState',
            'a',
            'b'
        ])
    }
}
```

<br/>

## `mutations actions getters`
这些属性并不会像 modules 中的 state 一样，被一个对象所包裹；\
事实上每一个模块中的这些属性，都是暴露出来的，所以他们的 **命名需要唯一**。\
如下实例：

`./main.js`
``` javascript
import Vue from 'vue'
import Vuex from 'vuex'
import App from './App'

Vue.use(Vuex)

const ModuleA = {
  state: {
    count: 0
  },
  getters: {
    // @param [state]     该参数是局部的，访问的是 module 中的状态
    // @param [rootState] 该参数访问的是根节点的 state 状态
    countFilterA (state, getters, rootState) {
      if (state.count > 5) {
        return state.count
      }
    }
  },
  mutations: {
    increaseA (state) {
      state.count ++
    }
  },
  actions: {
    increaseAsyncA ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increaseA')
      }
    }
  }
}

const ModuleB = {
  state: {
    count: 0
  },
  getters: {
    countFilterB (state, getters, rootState) {
      if (state.count > 5) {
        return state.count + rootState.count
      }
    }
  },
  mutations: {
    increaseB (state) {
      state.count ++
    }
  }
}

// 根节点
const store = new Vuex.Store({
  state: {
    count: 10
  },
  modules: {
    a: ModuleA,
    b: ModuleB
  }
})

// init
new Vue({
  el: '#app',
  store,
  template: '<App/>',
  components: { App }
})
```

`./App.vue`
``` html
<template>
    <div class="contianer">
        {{countFilterA}}
        {{countFilterB}}
        <p>module-a: <button @click="increaseA">{{a.count}}</button></p>
        <p>module-b: <button @click="increaseB">{{b.count}}</button></p>
        <button @click="increaseAsyncA">actions</button>
    </div>
</template>

<script>
import { mapState } from 'vuex'
import { mapGetters } from 'vuex'
import { mapMutations } from 'vuex'
import { mapActions } from 'vuex'

export default {
    computed: {
        // state
        ...mapState([
            'a',
            'b'
        ]),
        // getters
        ...mapGetters([
            'countFilterA',
            'countFilterB'
        ])
    },
    methods: {
        // mutations
        ...mapMutations([
            'increaseA',
            'increaseB'
        ]),
        // actions
        ...mapActions([
            'increaseAsyncA'
        ])
    }
}
</script>
```

## split
##### [返回索引](#索引)

> vuex 的模块拆分没有想象中的那么难，只要掌握 ES6 中 [Modules](http://www.infoq.com/cn/articles/es6-in-depth-modules) 相关的知识，就很容易理解。

### 1.项目结构
```
|-demo
  |--build
  |--config
  |--node_modules
  |--src
  |  |--assets
  |  |--components
  |  |--store
  |  |  |--actions 
  |  |  |  |--aAction.js
  |  |  |  |--bAction.js
  |  |  |  |--cAction.js
  |  |  |--constants
  |  |  |  |--types.js
  |  |  |--getters
  |  |  |  |--aGetter.js
  |  |  |--modules
  |  |  |  |--aModules.js
  |  |  |  |--bModules.js
  |  |  |  |--cModules.js
  |  |  |--mutations
  |  |  |  |--aMutation.js
  |  |  |  |--bMutation.js
  |  |  |  |--cMutation.js
  |  |  |--index.js
  |  |--App.vue
  |  |--main.js
  |--static
  |--utils
  |--test
  |--index.html
```
- `contstants/types.js`
    - 为了避免变量名冲突，这个 js 文件是用来存储一些 mutations 需要使用到的方法名称。
- `actions/aAction.js`
    - 所有的 actions 全部放在这个文件夹里。
    - 我们可以使用 actions 异步请求数据，再使用 mutations 将请求到的数据提交给 state。
- `mutations/aMutation.js`
    - 所有的 mutations 全部放在这个文件夹里。
    - mutations 负责提交修改对应的 modules 中 state 的值。
- `getters/aGetter.js`
    - 所有的 getters 圈全部放在这个文件夹里。
- `modules/aModules.js`
    - 这就是一个模块，所有的 `mutations actions getters` 都会作为变量导入到这里，组装成一个完整的模块。
- `index.js`
    - 出口文件，所有的模版汇集到这里，在根节点下组装，再导出到 main.js 中。
- `../main.js`
    - 在 main.js 中引入 store，完成整个模块制作。

### 2.type.js
> 这个文件负责定义 mutations 的方法名称。\
为避免命名冲突，需按照以下格式命名：\
> `event_moduleName_state`

*constants/types.js*
``` javascript
export const FETCH_TOPICS_REQ = 'FETCH_TOPICS_REQ'
export const FETCH_TOPICS_SUC = 'FETCH_TOPICS_SUC'
export const FETCH_TOPICS_ERR = 'FETCH_TOPICS_ERR'
// ...
```

### 3.actions && mutations
> `actions` 与 `mutations` 一般配合使用\
`actions` 是负责处理异步方法的，所以用它来完成 ajax 请求；\
`mutations` 是负责处理同步方法的，并且改变 state 的值的唯一方式就是使用 `mutations` 提交；\
所以， 当 `actions` 完成请求获取数据后，再使用 `mutations` 提交给 state。

*actions/topics.js*
``` javascript
// 引入 type.js 中定义的方法名称
import * as types from '../constants/types'
import axios from 'axios'

// 所有的 actions 都写在 topicsActions 对象中
// 格式：moduleNameActions
export const topicsActions = {
    // 请求 topics 的方法
    fetchTopicsActions({ commit, state }, param) {
        commit(types.FETCH_TOPICS_REQ);
        axios({
            method: 'get',
            url: 'topics'
        }).then((res) => {
            commit(types.FETCH_TOPICS_SUC, {
                data: res.data.data
            })
        }).catch((err) => {
            commit(types.FETCH_TOPICS_ERR, {
                error: err
            });
            console.log(err)
        });
    }
}
```

*mutations/topic.js*
``` javascript
// 引入 type.js 中定义的方法名称
import * as types from '../constants/types'

// 所有的 mutations 都写在 topicsMutations 对象中
// 格式：moduleNameMutations
export const topicMutations = {
    [types.FETCH_TOPICS_REQ](state) {
        state.isFetching = true
    },
    [types.FETCH_TOPICS_SUC](state, action) {
        state.isFetching = false;
        state.data = action.data
    },
    [types.FETCH_TOPICS_ERR](state, action) {
        state.isFetching = false;
        state.error = action.error
    }
}
```
### 4.getters
*getters/topic.js*
``` javascript
// 所有的 getters 都写在 topicsGetters 对象中
// 同样，getters 也需要保证命名唯一
// 格式：moduleNameGetters
import * a types from '../constants/types'
export const topicsGetters = {
    [types.getDataLen] (state) {
        return state.data.length
    }
}
```

### 5.modules
*modules/topic.js*
> 这里才是模块的主体，上面定义的 mutations actions getters 等文件都将作为变量引入到这里。\
在这里定义模块的 state 值；\
如果你不想将 getters 拆分出去，那可以单独在这里写 getter。以此类推。
``` javascript
// 将定义好的 mutations actions getters 引入
import { topicMutations } from '../mutations/topics'
import { topicsActions } from '../actions/topics'
import { topicsGetters } from '../getters/topics'

// 组装一个模块
const topics = {
    state: {
        isFetching: false,
        data: []
    },
    getters: topicsGetters,
    mutations: topicMutations,
    actions: topicsActions
} 

// 导出该模块
export default topics
```

### 6.index.js
> index.js 是整个 modules 的出口文件；\
所有模块都会汇集到这里，在根节点下组装，并导出到 main.js 文件中去

*index.js*
``` javascript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)

// 引入 modules
import topics from './modules/topics'
// import xxx from 'xxx'
// ...

// 根节点 store
// 始终保持模块名称与导入名称一致
// [topics: topics]
const store = new Vuex.Store({
    state: {
        rootState: 'this is root state!'
    },
    modules: {
        topics
    }
})

export default store
```

### 7.main.js
> main.js 是整个项目的出口文件。\
在这个项目中引入 store，完成 vuex modules 的封装

*../main.js*
``` javascript
// ...
import store from './store'

new Vue({
    // ...
    store,
    // ...
})
```

## 局部状态
##### [返回索引](#索引)

对于模块内部的 mutation 和 getter，接收的第一个参数是**模块的局部状态**。
``` javascript
const moduleA = {
  state: { count: 0 },
  mutations: {
    increment (state) {
      // state 模块的局部状态
      state.count++
    }
  },

  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}
```

同样，对于模块内部的 action，**context.state** 是局部状态，根节点的状态是 **context.rootState**:
``` javascript
const moduleA = {
  // ...
  actions: {
    action ({ state, rootState, getters, commit, dispatch }) {
      // state 是该模块的局部状态
      console.log(state)

      // rootState 是该模块的根级状态
      console.log(rootState)

      // getters 是该模块局部的getters
      console.log(getters)

      // commit 使用该方法提交一个 mutations
      // 该 mutations 可以是任何一个模块中的 mutations
      commit('mutation')

      // dispatch 使用该方法分发一个 actions
      // 该 actions 可以是任何一个模块中的 actions
      dispatch('action', {
          msg: 'test'
      })      
    }
  }
}
```

对于模块内部的 getter，根节点状态会作为第三个参数：
``` javascript
const moduleA = {
  // ...
  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
```

## 命名空间
##### [返回索引](#索引)

> 模块内部的 action、mutation、和 getter 现在仍然注册在全局命名空间——这样保证了多个模块能够响应同一 mutation 或 action。\
你可以通过添加前缀或后缀的方式隔离各模块，以避免名称冲突。你也可能希望写出一个可复用的模块，其使用环境不可控。例如，我们想创建一个 todos 模块：

``` javascript
// types.js

// 定义 getter、action、和 mutation 的名称为常量，以模块名 `todos` 为前缀
export const DONE_COUNT = 'todos/DONE_COUNT'
export const FETCH_ALL = 'todos/FETCH_ALL'
export const TOGGLE_DONE = 'todos/TOGGLE_DONE'

// modules/todos.js
import * as types from '../types'

// 使用添加了前缀的名称定义 getter、action 和 mutation
const todosModule = {
  state: { todos: [] },

  getters: {
    [types.DONE_COUNT] (state) {
      // ...
    }
  },

  actions: {
    [types.FETCH_ALL] (context, payload) {
      // ...
    }
  },

  mutations: {
    [types.TOGGLE_DONE] (state, payload) {
      // ...
    }
  }
}
```

## 使用规范
##### [返回索引](#索引)

- 不要使用 ``TOGGLE`` 的方式来切换状态，这样会为调试带来很大的困扰；
- 所有公共状态放在一个模块中（common.js）。如：dialog, tip, snack, nav, loading ...
  - 在该模块的局部 state 中，每一个组件应该有一个属于自己的对象；
  ``` javascript
  // modules/common.js
  const common = {
      state: {
          dialog: {
              isShow: false,
              // ...
          },
          nav: {
              isShow: false,
              // ...
          },
          // ......
      }
  }
  ```