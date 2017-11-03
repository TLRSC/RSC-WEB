<template>
  <section>
    <b-tabs position="is-centered" v-model="activeTab">
      <b-tab-item label="Console">
        <section class="console">
            <pre id="console">
              Console..
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

  export default {
    name: 'Home',
    data() {
      return {
        activeTab: 0,
        token: this.getToken(),
        socket: {},
        command: ''
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
        let console = document.getElementById('console')
        console.innerHTML = console.innerHTML + '<br>' + e
        console.scrollIntoView(false)
      },
      connect() {
        let p = jwt.Parser().parse(this.token).body;

        this.socket = io.connect("http://" + p.host + ":" + p.port + "/?secret=" + p.secret);
        this.socket.on('console', e => this.appendConsole(e))
      },
      sendCmd() {
        if (this.command.substr(0, 1) === '/')
          this.command = this.command.substr(1, this.command.length);
        this.socket.emit('command', {
          command: this.command
        });
        this.command = ''
      }
    }
  }
</script>

<style scoped>
  .sec {
    padding: 20px;
  }

  .console {
    height: auto;
    max-height: 100%;
    overflow: auto;
    background-color: #eeeeee;
    padding-bottom: 40px;
    position: relative;
  }

  .fixed-bottom {
    position: fixed;
    bottom: 0;
    width: 100%;
    background-color: white;
  }

  .fixed {
    position: fixed;
    top: 0;
    width: 100%;
  }
</style>
