<script lang="ts">
  import gptVideo from './assets/shiraga_gpt_3.mp4'
  import IconButton from '@smui/icon-button';
  import { MetaTags } from 'svelte-meta-tags';
  import axios from 'axios';

let video: HTMLVideoElement
let inputValue = ''
let isLoading = true
let isSpeaking = false
let question = ''
let isSending = false

let i = 0;
let txt = ''
let speed = 110; /* The speed/duration of the effect in milliseconds */

const API_URL = 'https://api.openai.com/v1/';
const MODEL = 'gpt-3.5-turbo';
const API_KEY = import.meta.env.VITE_OPEN_AI_API_KEY;

const typeWriter = () => {
  isSpeaking = true

  if (i < txt.length) {
    document.getElementById("message").innerHTML += txt.charAt(i);
    i++;
    setTimeout(typeWriter, speed);
  }else{
    video.pause()
    video.currentTime = 0;
    isSpeaking = false
  }
}

const sendMessageToChatGpt = async ( message: string ) => {
  try {
    const response = await axios.post( `${ API_URL }chat/completions`, {
      model: MODEL,
      messages: [
        {
          'role': 'system',
          'content': 'あなたの名前は「shiragaGPT」です。名前を聞かれたら「shiragaGPT」と答えてください。200文字以内で答えることを心がけてください'
        },
        {
          'role': 'user',
          'content': message,
        },
      ],
    }, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${ API_KEY }`
      }
    });
    console.log('response',response)
    // 回答の取得
    return response.data.choices[0].message.content;
  } catch ( error ) {
    console.error( error );
    return null;
  }
}

const send = async () => {
  if(!inputValue && isSpeaking){
    return
  }

  if(inputValue.length > 100){
    return
  }

  question = inputValue
  inputValue = ''
  document.getElementById("message").innerHTML = ''
  i = 0

  isSending = true

  const result = await sendMessageToChatGpt(question)

  if(!result){
    isSending = false
    alert('メッセージの送信に失敗しました')
    return
  }
  txt = result

  console.log('message: ',result)

  video.play()
  isSending = false
  typeWriter()
}

</script>

<MetaTags
  title="shiragaGPT"
  description="shiragaGPTがなんでも答えてくれる"
  canonical="https://shiraga-gpt.vercel.app/"
  openGraph={{
    type: 'website',
    url: 'https://shiraga-gpt.vercel.app/',
    title: 'shiragaGPT',
    description: 'shiragaGPTがなんでも答えてくれる',
    images: [
      {
        url: 'https://shiraga-gpt.vercel.app/src/assets/shiraga_gpt.png',
        width: 1456,
        height: 816,
        alt: 'shiragaGPT image'
      }
    ]
  }}
/>

<main>
  <div hidden={!isLoading}>
    <div class="spinner-box">
      <div class="circle-border">
        <div class="circle-core"></div>
      </div>  
    </div>
  </div>

  <div hidden={isLoading}>
    <video bind:this={video} src={gptVideo} muted loop on:loadeddata={() => isLoading = false} controlsList="nodownload" ><track kind="captions"></video>
      <div class="chat-area">
        <div class="message-area">
          <p class="question">{question}</p>
          <p id="message" class="message"></p>
        </div>
        <div class="input">
          <textarea disabled={isSpeaking || isSending} placeholder={isSpeaking || isSending ? '':'メッセージを入力してください'} maxlength="100"  bind:value={inputValue} ></textarea>
          <div class="icon-button">
            <IconButton class="material-icons" disabled={isSpeaking || isSending} style="outline: none;"  on:click={send}
              >send</IconButton
            >
          </div>
          <p class="input-length">{inputValue.length}/100</p>
          <!-- LOADING DOTS... -->
          {#if isSending}
          <div class="message-loading">
              <div class="pulse-container">  
                <div class="pulse-bubble pulse-bubble-1"></div>
                <div class="pulse-bubble pulse-bubble-2"></div>
                <div class="pulse-bubble pulse-bubble-3"></div>
              </div>
          </div>
          {/if}
        </div>
      </div>
    <div class="logo">
      <p class="logo-shiraga">shiraga</p>
      <p class="logo-gpt">GPT</p>
    </div>
  </div>

  
</main>

<style>
  main {
    position: relative;
  }

  .chat-area {
    position: absolute;
    top: 50px;
    left: 100px;
  }

  .message-area {
    color: #fff;
    background-color: #000;
    padding: 10px;
    border-radius: 10px;
    width: 300px;
    min-height: 340px;
    text-align: left;
    padding-left: 20px;
    opacity: 0.7;
  }

  .question {
    height: 50px;
    color: aqua;
    overflow: scroll;
  }

  .message {
    height: 310px;
    opacity: initial;
    display: flex;
    flex-direction: column-reverse;
    overflow: scroll;
  }

  .input {
    width: 320px;
    background-color: #000;
    margin-top: 10px;
    border-radius: 10px;
    opacity: 0.7;
    padding: 10px 0;
    padding-right: 10px;
  }

  textarea {
    border: none;
    background-color: #000;
    color: #fff;
    width: 270px;
    outline: none;
    padding-left: 20px;
    padding-right: 40px;
    border-radius: 10px;
    height: 92px;
    font-size: 16px;
    resize: none;
  }

  .icon-button {
    position: absolute;
    right: 48px;
    bottom: 50px;
    height: 0px;
    width: 0px;
  }

  .input-length {
    position: absolute;
    right: 0;
    bottom: -40px;
    font-weight: bold;
  }

  .message-loading {
    position: absolute;
    bottom: 160px;
    left: 20px;
  }

  .logo {
    background-color: #000;
    position: absolute;
    bottom: 10px;
    left: 5px;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    font-weight: bold;
  }

  .logo-shiraga {
    margin-top: 20px;
  }

  .logo-gpt {
    margin-top: -15px;
    font-size: 20px;
  }

  /* --------------------------------------------- mainLoading --------------------------------------------- */

  .spinner-box {
  width: 300px;
  height: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: transparent;
}

.circle-border {
  width: 150px;
  height: 150px;
  padding: 3px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  background: rgb(63,249,220);
  background: linear-gradient(0deg, rgba(63,249,220,0.1) 33%, rgba(63,249,220,1) 100%);
  animation: spin .8s linear 0s infinite;
}

.circle-core {
  width: 100%;
  height: 100%;
  background-color: #1d2630;
  border-radius: 50%;
}

@keyframes spin {
  from {
    transform: rotate(0);
  }
  to{
    transform: rotate(359deg);
  }
}

/* PULSE BUBBLES */

.pulse-container {
  width: 60px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.pulse-bubble {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background-color: #3ff9dc;
}

.pulse-bubble-1 {
    animation: pulse .4s ease 0s infinite alternate;
}
.pulse-bubble-2 {
    animation: pulse .4s ease .2s infinite alternate;
}
.pulse-bubble-3 {
    animation: pulse .4s ease .4s infinite alternate;
}

@keyframes pulse {
  from {
    opacity: 1;
    transform: scale(1);
  }
  to {
    opacity: .25;
    transform: scale(.75);
  }
}

</style>


