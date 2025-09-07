<template>
  <div id="signUpPage" class="bg-yellow">
    <div class="conatiner signUpPage vhContainer">
      <div class="side">
        <a href="#"><img class="logoImg" src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/logo.png" alt=""></a>
        <img class="d-m-n" src="https://raw.githubusercontent.com/hexschool/2022-web-layout-training/main/todolist/img.png" alt="workImg">
      </div>
        <!-- sign up -->
        <div v-if="mode === 'signup'">
          <form class="formControls">
            <h2 class="formControls_txt">註冊帳號</h2>
            <label class="formControls_label" for="email">Email</label>
            <input class="formControls_input" type="text" id="email" name="email" placeholder="請輸入 email" v-model="signupInfo.email" required>
            <label class="formControls_label" for="name">您的暱稱</label>
            <input class="formControls_input" type="text" name="name" id="name" placeholder="請輸入您的暱稱" v-model="signupInfo.nickname">
            <label class="formControls_label" for="pwd">密碼</label>
            <input class="formControls_input" type="password" name="pwd" id="pwd" placeholder="請輸入密碼" v-model="signupInfo.password" required>
            <label class="formControls_label" for="pwd">再次輸入密碼</label>
            <input class="formControls_input" type="password" name="pwd" id="pwdCheck" placeholder="請再次輸入密碼" v-model="signupInfo.passwordCheck" required>
            <input class="formControls_btnSubmit" type="button" value="註冊帳號" @click="signupAction">
            <button class="formControls_btnLink" type="button" @click="mode='login'">登入</button>
          </form>
        </div>

        <!-- login_page -->
        <div v-else>
          <form class="formControls">
            <h2 class="formControls_txt">最實用的線上代辦事項服務</h2>
            <label class="formControls_label" for="email">Email</label>
            <input class="formControls_input" type="text" id="email" name="email" placeholder="請輸入 email" v-model="loginInfo.email" required>
            <span>此欄位不可留空</span>
            <label class="formControls_label" for="pwd">密碼</label>
            <input class="formControls_input" type="password" name="pwd" id="pwd" placeholder="請輸入密碼" v-model="loginInfo.password" required>
            <input class="formControls_btnSubmit" type="button" value="登入" @click="loginAction">
            <button class="formControls_btnLink" type="button" @click="mode='signup'">註冊帳號</button>
          </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref, onMounted} from 'vue'
import { useRouter } from 'vue-router'
import axios from "axios";

const api     = 'https://todolist-api.hexschool.io';
const router  = useRouter();
const mode    = ref('signup');

const signupInfo = ref({
  email         :'',
  nickname      :'',
  password      :'',
  passwordCheck :''
})

const loginInfo = ref({
  email    :'',
  password :''
})

const toDoInfo = ref({
    user  :'',
    token :''
})

const signupAction = async()=>{
  if(Object.values(signupInfo.value).every(val => val !='')){
      if(signupInfo.value.password == signupInfo.value.passwordCheck){
          try{
            const res = await axios.post(`${api}/users/sign_up`,signupInfo.value);
            
            if(res.data.status)
            {
              cleanInfo(signupInfo);
              alert('success signup') 
            }
          }
          catch(err){
            console.log(err);
          }
      }
  }
}

const loginAction = async()=>{
  if(Object.values(loginInfo.value).every(val => val !='')){
      try{
        const res = await axios.post(`${api}/users/sign_in`,loginInfo.value);
        
        if(res.data.status) 
        {
            document.cookie = `todo_token=${res.data.token}; path=/; max-age=${60 * 60 * 24 * 7}`;
            await fetchMe();
            cleanInfo(loginInfo);
            alert('success sign in');
            router.push('/todo');
        }
      }
      catch(err){
        console.log(err);
      }
  }
}

const cleanInfo = (target)=>{
    target.value = Object.fromEntries(
        Object.keys(target.value).map(key => [key, ''])
    );
};

const fetchMe = async ()=>{
    const token = document.cookie.replace(/(?:^|.*;\s*)todo_token\s*=\s*([^;]*).*$/i,"$1");
    if (!token) throw new Error('no token');
    if(token != '')
    {
        const { data } = await axios.get(`${api}/users/checkout`, {
            headers: { Authorization: token }
        });
        toDoInfo.value.id = data.uid;
        toDoInfo.value.user = data.nickname;
    }
}

onMounted(async ()=>{
  try {
    await fetchMe()
    mode.value = 'signup'
    router.push('/todo');
  } 
  catch (err) {
    mode.value = 'signup'
  }
})
</script>

<style scoped>
.side {
  width: 386px;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  -ms-flex-direction: column;
  flex-direction: column;
  -webkit-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
}
.formControls {
  margin-left: 100px;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  -ms-flex-direction: column;
  flex-direction: column;
}
.formControls .formControls_txt {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 24px;
}
.formControls .formControls_label {
  font-size: 0.875rem;
  font-weight: bold;
  margin: 16px 0 4px 0;
}
.formControls .formControls_input {
  font-weight: normal;
  border: none;
  border-radius: 10px;
  width: 304px;
  padding: 12px 16px;
  margin: 4px 0;
}
.formControls .formControls_input:focus {
  outline: 3px solid #fff;
}
.formControls .formControls_input::-webkit-input-placeholder {
  color: #9F9A91;
}
.formControls .formControls_input:-ms-input-placeholder {
  color: #9F9A91;
}
.formControls .formControls_input::-ms-input-placeholder {
  color: #9F9A91;
}
.formControls .formControls_input::placeholder {
  color: #9F9A91;
}
.formControls .formControls_btnSubmit {
  width: 128px;
  height: 48px;
  border: none;
  border-radius: 10px;
  background: #333333;
  color: #fff;
  -ms-flex-item-align: center;
  -ms-grid-row-align: center;
  align-self: center;
  margin: 24px 0;
  font-weight: bold;
  cursor: pointer;
  text-align: center;
  font-size: 1rem;
}
.formControls a {
  text-decoration: none;
}
.formControls span {
  margin: 4px 0 16px 0;
  color: #d87355;
  font-size: 0.875rem;
}
.formControls .formControls_btnLink {
  width: 128px;
  height: 48px;
  border: none;
  border-radius: 10px;
  display: block;
  color: #333333;
  font-weight: bold;
  text-decoration: none;
  align-self: center;
  cursor: pointer;
}
.loginPage {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: justify;
  -ms-flex-pack: justify;
  justify-content: space-between;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  width: 800px;
}
.signUpPage {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: justify;
  -ms-flex-pack: justify;
  justify-content: space-between;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  width: 800px;
}
</style>
