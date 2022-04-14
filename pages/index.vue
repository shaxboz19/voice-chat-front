<template>
  <div class="container-fluid h-50">
    <div class="row justify-content-center h-100">
      <div class="col-md-4 col-xl-3 chat">
        <div class="card mb-sm-3 mb-md-0 contacts_card">
          <div class="card-header">
            <div class="input-group">
              <input
                type="text"
                placeholder="Search..."
                name=""
                class="form-control search"
              />
              <div class="input-group-prepend">
                <span class="input-group-text search_btn"
                  ><i class="fas fa-search"></i
                ></span>
              </div>
            </div>
          </div>
          <div class="card-body contacts_body">
            <ul class="contacts">
              <li class="active">
                <div class="d-flex bd-highlight">
                  <div class="img_cont">
                    <img
                      src="https://therichpost.com/wp-content/uploads/2020/06/avatar2.png"
                      class="rounded-circle user_img"
                    />
                    <span class="online_icon"></span>
                  </div>
                  <div class="user_info">
                    <span>jassa</span>
                    <p>Kalid is online</p>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <div class="card-footer"></div>
        </div>
      </div>
      <div class="col-md-8 col-xl-6 chat">
        <div class="card">
          <div class="card-header msg_head">
            <div class="d-flex bd-highlight">
              <div class="img_cont">
                <img
                  src="https://therichpost.com/wp-content/uploads/2020/06/avatar2.png"
                  class="rounded-circle user_img"
                />
                <span class="online_icon"></span>
              </div>
              <div class="user_info">
                <span>Chat with jassa</span>
                <p>1767 Messages</p>
              </div>
              <div class="video_cam">
                <span><i class="fas fa-video"></i></span>
                <span><i class="fas fa-phone"></i></span>
              </div>
            </div>
            <span id="action_menu_btn"><i class="fas fa-ellipsis-v"></i></span>
            <div class="action_menu">
              <ul>
                <li><i class="fas fa-user-circle"></i> View profile</li>
                <li><i class="fas fa-users"></i> Add to close friends</li>
                <li><i class="fas fa-plus"></i> Add to group</li>
                <li><i class="fas fa-ban"></i> Block</li>
              </ul>
            </div>
          </div>
          <div class="card-body msg_card_body">
            <div
              class="d-flex justify-content-start mb-4"
              v-for="(item, index) in messages"
              :key="index"
            >
              <div class="img_cont_msg">
                <img
                  src="https://therichpost.com/wp-content/uploads/2020/06/avatar2.png"
                  class="rounded-circle user_img_msg"
                />
              </div>
              <div class="msg_cotainer">
                <div class="action">
                  <span v-if="item.isPause">
                    <svgicon
                      name="pause"
                      color="#2675EC"
                      width="35px"
                      @click="pauseAudio(index, item)"
                    ></svgicon>
                  </span>
                  <span v-else>
                    <svgicon
                      name="play"
                      color="#2675EC"
                      width="35px"
                      @click="playAudio(index, item)"
                    ></svgicon>
                  </span>
                </div>
                <av-waveform
                  ref="audio"
                  :audio-src="item.url"
                  :canv-width="300"
                  played-line-color="#fff"
                  :played-line-width="2"
                  :played-line-height="20"
                  noplayed-line-color="#eef1f2"
                  :playtime-slider="false"
                  playtime-slider-color="#2675EC"
                ></av-waveform>
                <span class="msg_time">{{ item.createAt | formatDate }}</span>
              </div>
            </div>
          </div>
          <div class="card-footer">
            <div class="input-group footer-input">
              <div class="microphone" @mousedown="recordAudio" @mouseup="stop">
                <svgicon name="record" color="#858E99"></svgicon>
              </div>
              <div class="timer">
                {{ timer | formatTime }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "IndexPage",
  data() {
    return {
      recorder: null,
      chunks: [],
      device: null,
      blobObj: null,
      timer: 0,
      interval: null,
      messages: [],
      activeItemAudio: null,
      activeItemIndex: null,
    };
  },
  async mounted() {
    this.device = navigator.mediaDevices.getUserMedia({ audio: true });
  },
  methods: {
    recordAudio() {
      this.device.then((stream) => {
        this.recorder = new MediaRecorder(stream);
        this.recorder.ondataavailable = (e) => {
          this.chunks.push(e.data);
          if (this.recorder.state === "inactive") {
            let blob = new Blob(this.chunks, { type: "audio/wav" });
            this.blobObj = blob;
            this.chunks = [];
            this.sendAudio(this.blobObj);
            this.blobObj = null;
          }
        };
        this.recorder.start();
        this.interval = setInterval(() => {
          this.timer++;
        }, 10);
      });
    },
    stop() {
      this.recorder.stop();
      clearInterval(this.interval);
      this.timer = 0;
    },

    async sendAudio(blob) {
      const fd = new FormData();
      fd.append("audio", blob);
      const { data } = await this.$axios.post("/audio", fd);
      this.messages.push(data);
    },

    playAudio(index, item) {
      this.$refs.audio.forEach((element) => {
        element.audio.onpause = () => {
          this.$set(item, "isPause", false);
        };
        element.audio.pause();
      });
      this.$refs.audio[index].audio.play();
      this.$set(item, "isPause", true);
      this.activeItemAudio = this.$refs.audio[index].audio;
      this.activeItemIndex = index;
    },
    pauseAudio(index, item) {
      this.$refs.audio[index].audio.pause();
      this.$set(item, "isPause", false);
    },
  },
  computed: {
    getDuration() {
      console.log("sss");
      return this.activeItemAudio ? this.activeItemAudio.duration : 0;
    },
    getCurrentTime() {
      return this.activeItemAudio ? this.activeItemAudio.currentTime : 0;
    },
  },
  filters: {
    formatTime(time) {
      const seconds = Math.floor(time / 100);
      const minutes = Math.floor(seconds / 60);
      const hours = Math.floor(minutes / 60);
      const ml = time % 100;
      const secStr = seconds < 10 ? "0" + seconds : seconds;
      const minStr = minutes < 10 ? "0" + minutes : minutes;
      const hrStr = hours < 10 ? "0" + hours : hours;
      const mlStr = ml < 10 ? "0" + ml : ml;
      return `${hrStr}:${minStr}:${secStr}:${mlStr}`;
    },
    formatDate(val) {
      return new Date(val).toLocaleString();
    },
  },
};
</script>
