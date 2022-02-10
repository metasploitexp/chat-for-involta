<template>
    <div ref="chat" id="chat" class="container">
        <div ref="messages-block" class="messages-block">
            <div v-for="message in messages" :class="{ 'from-me': message.myself, 'from-them': !message.myself }" class="messages-block__message" :key="message">
                {{message.text}}
            </div> 
            <div v-if="!error && loading" class="alert-block alert-block--loading">Loading...</div>
            <div v-if="error" class="alert-block alert-block--error">{{error}}</div>
        </div>
        <div class="input-block">
            <div class="input-block__input">
                <b-form-textarea v-model="inputText" v-on:keyup.enter="sendMessage"></b-form-textarea>
            </div>
            <div class="input-block__btn" align-v="center">
                <b-button @click="sendMessage" variant="success">Отправить</b-button>
            </div>
        </div>
    </div>
</template>

<script>
import axios from '../axios';
export default {
    data() {
        return {
            messages: [],
            inputText: localStorage.getItem('input') || '',
            perPage: 20,
            currentPage: 0,
            isEnd: false,
            error: null,
            loading: false
        }
    },
    mounted() {
        this.getData();
        this.$refs['messages-block'].addEventListener('scroll', this.handleScroll);
    },
    methods: {
        async getData() {
            this.loading = true;

            try {
                const response = await axios.get('/getMessages?', {params : {offset: this.perPage * this.currentPage}});
                
                if (response.data.result) {
                    this.error = null;
                    let messages = response.data.result.map((msg) => {
                        return {
                            myself: false,
                            text: msg
                        }
                    });

                    this.messages = [].concat(messages, this.messages);

                    if (messages.length < this.perPage) {
                        this.isEnd = true;
                    }

                    this.onLoadMessages();
                } else {
                    this.error = 'Try it again!';
                }
            } catch (error) {
                return error;
            }

            this.loading = false;
        },

        onLoadMessages() {
            if (!this.currentPage) {
                this.scrollToDown();
            }
        },

        scrollToDown() {
            this.$nextTick(() =>  {
                this.$refs['messages-block'].scrollTop = this.$refs['messages-block'].scrollHeight;
            });
        },
        
        sendMessage() {
            this.messages.push({
                myself: true,
                text: this.inputText
            });
            this.inputText = '';
            this.scrollToDown();
        },

        getNextData() {
            if (this.isEnd) {
                return;
            }

            this.currentPage++;
            this.getData();
        },

        handleScroll(e) {
            let element = e.target;

            if (element.scrollTop <= 50) {
                this.getNextData();
            }
        }
    },
    watch: {
        inputText: function () {
            localStorage.setItem('input', this.inputText);
        }
    }
}
</script>

<style scoped>
#chat {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
    width: 650px;
    max-height: 100vh;
    display: flex;
    flex-flow: column;
    border: 1px solid #ccc;
    padding: 0;
}

#chat .messages-block {
    overflow-y: scroll;
    padding: 10px;
    display: flex;
    flex-flow: column;
}

#chat .messages-block .alert-block {
    display: inline;
    border-radius: 18px;
    padding: 8px 14px;
    margin: 5px 0;
    max-width: 75%;
    align-self: center;
    position: fixed;
    top: 5px;
}

#chat .messages-block .alert-block--loading {
    background-color: #b0d6fd;
    box-shadow: 0px 3px 10px #ccc;
    border: 1px solid #ccc;
}

#chat .messages-block .alert-block--error {
    background-color: #ffdcdc;
    box-shadow: 0px 3px 10px #ccc;
    border: 1px solid #ccc;
}

#chat .messages-block .messages-block__message {
    display: inline;
    border-radius: 18px;
    padding: 8px 14px;
    margin: 5px 0;
    max-width: 75%;
}

#chat .messages-block .messages-block__message.from-them {
    align-items: flex-start;
    background-color: #E9E9EB;
    color: #000;
}

#chat .messages-block .messages-block__message.from-me {
    align-self: flex-end;
    background-color: #248bf5;
    color: #fff;
}

#chat .input-block {
    padding: 10px;
    background-color: #fafbfc;
    border-top: 1px solid #ccc;
    display: flex;
    flex-flow: wrap;
    gap: 10px;
}

#chat .input-block .input-block__input {
    flex: 4;
}
</style>