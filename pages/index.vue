<template>
  <section class="container" v-on:mousemove="detect_mouse">
    <div class="question chinese" id="questions">
      <h2>{{choose_title}}</h2>
      <div class="choose">
        <div id="boy" class="sexual" v-on:mouseover="playanim(true)" v-on:mouseleave="stopanim(true)" v-on:click="choose_sex(true)" v-show="is_choosesex">
          <div id="bodymovin"></div>
          <h3>男生</h3>
        </div>
        <div id="girl" class="sexual" v-on:mouseover="playanim(false)" v-on:mouseleave="stopanim(false)" v-on:click="choose_sex(false)" v-show="is_choosesex">
          <div id="bodymovin_2"></div>
          <h3>女生</h3>
        </div>
        <div class="birthday">

        </div>
      </div>
      <div class="btn" v-on:click="btn_continue">{{choose_btn}}</div>
    </div>
    <div id="banner"></div>
    <div class="up">
      <button class="chinese btn btn-start" v-on:mouseover="offset_to_zero" v-on:click="start">開始</button>
    </div>
    <div class="logo" id="logos">Doraralab</div>
  </section>
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
      mousevalue: 0,
      title_size: 80,
      offset_X: 0,
      offset_Y: 0,
      myscope: null,
      is_active: true,
      fig1: null,
      fig2: null,
      padding: [400, 400, 300, 450],
      is_mobile: 1,
      user: {
        sex: "",
        birth: ""
      },
      is_choosesex: true,
      choose_btn: "繼續",
      choose_title: "你是....?"
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
        }
      ]
    };
  },
  mounted() {
    this.detect_screen();
    this.blotter();
    if (process.browser) {
      this.addfigure();
    }
  },
  methods: {
    btn_continue: function() {
      if(this.user.sex == "") {
        alert("先選性別")
        return
      }

      if(!this.is_choosesex){
        this.choose_btn = "繼續"
        this.choose_title = "你是....?"
        this.start()
      }
      this.is_choosesex = false
      this.choose_btn = "完成"
      this.choose_title = "生日"
    },
    choose_birthday: function(){

    },
    choose_sex: function(t){
      if(t){
        this.user.sex = "boy"
        document.getElementById('boy').style.backgroundColor = "rgba(255, 203, 85, 0.9)"
        document.getElementById('girl').style.backgroundColor = "white"
      } else {
        this.user.sex = "girl"
        document.getElementById('boy').style.backgroundColor = "white"
        document.getElementById('girl').style.backgroundColor = "rgba(255, 203, 85, 0.9)"
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
        this.is_choosesex = false
      } else {
        document.getElementById("logos").style.color = "white";
        document.body.style.backgroundColor = "black";
        document.getElementById("questions").style.display = "flex";
        this.is_active = !this.is_active;
        this.is_choosesex = true
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
      } else {
        this.is_mobile = 1;
        this.title_size = 100;
      }
    },
    angleBetweenPointsInDegrees: function(x, y) {
      var angle = (Math.atan2(y - 0.5, x - 0.5) * 180.0) / Math.PI;
      angle = 180 + angle;

      return angle;
    },
    distanceBetweenPoints: function(x, y) {
      var a = 0.5 - x;
      var b = 0.5 - y;

      return Math.sqrt(a * a + b * b);
    },
    offset_to_zero: function() {
      this.myscope.material.uniforms.uRotation.value = 45;
      this.myscope.material.uniforms.uOffset.value = 0;
    },
    detect_mouse: function() {
      var x = (event.clientX * 100) / window.innerWidth;
      var y = (event.clientY * 100) / window.innerHeight;
      this.offset_X = x;
      this.offset_Y = y;
    },
    blotter: function() {
      // https://blotter.js.org
      // Define text style
      const text = new Blotter.Text("人生倒計時", {
        family: "'Kosugi Maru', 'Source Sans Pro'",
        size: this.title_size,
        paddingLeft: 400 * this.is_mobile,
        paddingRight: 400 * this.is_mobile,
        paddingTop: 300 * this.is_mobile,
        paddingBottom: 300 * this.is_mobile + 50
      });

      // Use a material
      // https://blotter.js.org/#/materials
      // Channel Split Material
      let material = new Blotter.ChannelSplitMaterial();

      // Set material opts
      material.uniforms.uOffset.value = 0.05;
      material.uniforms.uRotation.value = 45;
      material.uniforms.uApplyBlur.value = 1;
      material.uniforms.uAnimateNoise.value = 1;

      let blotter = new Blotter(material, {
        texts: text
      });
      // Apply to element
      let myScope = blotter.forText(text);
      let elem = document.getElementById("banner");
      let vm = this;
      myScope.appendTo(elem);
      this.myscope = myScope;
      myScope.on("mousemove", function(mousePosition) {
        var angle = vm.angleBetweenPointsInDegrees(
          mousePosition.x,
          mousePosition.y
        );
        var blur = Math.min(
          0.07,
          vm.distanceBetweenPoints(mousePosition.x, mousePosition.y)
        );
        myScope.material.uniforms.uRotation.value = angle;
        myScope.material.uniforms.uOffset.value = blur;
        console.log("--", mousePosition.x, mousePosition.y);
      });

      // elem.addEventListener("mousemove", function(mousePosition) {
      //   var angle = vm.angleBetweenPointsInDegrees(
      //     mousePosition.x.toFixed(1) / 100,
      //     mousePosition.y.toFixed(1) / 100
      //   );
      //   var blur = Math.min(
      //     0.04,
      //     vm.distanceBetweenPoints(
      //       mousePosition.x.toFixed(1) / 100,
      //       mousePosition.y.toFixed(1) / 100
      //     )
      //   );
      //   myScope.material.uniforms.uRotation.value = angle;
      //   myScope.material.uniforms.uOffset.value = blur;
      //   console.log("++", mousePosition.x, mousePosition.y);
      // });
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
  margin-top: 10rem;
  padding: 10px 20px;
  cursor: pointer;
  animation: flash 3s infinite;
}

.question {
  background-color: #fff;
  width: 30%;
  height: 60%;
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
}

.sexual {
  cursor: pointer;
}

.sexual:hover {
  background-color: rgba(255, 203, 85, 0.9);
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
</style>
