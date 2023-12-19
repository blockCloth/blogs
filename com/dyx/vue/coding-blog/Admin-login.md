Admin端之登录篇

### 一、登录界面

​		登录界面位于`src\views\login\login.vue`，主要包含`Card`，输入用户名、密码即可登录

```vue
<el-card class="box-card">
      <div slot="header" class="clearfix">
        <!-- <h2>CodingMore后台管理登陆</h2> -->
        <img :src="logoUrl" width="360" />
      </div>
      <el-form ref="form" :model="form" label-width="60px">
        <el-form-item label="用户名">
          <el-input v-model="form.userLogin" maxlength="30" @keydown.native.enter="btnLoginClick"></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input type="password" v-model="form.userPass" show-password maxlength="50" @keydown.native.enter="btnLoginClick"></el-input>
        </el-form-item>
        <div class="text-right">
          <el-button type="primary" @click="btnLoginClick">登陆</el-button>
        </div>
      </el-form>
    </el-card>
```

### 二、登录流程

1. 获取用户名、密码，再将其转换成一个查询字符串如果 `this.form` 是 `{ username: 'test', password: '1234' }`，那么 `qs.stringify(this.form)` 将会返回 `'username=test&password=1234'`。

2. 调用 用户提交登陆请求`UserLogin（src/api/login.js）`

3. 成功则将`token`存储到`Cookie`

### 三、响应处理

1. 封装请求数据` const { code,   result, message } = response.data`
2. 判断请求结果是否成功`code === 200 && response.status === 200`
3. 处理数据

   



