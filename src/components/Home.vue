<template>
  <section>
    <b-tabs position="is-centered" v-model="activeTab">
      <b-tab-item label="Console">
        <b-loading :active.sync="isLoading" :canCancel="true" :onCancel="onCancel"></b-loading>
        <section class="console">
            <pre id="console">
            </pre>
        </section>

        <section class="sec fixed-bottom">
          <b-field grouped>
            <div class="control is-expanded is-clearfix">
              <input @keyup.enter="sendCmd()" v-model="command" type="text" autocomplete="on" placeholder="Command"
                     class="input">
            </div>
            <p class="control">
              <button @click="sendCmd()" class="button is-primary">Send</button>
            </p>
          </b-field>
        </section>
      </b-tab-item>

      <b-tab-item label="Info">
        <section class="sec">
          Server Info
        </section>
      </b-tab-item>

      <b-tab-item label="Setting">
        <section class="sec">
          <b-field grouped>
            <b-input v-model="token" placeholder="Token" expanded></b-input>
            <p class="control">
              <button @click="setToken()" class="button is-primary">Save</button>
            </p>
          </b-field>
        </section>
      </b-tab-item>
    </b-tabs>
  </section>
</template>

<script>
  import Cookies from 'js-cookie'
  import io from 'socket.io-client'
  import jwt from 'jwt-node'
  //  import Gamedig from 'gamedig'

  export default {
    name: 'Home',
    data() {
      return {
        activeTab: 0,
        token: this.getToken(),
        socket: {},
        command: '',
        isLoading: true
      }
    },
    created() {
      if (this.token === undefined) {
        this.activeTab = 2
      } else {
        this.connect()
      }
    },
    methods: {
      setToken() {
        Cookies.set("token", this.token);
        this.activeTab = 0;
        this.connect()
      },

      getToken() {
        return Cookies.get("token")
      },

      appendConsole(e) {
        let console = document.getElementById('console');
        console.innerHTML = console.innerHTML + '<br>' + e;
        console.scrollIntoView(false)
      },

      connect() {
        this.isLoading = true;

        let p;

        try {
          p = jwt.Parser().parse(this.token).body;
        } catch (e) {
          this.error('Token parse error');
          this.isLoading = false;
          this.activeTab = 2
        }

        this.socket = io.connect("ws://" + p.host + ":" + p.port + "/?" + p.secret, {transports: ['websocket']});
        this.socket.on('connect', e => this.onConnect(e));

        this.socket.on('connect_error', (e) => {
          this.error('Connecting error: ' + e);
          this.activeTab = 2
        });

        this.socket.on('console', e => this.appendConsole(e));

//        Gamedig.query({
//          type: 'minecraft',
//          host: 'mc.example.com'
//        }).then((state) => {
//          console.log(state);
//        }).catch((error) => {
//          console.log("Server is offline");
//        });

        this.isLoading = false;
      },

      sendCmd() {
        if (this.command.substr(0, 1) === '/')
          this.command = this.command.substr(1, this.command.length);
        this.socket.emit('command', {
          command: this.command
        });
        this.command = ''
      },

      onConnect(e) {
        this.success('Connected')
      },

      onCancel() {
        this.isLoading = false;
        this.activeTab = 2;
      },

      error(message) {
        this.$toast.open({
          duration: 3000,
          message: message,
          position: 'is-bottom',
          type: 'is-danger'
        })
      },

      success(message) {
        this.$toast.open({
          duration: 3000,
          message: message,
          position: 'is-top',
          type: 'is-success'
        })
      }
    },
    watch: {
//      '$route': function (e) {
//        this.activeTab = parseInt(e.path);
//      }
    }
  }
</script>

<style>
  .sec {
    padding: 20px;
  }

  .console {
    height: auto;
    max-height: 100%;
    overflow: auto;
    padding-bottom: 50px;
    position: relative;
  }

  .fixed-bottom {
    position: fixed;
    bottom: 0;
    width: 100%;
    background-color: white;
  }

  .tabs {
    z-index: 100;
    overflow: hidden;
    position: fixed;
    top: 0;
    width: 100%;
    background-color: white;
  }

  .tab-content {
    top: 41px;
  }
</style>
