<template>
    <v-container>
        <v-row justify="center">
            <v-col cols="12" md="8">
                <v-card>
                    <v-card-title class="text-center text-h5">
                        채팅
                    </v-card-title>
                    <v-card-text>
                        <div class="chat-box">
                            <div 
                            v-for="(msg, index) in messages"
                            :key="index"
                            :class="['chat-message', msg.senderEmail === this.senderEmail ? 'sent' : 'recieved']"
                            >
                            <strong>{{ msg.senderEmail }}:</strong> {{ msg.message }}
                            </div>
                        </div>
                        <v-text-field
                            v-model="newMessage"
                            label="메시지를 입력하세요"
                            @keyup.enter="sendMessage"
                        />
                        <v-btn color="primary" block @click="sendMessage">전송</v-btn>
                    </v-card-text>
                </v-card>
            </v-col>
        </v-row>
    </v-container>

</template>
<script>
import SockJS from 'sockjs-client';
import Stomp from 'webstomp-client';
// import axios from 'axios';

export default{
    data(){
        return{
            //뿌려줄 메시지 배열 선언
            messages: [],
            newMessage: "",
            stompClient: null,
            token: "",
            senderEmail: null,
        }
    },
    created(){
        this.senderEmail = localStorage.getItem("email");
        this.connectWebsocket();
    },
    //사용자가 현재 라우트에서 다른 라우트로 이동하려고 할 때 호출되는 훅함수
    beforeRouteLeave(to, from, next) {
        this.disconnectWebsocket();
        next();
    },
    //화면을 완전히 꺼버렸을때
    beforeUnmount() {
        //서버에 객체가 계속 쌓이면 안됨
        //정말로 중요한 훅
        this.disconnectWebsocket(); // 컴포넌트가 언마운트되기 전에 웹소켓 연결 종료
    },
    methods: {
        connectWebsocket(){
            if(this.stompClient && this.stompClient.connected) return;
            //Stomp 통신을 위한 객체 생성
            //SockJs는 웹소켓을 내장한 향상된 js 라이브러리 임
            // Http 앤드포인트를 사용함.
            const sockJs = new SockJS(`${process.env.VUE_APP_API_BASE_URL}/connect`);
            this.stompClient = Stomp.over(sockJs);
            this.token = localStorage.getItem("token")
            this.stompClient.connect({
                Authorization: `Bearer ${this.token}`
            },
                ()=>{
                    this.stompClient.subscribe(`/topic/1`, (message) => {
                        const parseMessage = JSON.parse(message.body);
                        //console.log("Received message: ", message);
                        this.messages.push(parseMessage); 
                        this.scrollToBottom(); 
                    });
                }
            )
        },
        sendMessage(){
            if(this.newMessage.trim() === "") return; //빈 메시지 방지

            //이메일과 메시지를 조합한 객체 생성
            const message={
                senderEmail: this.senderEmail,
                message: this.newMessage
            }

            //axios가 아니다보니 직접 Json으로 파싱필요
            this.stompClient.send(`/publish/1`, JSON.stringify(message));
            this.newMessage = ""; //입력창 초기화
        },
        // 메시지가 들어오면 스크롤을 아래로 자동으로 내려주는 메서드
        scrollToBottom() {
            this.$nextTick(() => {
                const chatBox = this.$el.querySelector('.chat-box');
                chatBox.scrollTop = chatBox.scrollHeight;
            });
        },
        disconnectWebsocket() {
            if(this.stompClient && this.stompClient.connected){
                this.stompClient.unsubscribe(`/topic/1`);
                this.stompClient.disconnect();
            }
        }
    }
}
</script>

<style>
.chat-box {
    height: 300px;
    overflow-y: auto;
    border: 1px solid #ddd;
    margin-bottom: 10px;
}

.chat-message {
    margin-bottom: 10px;
}

.sent{
    text-align: right;
}
.received{
    text-align: left;
}
</style>