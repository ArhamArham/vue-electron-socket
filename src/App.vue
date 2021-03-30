<template>
    <div id="app">
        <div class="container">
            <div class="col-lg-6 offset-lg-3">
                <div v-if='ready'>
                    <p v-for='(item,index) in info' :key="index">
                        {{item.name}} {{item.type}}
                    </p>
                </div>
                <div v-if='!ready'>
                    <h4>Set your name</h4>
                    <form @submit.prevent='addName'>
                        <div class="form-group">
                            <input type="text" v-model='name' class="form-control" placeholder="Pla enter your name"
                                   aria-describedby="helpId">
                        </div>
                    </form>
                </div>
            </div>
            <div class="col-lg-6 offset-lg-3" v-if='ready'>
                <h3 v-if='ready'>{{name}}</h3>
                <div class="card bg-info">
                    <div class="card-header text-white">
                        SocketIO ChatApp <span class="float-right">{{connections}}</span>
                    </div>
                    <ul class="list-group list-group-flush text-right">
                        <li class="list-group-item"
                            v-for='(message,index) in messages'
                            :key="index"
                        >
                            <span :class="{'float-left':message.type === 1}">
                                {{message.message}}
                                <small>:{{message.by}}</small>
                            </span>
                        </li>
                    </ul>
                    <div class="card-body">
                        <form @submit.prevent='send'>
                            <div class="form-group">
                                <small v-if="typing" class="text-white float-right">{{typing}} is typing</small>
                                <input type="text" v-if='ready' v-model='newMessage' class="form-control"
                                       placeholder="Enter Message" aria-describedby="helpId">
                            </div>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    import {io} from "socket.io-client"

    const socket = io('http://localhost:4004');

    export default {
        name: 'App',
        data() {
            return {
                newMessage: '',
                messages: [],
                typing: false,
                placeholder: '',
                info: [],
                ready: false,
                name: '',
                connections: 0
            }
        },
        watch: {
            newMessage(value) {
                value ? socket.emit('typing', this.name) : socket.emit('StopTyping')
            }
        },
        methods: {
            send() {
                this.messages.push({message: this.newMessage, type: 0, by: "Me"})
                socket.emit('chat-message', {message: this.newMessage, user: this.name})
                this.newMessage = ''
            },
            addName() {
                this.ready = true
                socket.emit('joined', this.name)
            }
        },
        created() {
            window.onbeforeunload = () => {
                socket.emit('leaved', this.name)
            }
            socket.on('connections', (data) => {
                this.connections = data
            })
            socket.emit('Created', 'Arham')

            socket.on('Created', (data) => {
                console.log(data);

            })
            socket.on('chat-message', (data) => {
                this.messages.push({message: data.message, type: 1, by: data.user})

            })
            socket.on('typing', (name) => {
                this.typing = name
            })
            socket.on('StopTyping', () => {
                this.typing = false
            })
            socket.on('joined', (data) => {
                this.info.push({name: data, type: 'joined'})
                setTimeout(() => {
                    this.info = []
                }, 5000)
            })
            socket.on('leaved', (data) => {
                this.info.push({name: data, type: 'leaved'})
                setTimeout(() => {
                    this.info = []
                }, 5000)
            })

        }
    }
</script>

<style>

</style>
