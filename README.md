## 项目重点

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
- 在api文件夹下的index.js文件中统一管理接口，将每个接口封装成函数并暴露，后续使用时直接调用函数即可。

### 进度条包 nprogress
1. 安装nprogress   ```npm i nprogress```
2. 使用：
    1. 引入nprogress
    2. 引入nprogress样式
    3. 在请求拦截器里调用```nprogress.start()```,在响应拦截器成功回调里调用```nprogress.done()```
3. start方法表示进度条开始、done方法表示进度条结束，进度条的样式可以在```nprogress.css```文件中修改

## home模块的开发

### 三级联动组件数据的动态获取
1. 在三级联动组件TypeNav中，利用mounted钩子，在组件挂载完毕时，利用$store上的dispatch方法向vuex中home模块下的actions派发任务。
2. 在actions里对应的方法中提交(commit)给mutations修改state数据。
3. 引入api文件里统一管理的获取三级联动数据的请求方法，在mutations中发起请求，并将返回的数据赋值给state中的categoryList。
4. 在TypeNav中引入映射方法mapState映射State中的categoryList，根据categoryList的数据动态渲染HTML中相应的数据。
5. **注意**：state中categoryList的初始值要根据服务器返回的数据类型来给，服务器返回的是对象初始值就为空对象{},服务器返回的是数组，初始值就应该是空数组[]

#### 一级菜单动态添加背景颜色及二三级菜单的显示隐藏
1. 通过给一级菜单的样式利用鼠标悬停添加背景样式:hover
2. 通过js实现为每个一级菜单添加背景样式：
    1. 在三级联动组件TypeNav里配置一个响应式数据:currentIndex 记录鼠标进入时一级菜单的下标。
    2. 给一级菜单标题所在的h3标签绑定@鼠标移入事件```mouseenter```并传入index下标。
    3. 在鼠标移入事件中将index下标的值赋值给data中的currentIndex记录  ```this.currentIndex = index```
    4. 循环遍历Item项时，如果满足下标和currentIndex中记录的下标相同，就动态绑定类名```:class = "{curStyle : currentIndex == index}"  ```(在curStyle样式里设置背景颜色)。
    5. 此时仅仅是鼠标进入时会有背景颜色，但是鼠标移除后背景颜色会残留，因此需要再绑定@鼠标移出事件```mouseleave```,在对应的事件函数中将currentIndex值重置```this.currentIndex = -1```。
    6. 如果想要实现鼠标悬浮在“全部商品分类盒子”时第0个一级菜单有背景颜色，需要全部商品分类盒子和一级菜单Item项是兄弟元素，且在他们外边包裹一个div，将鼠标移出事件绑定到该div上即可实现(这里利用了事件委派，div内子元素触发的事件会冒泡到他们的父元素上)。

3. 通过为二三级菜单动态绑定style属性实现鼠标悬停在哪个一级菜单，其对应的二三级菜单就显示```:style = "{display : currentIndex==index ? 'block' : 'none'}" ```。

#### 三级联动组件动态获取数据的优化
1. 存在问题：目前获取菜单数据的请求是在TypeNav组件的mounted钩子里发起的，这意味着该组件每一次销毁再创建的过程，都在不断的发起请求获取数据，会不断的消耗资源，而这个数据其实是只需要发起一次的，然后将其存储在store中，需要的时候直接从仓库获取即可。
2. 解决办法：
   1. 入口文件只会执行一次，因此将该请求放在创建Vue实例的配置对象里的mounted钩子里即可
   2. 也可以放在App根组件的mounted钩子里，该组件也只执行一次。


### 防抖和节流
#### 问题描述
- 在上述三级联动效果实现的过程中，如果鼠标快速经过多个一级菜单的话，内部的回调会被调用很多次，在这种情况下，如果内部回调业务较复杂，运算需要一定时间，就会在页面造成卡顿。

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
                cb.apply(this,arguments)
            },delayTime)
        }
    }
```


#### 节流
- 定义：在规定的间隔时间范围内不会重复触发回调，只有大于这个时间间隔才会触发回调，把频繁触发变为少量触发。
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
- 因此实现此处实现三级路由跳转采用**编程式路由导航**。
#### 通过编程式路由导航+事件委派+自定义属性(data-开头，H5出现的)实现路由跳转并传参(最优解)
##### 编程式路由导航+事件委派存在的问题和解决方案
1. 将三级分类的a标签上绑定的跳转事件提升到他们共同的父元素上才能实现事件委派，但是该父元素内又存在非常多其它类型的标签，**如何区分父元素内触发跳转事件的对象是a标签呢？**
   - 解决方案：
     1. 通过为三级联动组件内的a标签添加自定义属性data-categoryName，后续就可以通过检查标签属性是否含有data-categoryName属性就可以判断出触发跳转事件的对象是不是a标签。
     2. 注意：**自定义属性我们自己命名时是驼峰命名的，但是浏览器接续出来后命名就全是小写了**。
2. 通过为a标签打上自定义属性后，已经可以通过该方法判断触发事件的对象是不是a标签，但是又**怎么拿到触发事件的元素以及获取到自定义属性呢？**
   - 解决方案：
     1. 通过事件对象event可以拿到触发事件的元素对象。
     2. 通过```dataset```属性可以拿到元素对象身上的自定义属性。
3. 传递参数时，每一级菜单都有自己的```categoryName```和```categoryId```,**传递参数时又该如何区分呢？**
   - 解决方案：
     - categoryName可以通过自定义属性```data-categoryName```传递，categoryId可以为每一级的a标签打上不同的自定义属性进行区分，一级菜单a标签自定义属性可命名为```data-category1Id```，二级菜单a标签自定义属性可命名为```data-category2Id```，以此类推。
   

### 三级联动菜单路由跳转参数和Header组件中输入内容参数的合并问题
#### 情况一：先点击三级联动菜单跳转到search，再在search中输入内容并搜索
1. **存在问题**：Header组件中点击搜索按钮后跳转到search组件时携带的参数只有Header中的params参数，没有三级联动菜单带过来的query参数。
2. **解决办法**：在**Header组件**中**跳转search路由的函数**中**整理参数**时先**判断**当前路由中有没有query参数，有的话就加上。
3. **注意：**params参数动态传递时需要在配置路由时在路径后进行占位，且应该在其后面加上？表示该参数可有可无，避免单独传递query参数时报错。

#### 情况二：先在Header 组件中输入内容，然后再点击三级联动菜单
1. **存在问题**：此时就只有三级联动菜单路由跳转携带的参数，没有Header组件搜索框中输入的内容。
2. **解决办法**：在三级联动菜单中整体参数时先判断当前路由中有没有params参数，有的话就加上。

## search模块的开发
### 请求参数的问题
1. **存在问题**：
   1. 在向服务器请求时，要根据当前不同的参数来发送请求，根据API文档有了当前的参数结构，那么请求前的参数从哪里来？
   2. 获取到了参数，又将其存放到哪里？
   3. 当前是直接在mounted钩子里派发的actions请求，它只会执行一次，但是实际情况中需要根据用户的操作不断发送请求获取数据，这又如何解决？
2. **解决方案**：
   1. 请求的参数已经从路由跳转到Search组件时传递了过来，因此可以通过```this.$route```获取。
   2. 在Search组件中定义一个响应式数据```searchParams```专门存储参数，并根据API文档要求将其整理为对应参数格式并初始化。
   3. 利用```beforeMount```生命周期钩子可以在```Mounted```之前获取到请求参数，因此此时可以进行参数整理。
   4. 后续派发actions请求获取list数据会非常频繁，因此将其封装为一个函数，方便调用。可以首先在mounted里调用一次测试参数的获取有没有问题。
   5. 实际操作时应该每次搜索都能发送一次请求，而每次发起请求路由```$route```都会改变，因此可以通过watch监听器监听```$route```，一旦其发生改变，就发起请求，注意每次请求前都应该合并一下参数，而且每次请求结束后都应该把相应的一二三级ID清空，避免下次请求不同层级时会携带上一次的Id。

### 面包屑的实现
#### 面包屑处理一级分类名
1. **功能需求**：
   1. 点击对应关键词后，显示面包屑。
   2. 面包屑支持删除，删除后应再次发送请求。
   3. 同时地址栏的参数也应该将其清空。
2. **具体实现**：
   1. 通过```v-if```指令利用分类名```categoryName```的存在与否控制面包屑的显示与隐藏。
   2. 为删除UI绑定点击事件，点击后移除掉请求参数```searchParams```中的```categoryName```。
   3. 当```categoryName```被移除后，请求参数中的分类Id没有意义，因此在请求前也应该将其置空，但是置空的话请求时还是会携带该参数，这里可以做一个小的**优化**，将其置为**undefined**，这样请求时就不会携带这些参数
   4. 参数置为**undefined**后再次发起请求。
   5. 删除面包屑后，地址栏参数清空可以利用一个**非常巧妙**的方法，可以**利用路由器跳转到当前路由且不携带参数**，即可实现地址栏路径query参数的清空。但是应该只清除query参数，params参数应该保留，因此路由跳转时应该携带params参数。

#### 面包屑处理搜索关键字
1. **功能需求**:
   1. 移除关键字面包屑后，params参数就应该没有了，此时应该重新请求当前分类的数据进行展示，但是再次发送请求时，query参数应该携带。
   2. 同时地址栏应该更新。
   3. 同时Header组件搜索框中关键字也应该置空(兄弟组件通信--这里使用全局事件总线)。
  
2. **功能实现**：
   1. 通过```v-if```指令利用关键字```keyword```的存在与否控制面包屑的显示与隐藏。
   2. 绑定事件清空**keyword**
   3. 利用全局事件总线实现**Header组件**清空搜索栏关键字。
   4. 重新发起数据请求，同时携带query参数。

#### 面包屑处理品牌信息
 **问题分析与解决**：品牌信息组件是Search组件的一个**子组件**，目前需要实现在品牌信息里点击品牌，将对应的品牌字段生成面包屑显示，同时应该发起请求更新搜索区域的信息。**但是**发起请求的参数**searchParams**存在与品牌信息组件的父组件中，因此发送请求的地方应该在父组件中，同时子组件应该将用户点击的品牌信息传给父组件(子 ==> 父)，因此可以利用**自定义事件**实现**子组件和父组件通信**。

 #### 面包屑处理平台售卖信息和上面类似

 ## 商品列表排序
 ### 列表类名以及图标显示
 1. **功能需求**：列表里的**综合按钮**和**排序按钮**根据用户的点击相应变为红色，默认是综合按钮红色，同时两个按钮旁边都要动态显示升序和降序的图标。
 2. **实现思路**：
    1. 相应API文档提到可根据```searchParams.order```里的参数来判断显示哪个按钮以及是升序还是降序。
    2. order参数格式为 ```1: 综合,2: 价格 asc: 升序,desc: 降序``` 。
    3. 因此可以根据order参数里的升降序来显示对应的箭头图标。

## 分页器的开发
### 电商平台商品信息等为什么要分页？
1. 客户端的问题： 如果数据量太多，都显示在同一个页面的话，会因为页面太长严重影响到用户的体验，也不便于操作，也会出现加载太慢的问题。
2. 服务端的问题： 如果数据量太多，可能会造成内存溢出（OOM），而且一次请求携带的数据太多，对服务器的性能也是一个考验。

### 实现分页器需要知道哪些数据(条件)?
1. 当前是第几页：pageNo
2. 每页展示多少条数据：pageSize
3. 一共有多少条数据：total
4. 分页器连续的页码数：5|7  一般是奇数，奇数对称
5. 注意：根据上面几个条件可以计算出总的页码数totalPage  ```向上取整 totalPage = Math.ceil(total/pageSize)```

### 分页器计算连续页码开始start结尾end的注意事项：
#### 分页器计算中间连续页码时可能出现以下异常：
1. 连续页码数可能出现的异常：
   1. 当总数据total较少时，可能出现总的分页数```totalPage```小于连续页码数```continues```的情况
   2. 解决方式：这种情况让 ```start = 1  end = totalPage```

2. start可能出现的异常：
   1. start < 0 或 start == 0
   2. 解决办法： ```if(start < 1 ){ start = 1  end = totalPage} ```

3. end可能出现的异常：
   1. 结束页码end > 总页码数totalPage
   2. 解决办法： ``` end = totalPage  start = totalPage - continues + 1  ```

#### 分页器动态渲染时的注意点
1. 连续页码数要根据循环来获得，通过**循环连续页码数的end**，并**添加判断**动态生成中间需要几个连续页码(注意v-for遍历数字是index是从1开始的)。
2. 注意控制动态渲染出来的中间页码区域的的页码数得**对应**当前页码数pageNo。
3. 根据页码出现的条件，**动态调整**左侧页码和右侧页码是否显示。**比如**当中间区域动态渲染出的页码有1时1左侧页码1就不应该显示。

#### 分页器事件绑定
1. **实现思路**：当点击分页器时，应该将当前页码pageNo传递给父组件，父组件接收到参数后更新searchParams里的pageNo的值，然后再次发起请求。这里由于涉及到了**子组件向父组件传递数据**，因此可以使用**自定义事件**。
2. **注意事项**：
   1. 分页器的上一页和下一页在当前页数已经是最小或者最大时，上一页或下一页将不能点击。
   2. 分页器中传递当前页码时要注意页码的正确性。
   3. 动态为当前页码绑定类名，使其有激活项效果。

## 商品详情页开发
### 注意点：
1. 前端路由跳转时，当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。 vue-router 能做到，而且更好，它让你可以自定义路由切换时页面如何滚动。

### 售卖属性高亮实现注意点
1. **实现思路**：售卖属性中的高亮是取决于每一项的isChecked属性的，因此可以拿到整个列表，将所有的isChecked置为0，再将点击的那一项置为1(排他)。
2. **需要拿到的数据**： 当前点击项的对象，以及整个数据列表的数组。

```js
    changeActive(spuSaleAttrValue,spuSaleAttrValueList){
        // 将列表中所有的isChecked置0
        spuSaleAttrValueList.forEach(item => {
          item.isChecked = 0
        })
        // 再将选中项的isChecked置1
        spuSaleAttrValue.isChecked = 1
    }
```

### 商品数量输入框的注意点
1. **问题分析**：用户输入的数量可能出现非Number、小数、负数等，需要将这些情况考虑。
2. **解决思路**：通过```event.target.value```拿到输入框的值，将其乘1，这样如果其中的内容出现非Number，其返回值就是NaN，可以通过这个来判断输入的非法内容。

## 加入购物车的注意点
### 加入成功与否的判断
1. **情景分析**：点击加入购物车时会派发actions，在actions中借助api中封装好的函数向服务器发送请求，此时服务器会返回该次请求是成功还是失败，在商品详情组件里又如何知道此次请求是否成功呢？
2. **解决思路**：在actions中进行的是异步操作，因此使用了async和await，那么他的返回值就是一个Promise，可以在actions中进行判断后返回请求成功或失败的结果，那么在商品详情组件里也可以利用async和await进行异步操作，等待该Promise的返回结果，通过返回结果就可以知道这次请求的结果。同时需要利用**try、catch**捕获可能出现的异常。
### 加入成功后路由跳转参数的问题
1. **问题分析**：跳转成功后需要将商品详情的信息传递给添加商品成功AddCartSuccess组件进行数据的动态渲染，那么数据如何传递呢？
2. **解决办法**：
   1. 路由传参：通过路由传参可以将商品详情信息以及商品Id传递过去，但是这样地址栏就会非常混乱，不建议这样操作。
   2. 路由传参和会话存储结合：
      - 通过路由传参传递商品Id这样简单的数据，而商品详情信息这样复杂的数据通过会话存储存储到浏览器，然后在
AddCartSuccess组件中从浏览器中取对应数据即可。

### 购物车数据请求
1. **问题分析**： 
   1. 封装好请求购物车的api后，在调用该api时服务器确实返回请求成功，但是并没有数据，这是因为服务器不知道你的“身份”，不知道该返回什么数据。于是我们可以通过uuid模拟游客用户，但是，向服务器发起加入购物车时的api只能携带两个参数，那么uuid不通过api又怎么能发送给服务器呢？
   2. uuid每次调用都会生成一个不同的随机串，但是游客的临时身份肯定不能一直变，应该持久化的存储到浏览器，这又如何实现？
2. **解决方式**：
   1. 可以在请求拦截器中，发送请求时判断detail仓库中是否uuid_token，有的话就在请求头中携带uuid_token，请求头中的字段不能随意写，需和后端沟通好。
   2. 重新创建一个js文件，在这个函数中暴露一个函数，在这个函数中创建一个uuid，然后将其本地存储，每次调用该函数时都判断本地存储中是否有uuid,有的话就直接返回本地存储的uuid，没有的话就重新生成一个，这样就能保证游客的临时身份不会每次访问都发生改变了。

### 购物车中商品数量修改的问题：
1. **问题分析及解决思路**：
   1. 在购物车中修改数量时调用的还是先前向购物车中添加商品的api，因此不管是点击+号、-号还是输入框输入数量来修改商品数量，发起请求的都是同一个api，因此这三个元素身上绑定的事件的回调都是同一个，但是都是同一个回调要怎么判断点击的是哪一个？
   2. 加号和减号传递过去的值是变化量，但是中间输入框传递进去的值却是最终的量
   3. 最终发送请求时还需要商品的Id，因此点击对应事件时还需要传入相应商品的对象
2. **解决思路**：
   1. 可以通过点击时传参，根据参数的不同来判断点击的是哪一个对象。
   2. 在事件的回调中传入数值。加减传递变量值，输入框传入最终值。
   3. 同时在事件回调中传入对应的商品对象。


### 批量删除购物车中选中商品
1. **问题分析**：删除商品的接口只有单个删除的，没有批量删除的接口，因此只能一个一个的删。
   1. 因此要为删除选中商品按钮绑定一个事件，该事件触发市派发一个action，该action中的上下文对象context中可以拿到当前小仓库中的数据，并能通过context派发action给单个删除的接口去删除，因此可以遍历存储购物车商品的数组，检测其勾选状态，然后调用接口将其删除。**注意**，每次调用接口都会返回一个Promise，多次删除就有多个Promise，因此可以创建一个数组来存储每一次返回的Promise，遍历结束后调用Promise.all方法判断是否所有的Promise都执行成功，然后将其结果返回。

## 登录与注册
### 登录
#### token
1. 一般注册成功后，登录时，如果登录成功，服务器会下发一个token【令牌】，前台需要持久化的存储token，请求时需要带着token向服务器拿数据。
2. 登录成功后跳转到home组件时在mounted中派发action，在请求拦截器中判断，在请求头中携带token发送请求获取用户信息。

#### vuex
1. 登录获取到的用户token不能存储在vuex中，因为vuex中的数据并不是持久化存储的。因此浏览器一刷新，vuex中的token就不存在了，因此请求拦截器中的判断也会失效，不会携带token，因而home页面的用户信息就消失了。
2. 需要将获取到的token存储到浏览器本地，并在请求拦截器中判断是否含有token，然后再在请求头中携带token。
   
#### 登录业务目前存在问题
1. 登录成功后，只有在home页面登录状态能保持，切换其他路由后一刷新登录状态不会更新。
2. 用户登录成功后，按理说不能再进入登录路由，但目前还能。
3. **解决思路**：路由守卫(导航守卫)。

### 路由守卫处理登录权限问题
1. 用户一旦登录成功，那么将不能再跳转登录页面
2. 用户登录成功后会自动跳转到首页，此时具有用户名，那么用户跳转到其他页面时要进行判断，如果不是去的登录页面，且具有用户名则放行，否则重新派发action获取用户名再放行

13700000000   111111 
## 支付注意事项
1. 用户点击立即支付后会弹出支付二维码，需要不断查询用户订单的支付状态，利用延时器setInterval。
2. 如果用户支付成功，需要保存支付成功的code，用于判断弹出框的点击事件，弹出框用的ElementUI库。

## 个人中心单元格合并问题
- 每个订单可能多多个商品，因此在表单内部会有多个，但是订单的信息只需要有一个就够了，因此纵向的单元格合并的rowspan不能写死了，要和当前列表的商品数量一样，然后后面的订单信息的订单渲染只要第一个就好了，可以利用v-if对索引进行判断。```v-if = " index == 0"```。
- 使用自己封装的分页器


## 完善全局守卫未登录的权限控制
1.
  1. 用户没有登陆：
          1. 不能去个人中心
          2. 不能去交易相关的
          3. 不能去支付相关的
  2. 当未登录用户想跳转到以上页面时，应该重定向至登录页。
  3. 去的不是以上页面时，放行。

2. 用户开始没有登陆，点击了上述相关的页面，然后用户跳转到了登录页面，等登录成功后，用户应该跳转到先前点的路由。
   1. **实现思路**：
      1. 在用户放行至登录页面时添加一个query参数，动态携带用户想要跳转的路径，将其传递给login组件，``` next('/login?redirect=' + toPath)```。
      2. 然后在login组件中检测是否含有该参数，如果不包含，正常跳转到home，如果包含，则跳转到toPath路径的页面。

## 完善全局守卫已登录的权限控制
1. 用户即使已经成功登录了，但是支付成功的页面用户不能直接去。用户想跳转到交易相关页面，只有从结算页面才能跳转。
2. **解决思路**：
   1. 在**全局守卫**中判断
   2. 利用 **路由独享守卫** 也可以解决该问题
   


## 图片懒加载
- 利用vue-lazyload库可快速实现图片懒加载

## 表单验证
### 表单验证插件配置

```js
// VeeValidate插件表单验证区域
import Vue from 'vue'
import VeeValidate from 'vee-validate'
// 引入中文
import zh_CN from 'vee-validate/dist/locale/zh_CN'
Vue.use(VeeValidate)

// 表单验证
VeeValidate.Validator.localize('zh_CN',{
    messages:{
        ...zh_CN.messages,
        is: (failed) => `${failed}必须与密码相同` // 修改内置规则的message
    },
    attributes:{
      // 将每个提示字段转换成中文
        phone: '手机号',
        code: '验证码',
        password: '密码',
        password: '确认密码',
        agree: '协议'
    }
})

// 自定义校验规则
VeeValidate.Validator.extend('agree',{
    validate: value => {
        return value
    },
    getMessage: failed => failed + '必须同意'
})

```

### 表单验证插件的使用

```HTML
   <label>手机号:</label>
   <input v-model="phone"
   placeholder="请输入你的手机号"
   name="phone"
   v-validate="{required:true , regex: /^1\d{10}$/}"
   :class="{invalid: errors.has('phone')}"
      />
   <span class="error-msg">{{errors.first('phone')}}</span>

   <!-- 确认密码的校验有点区别 -->
   v-validate="{required:true, is:password}"
```

### 在所有的表单验证都通过后再允许注册

```js
   // 等待所有的表单验证成功，返回布尔值
   const success = await this.$validator.validateAll();

```

## 路由懒加载
- 当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后**当路由被访问的时候才加载对应组件**，这样就会更加高效。(因此后续路由都用**路由懒加载**)

```js
   // 将
   // import UserDetails from './views/UserDetails.vue'
   // 替换成
   const UserDetails = () => import('./views/UserDetails.vue')

   const router = createRouter({
   // ...
   routes: [{ path: '/users/:id', component: UserDetails }],
   })
```



