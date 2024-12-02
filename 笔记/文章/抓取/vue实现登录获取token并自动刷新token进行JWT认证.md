
[vue实现登录获取token并自动刷新token进行JWT认证 (qq.com)](https://mp.weixin.qq.com/s?__biz=MjM5NDMwMjEwMg==&mid=2451851874&idx=1&sn=0f641cec474cd7c827b84b195c6be885&chksm=b0cb0525aeea5a751a673a5275bef56b3e18d84a02c4486fc33ecee18f2d24617898836d0921&mpshare=1&scene=23&srcid=1129rlofY0vs4RrnDoMQRXaK&sharer_shareinfo=e14062bb300e3b42c87bc4045547a185&sharer_shareinfo_first=e14062bb300e3b42c87bc4045547a185#rd)



![](Pasted%20image%2020241129141717.png)

## 一、账号密码登录获取JWT

通过Login.vue实现登录的用户名、密码表单信息收集，调用getToken()方法进行鉴权验证并获取jwt的token。  
Login.vue

```vue
<template>  
    <div class="body">  
        <el-form :model="form" :rules="rules" ref="loginForm" class="loginContainer">  
            <h3 class="loginTitle">  
            欢迎登录  
            </h3>  
            <el-form-item label="用户名" prop="username" :label-width="formLabelWidth">   
                <el-input type="text" v-model="form.username" placeholder="请输入用户名"></el-input>  
            </el-form-item>  
            <el-form-item label="密码"  prop="password" :label-width="formLabelWidth">   
                <el-input type="password" v-model="form.password" placeholder="请输入密码"></el-input>  
            </el-form-item>  
            <el-form-item :label-width="formLabelWidth">   
                <el-button type="primary" :plain="true" @click="submitForm('form')">登录</el-button>  
            </el-form-item>  
        </el-form>  
    </div>  
</template>  
  
<script>  
import { useUserStore } from '@/stores/user'  
import { ElMessage } from 'element-plus';  
import {getToken} from '../api/user'  
export default {  
  data() {  
    return {  
      form: {  
        username: '',  
        password: '',  
        err_username: "",  
        err_password: "",  
      },  
      rules: {  
        username: [  
          { required: true, message: '请输入用户名', trigger: 'blur' }  
        ],  
        password: [  
          { required: true, min:6, message: '请输入密码', trigger: 'blur' }  
        ]  
      },       
      formLabelWidth: '120px'  
    };  
  },  
  methods: {  
    submitForm(formName) {  
        if (this.$refs.loginForm) {  
            this.$refs.loginForm.validate(valid => {  
            if (valid) {  
                // 提交表单逻辑  
                console.log('提交成功:', this.form);  
                this.login();  
            } else {  
                console.log('验证失败');  
                ElMessage.error('验证失败，请检查您的输入！');  
            }  
            });  
      } else {  
        console.error('表单未找到');  
      }  
    },  
    login() {  
      var that = this;  
      this.message = "";  
      // 用户名密码鉴权获取jwt的token  
      getToken({  
        'username': this.form.username,  
        'password': this.form.password,  
      }).then((Response) => {  
          console.log(Response);  
          if (Response && Response.access) {  
            // //保存数据到本地存储  
            this.username= that.form.username;  
            useUserStore().login(this.username,Response.access,Response.refresh)  
            this.username = "";  
            this.password = "";  
            this.$router.push({name:"home"}); //跳转到首页  
          }  
        })  
        .catch(function (error) {  
          console.log(error);  
          if ("username" in error) {  
            that.err_username = error.username[0];  
          } else if ("password" in error) {  
            that.err_password = error.password[0];  
          } else {  
            ElMessage.error('登录失败！');  
          }  
        });  
    },  
  
  }  
};  
</script>  
  
<style scoped>  
  .loginContainer{  
        border-radius: 15px;  
        background-clip: padding-box;  
        text-align: left;  
        margin: auto;  
        margin-top: 180px;  
        width: 450px;  
        padding: 15px 35px 15px 35px;  
        background: aliceblue;  
        border:1px solid blueviolet;  
        box-shadow: 0 0 25px #f885ff;  
    }  
    .loginTitle{  
        margin: 0px auto 48px auto;  
        text-align: center;  
        font-size: 40px;  
    }  
    .loginRemember{  
        text-align: left;  
        margin: 0px 0px 15px 0px;  
    }  
    .loginbody{  
        width: 100vw;  
        height: 100vh;  
        background-size:100%;  
        overflow: hidden;  
    }  
</style>
```

在登录时调用getToken()方法获取jwt的token  
getToken的封装方法如下：

|                                                           |                                                                                                                                                                                                       |
| --------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9 | import request from '@/utils/request'  <br>  <br>export function getToken(data) {  <br>    return request({  <br>      url: 'token/',  <br>      method: 'post',  <br>      data  <br>    })  <br>  } |

通过用户名和密码鉴权可以获得JWT的token，接口会返回access的token和refresh的token，需要将这两个token保存下来，access的token用来进行API接口的jwt认证，refresh的token用来刷新失效的access的token。

## 二、将JWT保存至本地

通过pinia将token保存至浏览器的本地存储，以便于后面请求API时带上访问的token

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35  <br>36  <br>37  <br>38  <br>39  <br>40  <br>41  <br>42  <br>43  <br>44  <br>45  <br>46  <br>47  <br>48  <br>49  <br>50  <br>51  <br>52  <br>53  <br>54|import { defineStore } from 'pinia'  <br>import { refreshToken } from '../api/user'  <br>  <br>export const useUserStore = defineStore('user', {  <br>    persist: {  <br>        enabled: true, //开启数据持久化  <br>        strategies: [  <br>            {  <br>                key: "userState", //给一个要保存的名称  <br>                storage: localStorage, //sessionStorage / localStorage 存储方式  <br>            },  <br>        ],  <br>    },  <br>    state: () => ({  <br>        isLoggedIn: false,  <br>        username: '',  <br>        jwtAccessToken: null,  <br>        jwtRefreshToken: null,  <br>    }),  <br>    actions: {  <br>        login(username, accessToken,refreshToken) {  <br>            this.username = username  <br>            this.isLoggedIn = true  <br>            this.setToken(accessToken, refreshToken)  <br>        },  <br>        logout() {  <br>            this.username = ''  <br>            this.jwtAccessToken = null  <br>            this.isLoggedIn = false  <br>        },  <br>        setToken(accessToken, refreshToken) {  <br>            this.jwtAccessToken = accessToken  <br>            this.jwtRefreshToken = refreshToken  <br>        },  <br>        refreshToken() {  <br>            return new Promise((resolve, reject) => {  <br>                refreshToken({"refresh":this.jwtRefreshToken}).then((response) => {  <br>                    this.setToken(response.access, this.jwtRefreshToken)  <br>                    resolve(response.access)  <br>                    console.log('return refreshToken-----------'+response.access)  <br>                }).catch((error) => {  <br>                    reject(error)  <br>                })  <br>            })  <br>        }  <br>  <br>    },  <br>    getters: {  <br>        getIsLoggedIn: (state) => state.isLoggedIn,  <br>        getUsername: (state) => state.username,  <br>        getUserAccessToken: (state) => state.jwtAccessToken,  <br>        getRefreshToken: (state) => state.jwtRefreshToken,  <br>    }  <br>})|

在登录的Login.vue组件中调用

`useUserStore().login(this.username,Response.access,Response.refresh)`将用户名、access的token、refresh的token保存至浏览器的本地存储。

## 三、请求API带上JWT

将axios的调用封装成request.js在调用API接口时带上JWT

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35|import axios from 'axios'  <br>import Router from '@/components/tools/Router'  <br>import { useUserStore } from '@/stores/user'  <br>import { ElMessage } from 'element-plus'  <br>import { refreshToken } from '../api/user'  <br>  <br>const api_rul = import.meta.env.VITE_APP_API_URL  <br>  <br>// create an axios instance  <br>const service = axios.create({  <br>    baseURL: api_rul,  <br>    timeout: 5000, // request timeout  <br>})  <br>  <br>// request interceptor  <br>service.interceptors.request.use(  <br>    config => {  <br>        // do something before request is sent  <br>        const { url } = config  <br>        // 指定页面访问需要JWT认证。  <br>        if (url.indexOf('/login')!== -1) {  <br>            return config  <br>        }  <br>        let jwt = useUserStore().getUserAccessToken  <br>        config.headers.Authorization = `Bearer ${jwt}`  <br>        return config  <br>    },  <br>    error => {  <br>        // do something with request error  <br>        console.log(error) // for debug  <br>        return Promise.reject(error)  <br>    }  <br>)  <br>  <br>export default service|

主要是在请求头重带着jwt的信息

|   |   |
|---|---|
|1  <br>2|let jwt = useUserStore().getUserAccessToken  <br>config.headers.Authorization = `Bearer ${jwt}`|

## 四、在token失效时自动重新获取token

前面提到JWT基于安全考虑有两个token，一个是access token ,一个是refresh token 。access token的失效时间较短，可以有效降低泄露而造成的影响，两个token的区别和作用如下：

||access token|refresh token|
|---|---|---|
|有效时间|较短(如半小时)|较长(如一天)|
|作用|鉴权验证|重新获取access token|
|什么时候使用|每次接口鉴权验证时|access token失效时使用|

使用refresh token的逻辑如下：  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/6cNzMD9ovmNe92tDsOW1dXibyJQffAVbYmbIAISQmhibvu8Tiao8BzKO3kE5fEPNVheYCGkcq1PeCsxCJicIeC2Zgg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

以下通过拦截器实现token失效后重新获取access token

|   |   |
|---|---|
|1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9  <br>10  <br>11  <br>12  <br>13  <br>14  <br>15  <br>16  <br>17  <br>18  <br>19  <br>20  <br>21  <br>22  <br>23  <br>24  <br>25  <br>26  <br>27  <br>28  <br>29  <br>30  <br>31  <br>32  <br>33  <br>34  <br>35  <br>36  <br>37  <br>38  <br>39  <br>40  <br>41  <br>42  <br>43  <br>44  <br>45  <br>46  <br>47  <br>48  <br>49  <br>50  <br>51  <br>52  <br>53  <br>54  <br>55  <br>56  <br>57  <br>58  <br>59  <br>60  <br>61  <br>62  <br>63  <br>64  <br>65  <br>66  <br>67  <br>68  <br>69  <br>70  <br>71  <br>72  <br>73  <br>74  <br>75  <br>76  <br>77|import axios from 'axios'  <br>import Router from '@/components/tools/Router'  <br>import { useUserStore } from '@/stores/user'  <br>import { ElMessage } from 'element-plus'  <br>import { refreshToken } from '../api/user'  <br>  <br>const api_rul = import.meta.env.VITE_APP_API_URL  <br>  <br>// create an axios instance  <br>const service = axios.create({  <br>    baseURL: api_rul,  <br>    timeout: 5000, // request timeout  <br>})  <br>  <br>// request interceptor  <br>service.interceptors.request.use(  <br>    config => {  <br>        // do something before request is sent  <br>        const { url } = config  <br>        // 指定页面访问需要JWT认证。  <br>        if (url.indexOf('/login')!== -1) {  <br>            return config  <br>        }  <br>        let jwt = useUserStore().getUserAccessToken  <br>        config.headers.Authorization = `Bearer ${jwt}`  <br>        return config  <br>    },  <br>    error => {  <br>        // do something with request error  <br>        console.log(error) // for debug  <br>        return Promise.reject(error)  <br>    }  <br>)  <br>  <br>// response interceptor  <br>service.interceptors.response.use(  <br>   <br>    response => {  <br>        const res = response.data  <br>        return res  <br>    },  <br>    async error => {  <br>        console.log('err' + error) // for debug  <br>        const originalRequest = error.config;  <br>        // 授权验证失败  <br>        if (error.response.status === 401 && originalRequest._retry!== true) {  <br>            originalRequest._retry = true;  <br>            // 刷新token  <br>            let jwtRefreshToken=useUserStore().getRefreshToken  <br>            await refreshToken({"refresh":jwtRefreshToken}).then((response) => {  <br>                // 刷新token成功，重新请求  <br>                let jwtToken=response.access  <br>                useUserStore().setToken(jwtToken, jwtRefreshToken)  <br>                console.log('return refreshToken-----------'+response.access)  <br>                originalRequest.headers.Authorization = `Bearer ${jwtToken}`  <br>                return service(originalRequest)                  <br>            }).catch((error) => {  <br>                // 刷新token失败，跳转到登录页面  <br>                ElMessage.error('请重新登录！')  <br>                Router.push({name:'login'})  <br>            })  <br>  <br>        }  <br>        // 内部错误  <br>        if (error.response.status === 500) {  <br>            let errormsg=error.response.data.msg  <br>            ElMessage.error('服务器内部错误！'+errormsg)  <br>        }  <br>        if (error.response.status === 400)  <br>        {  <br>            ElMessage.error('错误的请求！')  <br>        }  <br>        return Promise.reject(error)  <br>    }  <br>)  <br>  <br>export default service|

在判断`error.response.status === 401`时调用refreshToken重新获取jwttoken进行接口的调用。