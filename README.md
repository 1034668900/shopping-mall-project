## 项目注意点

### 访问接口获取数据时需要配置代理服务器

```js
  server:{
    proxy:{
      "/api": {
        target: 'http://gmall-h5-api.atguigu.cn',
        changeOrigin:true
      }

    }
  }

```
### 接口统一管理
- 在api文件夹下的index.js文件中统一管理接口，将每个接口封装成函数并暴露，后续使用时直接调用函数即可

### 进度条包 nprogress
1. 安装nprogress   ```npm i nprogress```
2. 使用：
    1. 引入nprogress
    2. 引入nprogress样式
    3. 在请求拦截器里调用```nprogress.start()```,在响应拦截器成功回调里调用```nprogress.done()```
3. start方法表示进度条开始、done方法表示进度条结束，进度条的样式可以在```nprogress.css```文件中修改

### 三级联动组件数据的动态获取
1. 在三级联动组件TypeNav中，利用mounted钩子，在组件挂载完毕时，利用$store上的dispatch方法向vuex中home模块下的actions派发任务
2. 在actions里对应的方法中提交(commit)给mutations修改state数据
3. 引入api文件里统一管理的获取三级联动数据的请求方法，在mutations中发起请求，并将返回的数据赋值给state中的categoryList
4. 在TypeNav中引入映射方法mapState映射State中的categoryList，根据categoryList的数据动态渲染HTML中相应的数据
5. **注意**：state中categoryList的初始值要根据服务器返回的数据类型来给，服务器返回的是对象初始值就为空对象{},服务器返回的是数组，初始值就应该是空数组[]

#### 一级菜单动态添加背景颜色及二三级菜单的显示隐藏
1. 通过给一级菜单的样式利用鼠标悬停添加背景样式:hover
2. 通过js实现为每个一级菜单添加背景样式
    1. 在三级联动组件TypeNav里配置一个响应式数据:currentIndex 记录鼠标进入时一级菜单的下标
    2. 给一级菜单标题所在的h3标签绑定@鼠标移入事件```mouseenter```并传入index下标
    3. 在鼠标移入事件中将index下标的值赋值给data中的currentIndex记录  ```this.currentIndex = index```
    4. 循环遍历Item项时，如果满足下标和currentIndex中记录的下标相同，就动态绑定类名```:class = "{curStyle : currentIndex == index}"  ```(在curStyle样式里设置背景颜色)
    5. 此时仅仅是鼠标进入时会有背景颜色，但是鼠标移除后背景颜色会残留，因此需要再绑定@鼠标移出事件```mouseleave```,在对应的事件函数中将currentIndex值重置```this.currentIndex = -1```
    6. 如果想要实现鼠标悬浮在“全部商品分类盒子”时第0个一级菜单有背景颜色，需要全部商品分类盒子和一级菜单Item项是兄弟元素，且在他们外边包裹一个div，将鼠标移出事件绑定到该div上即可实现(这里利用了事件委派，div内子元素触发的事件会冒泡到他们的父元素上)

3. 通过为二三级菜单动态绑定style属性实现鼠标悬停在哪个一级菜单，其对应的二三级菜单就显示```:style = "{display : currentIndex==index ? 'block' : 'none'}" ```


### 防抖和节流
#### 问题描述
- 在上述三级联动效果实现的过程中，如果鼠标快速经过多个一级菜单的话，内部的回调会被调用很多次，在这种情况下，如果内部回调业务较复杂，运算需要一定时间，就会在页面造成卡顿

#### 防抖
- 定义：某一事件短时多次触发时，前面的所有的触发都会被取消，最后一次执行在规定的时间之后才会触发，也就是说如果连续快速的触发，只会执行一次。

- 手写防抖：

```js
    // 防抖函数
    function debounce(cb,delayTime){
        // 创建存数定时器ID的变量
        let timer = null

        // 返回一个函数
        return function(){
            // 清理上一个timer
            clearTimeout(timer)
            // 在嵌套函数内使用上一层函数的变量，形成闭包
            timer = setTimeout(()=>{
                // 调用传入的回调
                cb()
            },delayTime)
        }
    }
```


#### 节流
- 定义：在规定的间隔时间范围内不会重复触发回调，只有大于这个时间间隔才会触发回调，把频繁触发变为少量触发
- 手写节流：

```js
    // 节流函数
    function throttle(cb, delayTime){
        // 设置一个节流阀(valve) 默认为false
        let valve = false

        return function (){
            // 触发事件时判断节流阀的状态，为false执行，true就不执行
            if(valve){
                return
            }else{
                // 进入此处说明valve为false，立马将其状态改变，表示打开节流阀
                valve = true
                // 开启定时器
                setTimeout(()=>{
                    // 将传入的cb回调执行,要注意此时this指向问题
                    cb.apply(this,arguments)
                    
                    // cb执行完毕后关闭节流阀
                    valve = false
                }, delayTime)
            }
        }
    }
```

### 三级联动实现路由跳转并传参
#### 利用router-link实现路由跳转并传参(不推荐)
- 当通过**声明式路由导航**将三级联动里的a标签替换为router-link后，可以实现路由跳转，并携带参数。但是此时变得非常卡顿。
  - 原因如下：
    1. router-link是一个组件，当服务器的数据返回后，会根据服务器的数据创建出许多的router-link组件实例
    2. 短时间内创建的组件实例过多时，就耗费了大量的内存资源，使得非常卡顿。
- 因此实现此处实现三级路由跳转采用**编程式路由导航**
#### 通过编程式路由导航+事件委派+自定义属性(data-开头，H5出现的)实现路由跳转并传参(最优解)
##### 编程式路由导航+事件委派存在的问题和解决方案
1. 将三级分类的a标签上绑定的跳转事件提升到他们共同的父元素上才能实现事件委派，但是该父元素内又存在非常多其它类型的标签，**如何区分父元素内触发跳转事件的对象是a标签呢？**
   - 解决方案：
     1. 通过为三级联动组件内的a标签添加自定义属性data-categoryName，后续就可以通过检查标签属性是否含有data-categoryName属性就可以判断出触发跳转事件的对象是不是a标签。
     2. 注意：**自定义属性我们自己命名时是驼峰命名的，但是浏览器接续出来后命名就全是小写了** 
2. 通过为a标签打上自定义属性后，已经可以通过该方法判断触发事件的对象是不是a标签，但是又**怎么拿到触发事件的元素以及获取到自定义属性呢？**
   - 解决方案：
     1. 通过事件对象event可以拿到触发事件的元素对象。
     2. 通过```dataset```属性可以拿到元素对象身上的自定义属性。
3. 传递参数时，每一级菜单都有自己的```categoryName```和```categoryId```,**传递参数时又该如何区分呢？**
   - 解决方案：
     - categoryName可以通过自定义属性```data-categoryName```传递，categoryId可以为每一级的a标签打上不同的自定义属性进行区分，一级菜单a标签自定义属性可命名为```data-category1Id```，二级菜单a标签自定义属性可命名为```data-category2Id```，以此类推。
   

