<template lang="pug">
 section.container
  #questions.question.chinese
    h2 {{choose_title}}
    .choose
      #boy.sexual(v-on:mouseover='playanim(true)', v-on:mouseleave='stopanim(true)', v-on:click='choose_sex(true)', v-show='is_choosesex')
        #bodymovin
        h3 男生
      #girl.sexual(v-on:mouseover='playanim(false)', v-on:mouseleave='stopanim(false)', v-on:click='choose_sex(false)', v-show='is_choosesex')
        #bodymovin_2
        h3 女生
      .birthday(v-show='!is_choosesex')
        datetime.theme-dark(v-model="user['birth']", format='yyyy-MM-dd') 西元年/月/日
    .btn(v-on:click='btn_continue') {{choose_btn}}
  #banner
    .content 
      #cc.countdown(v-if='is_countdown',) 
        span#hint 人生只剩下
        <br> {{timer}}
        //- .times(v-for='(item, index) in timer_split')
        //-   | {{ item }}

        //- - for(var i = 0; i < 6; i++)
        //-   .times {{timer_split[i]}}

      .char(v-for='(item, index) in banner_length', v-if='!is_countdown', :class="['c-' + index]")

          //- div(class=`c-${index} char`)
  .up
    button.chinese.btn.btn-start(v-on:click='start') 開始
  a#logos.logo(href='https://www.google.com') Doraralab
  //
    <div class="keyboard" v-if="!is_mobile">
    <van-datetime-picker v-model="currentDate" type="date" :confirm-button-text="comfirm_text" :cancel-button-text="cancel_text"/>
    </div>
</template>

<script>
import Logo from "~/components/Logo.vue";

let bodymovin;
if (process.browser) {
  bodymovin = require("bodymovin");
}

export default {
  components: {
    Logo
  },
  data() {
    return {
      timer: "asd",
      comfirm_text: "ok",
      cancel_text: "cancel",
      currentDate: new Date(),
      mousevalue: 0,
      title_size: 80,
      offset_X: 0,
      offset_Y: 0,
      myscope: null,
      is_active: true,
      is_countdown: false,
      fig1: null,
      fig2: null,
      padding: [400, 400, 300, 450],
      is_mobile: 1,
      user: {
        sex: "girl",
        birth: "生日"
      },
      is_choosesex: false,
      choose_btn: "繼續",
      choose_title: "生理性別",
      banner: "人生倒數計時",
      is_choosebirth: false
    };
  },
  head() {
    return {
      script: [
        {
          src:
            "https://cdn.rawgit.com/bradley/Blotter/3007fe6e/build/blotter.min.js"
        },
        {
          src:
            "https://cdn.rawgit.com/bradley/Blotter/3007fe6e/build/materials/channelSplitMaterial.js"
        },
        {
          src:
            "https://cdn.rawgit.com/bradley/Blotter/3007fe6e/build/materials/fliesMaterial.js"
        },
        {
          src:
            "https://cdnjs.cloudflare.com/ajax/libs/gsap/1.20.2/TweenMax.min.js"
        },
        {
          src:
            "https://cdnjs.cloudflare.com/ajax/libs/gl-matrix/2.8.1/gl-matrix-min.js"
        }
      ]
    };
  },
  mounted() {
    this.detect_screen();
    // this.blotter();
    this.textAnimated(this.banner);
    if (process.browser) {
      this.addfigure();
    }
    if (this.is_mobile) {
      this.datpicker();
    }
  },
  computed: {
    banner_length: function() {
      return this.banner.split("");
    },
    timer_split: function() {
      return this.timer.split(" ");
    }
  },
  methods: {
    textAnimated: function(chars) {
      const content = document.querySelector(".content");
      const rAF = requestAnimationFrame;
      chars.split("").forEach((char, i) => {
        const text = new Blotter.Text(char, {
          family: "serif",
          size: this.title_size,
          fill: "#171717"
        });
        const values = {
          degrees: Math.random() * 2 - 1,
          maxDist: 0.02
        };
        const material = new Blotter.ChannelSplitMaterial();
        const blotter = new Blotter(material, {
          texts: text
        });
        const elem = document.querySelector(`.c-${i}`);
        const scope = blotter.forText(text);
        const radTodegrees = 180 / Math.PI;

        scope.appendTo(elem);

        const b_canvas = elem.querySelector(".b-canvas");

        content.addEventListener("mousemove", e => {
          const bounds = b_canvas.getBoundingClientRect();
          const radiusX = bounds.width / 2;
          const radiusY = bounds.height / 2;
          const offX = bounds.left + radiusX;
          const offY = bounds.top + radiusY;
          const mouse = vec2.fromValues(e.clientX, e.clientY);
          const offset = vec2.fromValues(offX, offY);
          const rad = angleTo(mouse, offset);
          const dist = vec2.distance(mouse, offset);

          values.degrees = angle360(radTodegrees * rad);
          values.maxDist = Math.min(
            Math.min(dist / radiusX, dist / radiusY) * 0.02,
            0.05
          );
        });

        const animate = () => {
          b_canvas.style.transform = `translate3d(mouse.x,0,0)`;
          material.uniforms.uOffset.value = values.maxDist + 0.01;
          material.uniforms.uRotation.value = values.degrees;
          material.uniforms.uApplyBlur.value = values.maxDist;
          material.uniforms.uAnimateNoise.value = values.maxDist;
          rAF(animate);
        };
        animate();
      });
      const angleTo = ([x1, y1], [x2, y2]) => Math.atan2(y1 - y2, x1 - x2);
      const angle360 = x => (x + 360) % 360;
    },
    birthcountdown: function() {
      this.is_countdown = true;
      var nowdate = new Date();
      let girl_day = 30571;
      let boy_day = 28226;
      var bir = new Date(this.user.birth);
      if (this.user["sex"] == "girl") {
        bir.setDate(bir.getDate() + girl_day);
      } else {
        bir.setDate(bir.getDate() + boy_day);
      }

      setInterval(() => {
        var date = new Date();
        var countDownDate = bir - date;
        var years = Math.floor(countDownDate / (1000 * 60 * 60 * 24 * 365));
        var days = Math.floor(
          (countDownDate % (1000 * 60 * 60 * 24 * 365)) / (1000 * 60 * 60 * 24)
        );
        var hours = Math.floor(
          (countDownDate % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)
        );
        var minutes = Math.floor(
          (countDownDate % (1000 * 60 * 60)) / (1000 * 60)
        );
        var seconds = Math.floor((countDownDate % (1000 * 60)) / 1000);
        var ms = Math.floor(countDownDate % 1000);
        if(ms<100){
          ms = '0' + ms
          if(ms<10){
            ms = '00' + ms
          }
        }
        this.timer =
          years +
          "y " +
          days +
          "d " +
          hours +
          "h \n " +
          minutes +
          "m " +
          seconds +
          "s " +
          ms +
          "ms";
      }, 1);
    },
    datpicker: function() {},
    btn_continue: function() {
      if (this.user.sex == "") {
        alert("先選性別");
        return;
      }

      if (!this.is_choosesex) {
        this.choose_btn = "繼續";
        this.choose_title = "生理性別";
        if (this.user.birth == "") {
          alert("先選生日");
          return;
        }
        this.birthcountdown();
        this.start();
      }
      this.is_choosesex = false;
      this.choose_btn = "完成";
      this.choose_title = "生日";
      console.log(this.is_choosebirth);
    },
    choose_birthday: function() {},
    choose_sex: function(t) {
      if (t) {
        this.user.sex = "boy";
        document.getElementById("boy").style.backgroundColor =
          "rgba(255, 203, 85, 0.9)";
        document.getElementById("girl").style.backgroundColor = "white";
      } else {
        this.user.sex = "girl";
        document.getElementById("boy").style.backgroundColor = "white";
        document.getElementById("girl").style.backgroundColor =
          "rgba(255, 203, 85, 0.9)";
      }
    },
    playanim: function(t) {
      if (t) {
        this.fig1.play();
      } else {
        this.fig2.play();
      }
    },
    stopanim: function(t) {
      if (t) {
        this.fig1.stop();
      } else {
        this.fig2.stop();
      }
    },
    addfigure: function() {
      /* Shapes */
      var svgContainer = document.getElementById("bodymovin");
      var svgContainer2 = document.getElementById("bodymovin_2");
      var animItem = bodymovin.loadAnimation({
        wrapper: svgContainer,
        animType: "svg",
        autoplay: false,
        loop: false,
        path: "/data.json"
      });
      var animItem2 = bodymovin.loadAnimation({
        wrapper: svgContainer2,
        animType: "svg",
        autoplay: false,
        loop: false,
        path: "/cat.json"
      });
      this.fig1 = animItem;
      this.fig2 = animItem2;
    },
    test: function() {
      console.log("test");
    },
    start: function() {
      if (!this.is_active) {
        document.getElementById("logos").style.color = "black";
        document.body.style.backgroundColor = "white";
        document.getElementById("questions").style.display = "none";
        this.is_active = !this.is_active;
        this.is_choosesex = false;
      } else {
        document.getElementById("logos").style.color = "white";
        document.body.style.backgroundColor = "black";
        document.getElementById("questions").style.display = "flex";
        this.is_active = !this.is_active;
        this.is_choosesex = true;
        this.user.sex = "";
        this.user.birth = "";
      }
    },
    detect_screen: function() {
      if (
        /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
          navigator.userAgent
        )
      ) {
        this.is_mobile = 0;
        this.title_size = 60;
        document.getElementById("questions").style.width = "90%";
        document.getElementById("questions").style.height = "70%";
      } else {
        this.is_mobile = 1;
        this.title_size = 150;
      }
    }
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.chinese {
  font-family: "Source Sans Pro";
}

.logo {
  position: absolute;
  bottom: 20px;
  font-size: 0.2rem;
  padding: 6px 80px;
  border-radius: 1px;
  cursor: pointer;
  border: 2px solid #000;
  font-size: 0.8rem;
  color: black;
}

#banner {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

#banner * {
  /* pointer-events: none; */
}

.title {
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}

.up {
  z-index: 900;
}

.btn-start {
  margin-top: 20rem;
  padding: 10px 20px;
  cursor: pointer;
  /* animation: flash 3s infinite; */
}

.question {
  background-color: #fff;
  width: 50rem;
  height: 50rem;
  position: absolute;
  z-index: 999;
  display: flex;
  justify-content: space-around;
  color: black;
  display: none;
  align-items: center;
  flex-direction: column;
}

.choose {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 70%;
}

.sexual {
  cursor: pointer;
}

.sexual:hover {
  animation: flash 5s infinite;
}

.birthday {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

#bodymovin {
  /* max-width: 50%;
  max-height: 50%;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto; */
}

body {
  transition: 200ms;
}

.btn-start:hover {
  background-color: white;
  color: black;
}

.btn {
  padding: 10px 30px;
  background-color: #000;
  color: white;
}
.btn:hover {
  background-color: rgb(255, 44, 220);
  animation: flash 1s infinite;
}

.keyboard {
  position: absolute;
  width: 100%;
  height: 30%;
  bottom: -20%;
  transition: 1s;
}

@keyframes flash {
  0% {
    background-color: rgb(255, 90, 247);
  }
  25% {
    background-color: rgb(0, 247, 255);
  }
  70% {
    background-color: rgb(30, 255, 0);
  }
  100% {
    background-color: rgb(255, 90, 247);
  }
}

.theme-dark .vdatetime-popup__header,
.theme-dark .vdatetime-calendar__month__day--selected > span > span,
.theme-dark .vdatetime-calendar__month__day--selected:hover > span > span {
  background: #000;
}

.theme-dark .vdatetime-year-picker__item--selected,
.theme-dark .vdatetime-time-picker__item--selected,
.theme-dark .vdatetime-popup__actions__button {
  color: #000;
}

.content {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
}

.char {
  margin-bottom: 5rem;
}

#hint {
  font-size: 2rem;
}

.countdown {
  font-family: "Roboto Mono";
  font-weight: bold;
  width: 80vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 4rem;
  flex-direction: column;
}


@media only screen and (max-width: 600px){
  .countdown{
    font-size: 1.5rem;
  }
}




/* .times {
  width: 20%;
  flex: 1;
  padding: 1em;
} */
</style>
