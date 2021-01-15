<template>
  <section>
    <div class="alert h6 text-right" 
      @click="doLogin">[login:{{data.user != null ? data.user.displayName : '---'}}]</div>
    <h2>{{title}}</h2>
    <p class="h5">{{data.message}}</p>
    <div v-if="data.user" class="alert alert-primary">
      <div class="form-group text-left">
        <label class="h5">Message</label>
        <div class="form-row">
          <div class="col-10">
            <input v-model="data.msg" class="form-control">
          </div>
          <button @click="add" class="btn btn-primary col-2">投稿</button>
        </div>
      </div>
    <h3 class="my-3">Messages</h3>
    <ul v-for="(item, key) in data.fire_data"
        class="list-group text-left">
      <li class="list-group-item">
        <div class="h5">{{item.msg}}</div>
        <div class="small text-right">{{item.user}} ({{item.posted}})</div>
      </li>
    </ul>
    </div>
    <div v-else>
      <div class="alert alert-warning">
        ※現在、ログインされていません。
      </div>
    </div>
  </section>
</template>

<script>
import { onMounted, reactive } from 'vue'
import firebase from 'firebase'

// Your web app's Firebase configuration
var firebaseConfig = {
  apiKey: "AIzaSyBRhvm4aCop7H4sxX3Hj2kJOKGvfP0Kmb8",
  authDomain: "sho-vue3.firebaseapp.com",
  databaseURL: "https://sho-vue3-default-rtdb.firebaseio.com",
  projectId: "sho-vue3",
  storageBucket: "sho-vue3.appspot.com",
  messagingSenderId: "149000623016",
  appId: "1:149000623016:web:d7dc252f3bf8c1cd53039d"
};

// Initialize Firebase
firebase.initializeApp(firebaseConfig)

var provider = new firebase.auth.GoogleAuthProvider()
const person = firebase.database().ref('board/')

export default {
  setup(props) {
    const data = reactive({
      title:'Board',
      message:'ミニ伝言板。最新の投稿を表示します。',
      user: null,
      msg:'',
      num_per_page:10, //☆取り出すデータ数
      fire_data:{},
    })
    // ログイン処理
    const login = ()=> {
      firebase.auth().signInWithPopup(provider)
          .then((result)=> {
        data.user = result.user
        data.message = 'ログインしました。'
        firebase.database().ref('board')
            .orderByKey()
            .limitToLast(data.num_per_page)
            .on('value', (snapshot)=> {
          if (firebase.auth().currentUser != null){
            let arr = []
            let result = snapshot.val()
            for(let item in result){
              arr.unshift(result[item])
            }
            console.log(arr)
            data.fire_data =arr
          } else {
            data.fire_data = {}
          }
        })
      })
    }
    // ログアウト処理
    const logout = ()=> {
      firebase.auth().signOut()
      data.user = null
      data.fire_data = {}
      data.message = 'ログアウトしました。'
    }
    // ログイン・ログアウト実行
    const doLogin = ()=> {
      if (firebase.auth().currentUser == null){
        login()
      } else {
        logout()
      }
    }
    // メッセージ追加
    const add = ()=> {
      if (firebase.auth().currentUser == null){
        alert('ログインしないと投稿できません。')
        return
      }
      let user = firebase.auth().currentUser
      console.log(user)
      let d = new Date()
      let dstr = d.getFullYear() + '-' + (d.getMonth() + 1) + '-'
        + d.getDate() + ' ' + d.getHours() + ':' + d.getMinutes()
        + ':' + d.getSeconds()
      let id = d.getTime()
      let obj = {
        msg: data.msg,
        user: user.displayName,
        posted: dstr,
      }
      firebase.database().ref('board/' + id).set(obj)
      data.msg = ''
      data.message = '投稿しました。'
    }
    onMounted(()=> {
      if (firebase.auth().currentUser == null){
        login()           
      }
      console.log(firebase.auth().currentUser)
    })
    return { data, login, logout, doLogin, add }
  },
}
</script>