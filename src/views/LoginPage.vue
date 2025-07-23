<template>
    <v-container>
        <v-row justify="center">
            <v-col cols="12" sm="4" md="6">
                <v-card>
                    <v-card-title class="text-h5 text-center">로그인</v-card-title>
                        <v-card-text>
                        <v-form @submit.prevent="doLogin">
                            <v-text-field
                                label="이메일"
                                v-model="email"
                                type="email"
                                required
                            >
                            </v-text-field>
                            <v-text-field
                                label="password"
                                v-model="password"
                                type="password"
                                required
                            >
                            </v-text-field>
                            <v-btn type="submit" color="primary" block>로그인</v-btn>
                        </v-form>
                    </v-card-text>
                </v-card>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
import axios from 'axios'
import { jwtDecode } from 'jwt-decode';

export default{
    data(){
        return{
            email: "",
            password: "",
        }
    },
    methods:{
        async doLogin(){
            const loginData = {email:this.email, password: this.password};
            const res = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/member/doLogin`, loginData);
            console.log(res);
            const token = res.data.token;
            // role과 email을 localStorage전역에 저장해주기위한 decode (encode되어있기 때문에)
            const role = jwtDecode(token).role;
            // 서버의 createToken 메서드를 확인해보면 Jwts.claims().setSubject(email); 에 email이 담겨있음
            const email = jwtDecode(token).sub;
            localStorage.setItem("token",token);
            localStorage.setItem("role",role);
            localStorage.setItem("email",email);
            window.location.href="/";
        }
    }
    
}
</script>