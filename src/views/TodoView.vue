<template>
  <!-- ToDo List -->
  <div id="todoListPage" class="bg-half">
    <nav>
      <h1><a href="#">ONLINE TODO LIST</a></h1>
      <ul>
        <li class="todo_sm"><a href="#"><span>leepiny的代辦</span></a></li>
        <li><a @click="logout">登出</a></li>
      </ul>
    </nav>
    <div class="conatiner todoListPage vhContainer">
      <div class="todoList_Content">
        <div class="inputBox">
          <input type="text" placeholder="請輸入待辦事項" v-model="taskInput" @keyup.enter="inputTask">
          <a @click="inputTask">
            <i class="fa fa-plus"></i>
          </a>
        </div>
        <div class="todoList_list" v-if="listInfo.toDo.length > 0">
          <ul class="todoList_tab">
            <li><a href="#" @click.prevent="activeTab = 'all'" :class="{active:activeTab === 'all'}">全部</a></li>
            <li><a href="#" @click.prevent="activeTab = 'pending'" :class="{active:activeTab === 'pending'}">待完成</a></li>
            <li><a href="#" @click.prevent="activeTab = 'complete'" :class="{active:activeTab === 'complete'}">已完成</a></li>
          </ul>
          <div class="todoList_items">
            <ul class="todoList_item">
                <li v-for="(item) in filteredTodos" :key="item.id">
                    <label class="todoList_label">
                        <input class="todoList_input" type="checkbox" value="true" v-model="item.status" true-value="complete" false-value="">
                        <span>{{ item.task }}</span>
                    </label>
                    <a @click="deleteTask(item.id)">
                        <i class="fa fa-times"></i>
                    </a>
                </li>
            </ul>
            <div class="todoList_statistics">
              <p>{{ completedCount }} 個等待完成項目</p>
            </div>
          </div>
        </div>
        <div v-else>目前尚無待辦事項</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref, computed} from 'vue'
import { useRouter } from 'vue-router'
import axios from "axios";

const api = 'https://todolist-api.hexschool.io';
const router  = useRouter();

const activeTab = ref('all')
const taskInput = ref('');

const listInfo = ref({
    toDo: [
        {
            id      : Date.now()-3,
            task    : 'Vue',
            status  : ''
        },
        {
            id      : Date.now()-2,
            task    : 'Tokyo',
            status  : ''
        },
        {
            id      : Date.now()-1,
            task    : 'AWS',
            status  : ''
        }
    ]
})

const inputTask = ()=> {
  if (!taskInput.value.trim()) return
  listInfo.value.toDo.push({
    id      :Date.now(),
    task    :taskInput.value,
    status  :''
  })
}

const deleteTask = (id)=>{
  listInfo.value.toDo = listInfo.value.toDo.filter(item => item.id !== id)
}

const filteredTodos = computed(() => {
  const todos = listInfo.value.toDo;
  if (activeTab.value === 'complete') return todos.filter(t => t.status === 'complete')
  if (activeTab.value === 'pending')  return todos.filter(t => t.status !== 'complete')
  return todos;
})

const completedCount = computed(()=>{
  return listInfo.value.toDo.filter(item => item.status === '').length
})

const logout = async ()=>{
    const token = document.cookie.replace(/(?:^|.*;\s*)todo_token\s*=\s*([^;]*).*$/i,"$1");
    
    try{
        const res = await axios.post(`${api}/users/sign_out`,{}, {
            headers: {
                Authorization: token
            }
        })
        
        if(res.data.status)
        {
            document.cookie = "todo_token=; path=/; max-age=0";
            router.push('/signup');
        }
    }
    catch (err){
        console.log(err);
    }
}
</script>

<style scoped>
  .todoListPage {
    padding: 16px 32px;
  }
  .todoList_Content {
    width: 500px;
    margin: 0 auto;
  }
  .inputBox {
    width: 100%;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    position: relative;
    margin-bottom: 16px;
    -webkit-box-shadow: 0 0 15px 0 rgba(0, 0, 0, 0.15);
    box-shadow: 0 0 15px 0 rgba(0, 0, 0, 0.15);
  }
  .inputBox input {
    background: #fff;
    border: none;
    border-radius: 10px;
    position: relative;
    width: 100%;
    height: 47px;
    font-size: 1rem;
    padding-left: 16px;
  }
  .inputBox a {
    display: block;
    width: 40px;
    height: 39px;
    position: absolute;
    background: #333333;
    color: white;
    font-size: 20px;
    text-decoration: none;
    text-align: center;
    border-radius: 10px;
    top: 4px;
    right: 4px;
    padding: 10px;
  }
  .todoList_list {
    background: #fff;
    border-radius: 10px;
    -webkit-box-shadow: 0 0 15px 0 rgba(0, 0, 0, 0.15);
    box-shadow: 0 0 15px 0 rgba(0, 0, 0, 0.15);
  }
  .todoList_list .todoList_tab {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: space-evenly;
    -ms-flex-pack: space-evenly;
    justify-content: space-evenly;
  }
  .todoList_list .todoList_tab li {
    width: 100%;
  }
  .todoList_list .todoList_tab a {
    display: block;
    color: #9F9A91;
    text-decoration: none;
    line-height: 20px;
    font-weight: bold;
    text-align: center;
    padding: 16px;
    border-bottom: 2px solid #efefef;
  }
  .todoList_list .todoList_tab .active {
    color: #333333;
    border-bottom: 2px solid #333333;
  }
  .todoList_list .todoList_items {
    padding-top: 23px;
    padding-left: 24px;
    padding-right: 17px;
    padding-bottom: 32px;
  }
  .todoList_list .todoList_items .todoList_item {
    margin-bottom: 8px;
  }
  .todoList_list .todoList_items .todoList_label {
    width: 100%;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    border-bottom: 1px solid #e5e5e5;
    padding-bottom: 15px;
    color: #333333;
    line-height: 20.27px;
  }
  .todoList_list .todoList_items .todoList_input {
    width: 20px;
    height: 20px;
    border: 1px solid #9F9A91;
    border-radius: 5px;
    margin-right: 16px;
  }
  .todoList_list .todoList_items .todoList_input:checked ~ span {
    color: #9F9A91;
    text-decoration: line-through;
    -webkit-transition: all 0.4s ease-in-out;
    transition: all 0.4s ease-in-out;
  }
  .todoList_list .todoList_items li {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    margin-bottom: 17px;
  }
  .todoList_list .todoList_items li a {
    margin-left: 17px;
    display: block;
    font-size: 14px;
    color: #333333;
    opacity: 0;
  }
  .todoList_list .todoList_items li:hover a {
    opacity: 1;
  }
  .todoList_list .todoList_statistics {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: justify;
    -ms-flex-pack: justify;
    justify-content: space-between;
  }
  .todoList_list .todoList_statistics p {
    color: #333333;
    font-size: 0.875rem;
  }
  .todoList_list .todoList_statistics a {
    color: #9F9A91;
    font-size: 0.875rem;
    text-decoration: none;
  }
</style>
