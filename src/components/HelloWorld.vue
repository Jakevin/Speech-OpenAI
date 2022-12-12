<script setup>
import { ref } from 'vue'

//Backend Server
const BASE_URL = ""

const SpeechRecognition = webkitSpeechRecognition || SpeechRecognition;
const recognition = new SpeechRecognition();
const u = new SpeechSynthesisUtterance();

defineProps({
  msg: String,
})

const speechResult = ref("")
const aiAns = ref("是的，我是一個文字聊天機器人。你想聽我講什麼？")

const startRec = ref("cyan")

//#region status  
const isRecing = ref(false)
const isCalling = ref(false)
const speakCommentFlg = ref(false);
//#endregion

const pickedLang = ref("cmn-Hant-TW")


/**
 * 錄音完的回傳事件
 * The callback event after recording
 * 録音後のコールバックイベント
 * @param {*} event 
 */
recognition.onresult = (event) => {
  console.log(event);
  startRec.value = "cyan"
  isRecing.value = false

  speechResult.value = event.results[0][0].transcript
  onSearchClick()
}

/**
 * 開始錄音
 * Start Recognition
 * 録音を開始する
 */
function onClickRec(){
  if(isCalling.value) return
  
  recognition.lang = pickedLang.value;

  if(isRecing.value === true){
    isRecing.value = false
    recognition.stop(); 
  }else{
    startRec.value ="red"
    isRecing.value = true
    recognition.start();  
  }
}


function onSearchClick(){
  if(isCalling.value) return

  isCalling.value = true

  fetch(BASE_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      lang:pickedLang.value,
      question:speechResult.value
    })
  })
  .then(response => response.text())
  .catch(error => console.error('Error:', error))
  .then(response => {
    console.log('Success:', response)
    isCalling.value = false

    aiAns.value = response
    const synth = window.speechSynthesis;

    const speak = (msg) => {
      function changLang() {
        console.log("pickedLang",pickedLang.value)

        switch(pickedLang.value){
          case "cmn-Hant-TW":
            return "zh-TW"
          case "ja-JP":
            return "ja-JP"
          case "en-US":
            return "en-US"
          default:
            return "en-US"
        }
      }
      u.lang =changLang()
      console.log("changLang",changLang())
      u.text = msg;
      synth.speak(u);
      
    };
    speak(response);

  });
}

//for iOS, can't auto speak.
function speakSwitch() {
  const synth = window.speechSynthesis;

  let speakMsg = new SpeechSynthesisUtterance('Hi');
  speakMsg.lang = 'en-US';
  speakMsg.rate = 1.0;
  if ( speakCommentFlg.value == true ) {
    speakCommentFlg.value = false;
    speakMsg.text = '';
    synth.speak(speakMsg);
    return;
  }
  speakCommentFlg.value = true;
  speakMsg.text = 'Hi';
  synth.speak(speakMsg);
}
</script>

<template>
  <h1>{{ msg }}</h1>


  <div class="card">
    <input type="radio" id="chinese" value="cmn-Hant-TW" v-model="pickedLang" />
    <label for="chinese">中文</label>

    <input type="radio" id="eng" value="en-US" v-model="pickedLang" />
    <label for="eng">English</label>

    <input type="radio" id="jp" value="ja-JP" v-model="pickedLang" />
    <label for="jp">日本語</label>

    <div>
      <button @click="speakSwitch();">
        <span v-if="speakCommentFlg==false" class="material-symbols-outlined">record_voice_over</span>
        <span v-else class="material-symbols-outlined">voice_over_off</span>
      </button>
    </div>

    <textarea rows="4" cols="50" class="inputResult" placeholder="👇 Click MIC 👇" type="text" v-model="speechResult" />
    <div style="text-align: -webkit-center;">
      <div v-if="isCalling==true" class="loader"></div>
      <p  v-else class="ai-response">{{aiAns}}</p>
    </div>
    
  </div>

  <div>
    <button> <span class="material-symbols-outlined color-rec" style="font-size:50vmin "  @click="onClickRec()">mic</span> </button>
    <div> || </div>
    <button> <span class="material-symbols-outlined" style="font-size:50vmin " @click="onSearchClick()" >search</span> </button> 
  </div>

  <!-- <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Install
    <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p> -->
</template>

<style scoped>
.read-the-docs {
  color: #888;
}

.inputResult{
  margin-top: 30px;
  font-size: 8vmin;
  width:100%;
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}

.color-rec{
  color: v-bind(startRec);
}

.ai-response{
  margin-top: 50px;
  font-size: 8vmin;
  line-height:1.2
}

.material-symbols-outlined {
  font-size: 20px;
  font-variation-settings:
  'FILL' 0,
  'wght' 400,
  'GRAD' 0,
  'opsz' 48
}

.loader {
  border: 16px solid #f3f3f3;
  border-radius: 50%;
  border-top: 16px solid #3498db;
  width: 120px;
  height: 120px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 2s linear infinite;
}

/* Safari */
@-webkit-keyframes spin {
  0% { -webkit-transform: rotate(0deg); }
  100% { -webkit-transform: rotate(360deg); }
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
