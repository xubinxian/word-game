<template>
  <div class="main-wrapper">
    <div ref="header" class="header">
      <div ref="info" class="info">
        <div style="margin-left: 20px">LETTER: {{letter}}</div>
        <div style="margin-right: 20px">SCORE: {{score}}</div>
      </div>
      <div ref="complete" class="complete">{{complete}}</div>
    </div>
    <canvas ref="canvas">Ah! Are you ok ?</canvas>
    <div ref="controller" class="controller">
      <div ref="l" class="l">
        <img :src="imgSrc" @click="toLeft" />
      </div>
      <div ref="r" class="r">
        <img :src="imgSrc" @click="toRight" />
      </div>
    </div>
    <div ref="modal" class="modal">{{text}}</div>
  </div>
</template>

<script>

import '@/assets/common.css'
import img from '@/assets/arrow.svg'
import dict from '@/common/dict';

export default {
  name: 'Main',
  data() {
    return {
      ctx: null,
      an: null,
      background: '#233',
      strokeStyle: '#000',
      fillStyle: '#000',
      fontFamily: 'Georgia',
      fontStyle: '#fff',
      WIDTH: 0,
      HEIGHT: 0,
      WIDTH_UNIT: 0,
      HEIGHT_UNIT: 0,
      imgSrc: img,
      word: 'BUT',
      score: 0,
      xi: 0,
      yi: 2,
      letter: 'A',
      complete: "",
      text: "3",
      min: 14,
      data: [],
      letters: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']
    }
  },
  mounted() {
    this.init();
  },
  computed: {
    dictJson() {
      let json = {};
      if (dict) {
        dict.forEach(d => {
          let key = d.substring(0, d.indexOf(' '));
          let value = d.substring(d.lastIndexOf(' ') + 1);
          if (key.length <= 5) {
            json[key] = value;
          }
        });
      }
      return json;
    }
  },
  methods: {
    init() {
      this.fullscreen();
      this.initCanvas();
      this.initModal();
      this.initHeader();
      this.initController();
      this.initWord();
    },
    fullscreen() {
      var el = document.documentElement;
      var rfs = el.requestFullScreen || el.webkitRequestFullScreen || el.mozRequestFullScreen || el.msRequestFullScreen;

      //typeof rfs != "undefined" && rfs
      if (rfs) {
        rfs.call(el);
      }
      else if (typeof window.ActiveXObject !== "undefined") {
        //for IE，这里其实就是模拟了按下键盘的F11，使浏览器全屏
        var wscript = new ActiveXObject("WScript.Shell");
        if (wscript != null) {
          wscript.SendKeys("{F11}");
        }
      }
    },
    initCanvas() {
      let canvas = this.$refs.canvas;
      this.ctx = canvas.getContext('2d');
      canvas.style.background = this.background;
      this.WIDTH = screen.availWidth;
      this.HEIGHT = screen.availHeight;
      canvas.width = this.WIDTH;
      canvas.height = this.HEIGHT;
      console.log('WIDTH', this.WIDTH, 'HEIGHT', this.HEIGHT);
      this.WIDTH_UNIT = this.WIDTH / 9;
      this.HEIGHT_UNIT = this.HEIGHT / 16;
      console.log('WIDTH_UNIT', this.WIDTH_UNIT, 'HEIGHT_UNIT', this.HEIGHT_UNIT);
    },
    initContext() {
      this.ctx.strokeStyle = this.strokeStyle;
      this.ctx.fillStyle = this.fillStyle;
      let fontSize = this.HEIGHT_UNIT * 0.8 + 'px';
      this.ctx.font = `bold ${fontSize} ${this.fontFamily}`;
      return this.ctx;
    },
    initHeader() {
      let header = this.$refs.header;
      header.style.height = this.HEIGHT_UNIT * 2 + 'px';
      let info = this.$refs.info;
      info.style.height = this.HEIGHT_UNIT + 'px';
      info.style.fontSize = this.HEIGHT_UNIT * 0.6 + 'px';
      let complete = this.$refs.complete;
      complete.style.height = this.HEIGHT_UNIT + 'px';
      complete.style.fontSize = this.HEIGHT_UNIT * 0.6 + 'px';
      let modal = this.$refs.modal;
      modal.style.height = this.HEIGHT_UNIT + 'px';
      modal.style.fontSize = this.HEIGHT_UNIT * 0.8 + 'px';
    },
    initController() {
      let controller = this.$refs.controller;
      controller.style.height = this.HEIGHT_UNIT * 2 + 'px';
      controller.style.bottom = 0;

      let left = this.$refs.l;
      left.style.width = this.WIDTH_UNIT * 1.5 + 'px';
      left.style.height = '100%';
      left.style.marginLeft = this.WIDTH_UNIT + 'px';

      let right = this.$refs.r;
      right.style.width = this.WIDTH_UNIT * 1.5 + 'px';
      right.style.height = '100%';
      right.style.marginRight = this.WIDTH_UNIT + 'px';
    },
    initModal() {
      let modal = this.$refs.modal;
      modal.style.height = this.HEIGHT_UNIT * 4 + 'px';
      modal.style.top = this.HEIGHT_UNIT * 6 + 'px';
      setTimeout(_ => {
        this.text = 2
        setTimeout(_ => {
          this.text = 1
          setTimeout(_ => {
            this.text = 'GO!'
            setTimeout(() => {
              this.text = '';
              this.animate();
            }, 1000);
          }, 1000);
        }, 1000);
      }, 1000);
    },
    initWord(xi, yi, letter) {
      let ctx = this.initContext();
      ctx.fillStyle = this.fontStyle;
      xi = xi || this.xi;
      yi = yi || this.yi;
      letter = letter || this.letter;
      ctx.fillText(letter, this.WIDTH_UNIT * xi, this.HEIGHT_UNIT * yi);
    },
    loadData() {
      this.data.forEach(d => {
        this.initWord(d.x, d.y, d.letter);
      });
    },
    checkCollision() {
      let isCollison;
      for (let d of this.data) {
        if (this.xi == d.x && this.yi + 1 == d.y) {
          isCollison = true;
          break;
        }
      }
      if (!isCollison) {
        if (this.yi >= 14) {
          isCollison = true;
        }
      }
      return isCollison;
    },
    checkScore() {
      for (let i = parseInt(this.min); i <= 14; i++) {
        let letters = this.data.filter(d => d.y == i);
        for (let j = 0; j <= 8; j++) {
          if (!letters.some(l => l.x == j)) {
            letters.splice(j, 1, {
              letter: '1',
              x: i
            });
          }
        }
        let isComplete = this.checkWord(letters);
        if (isComplete) {
          this.score += isComplete.length;
          this.clearCompleteWord(isComplete);
        }
      }
    },
    checkWord(letters) {
      let isComplete;
      let word = letters.map(l => l.letter.toLowerCase()).join('');
      for (let key in this.dictJson) {
        let index = word.indexOf(key);
        if (index > -1) {
          isComplete = letters.slice().splice(index, key.length);
          this.complete = `${key} ${this.dictJson[key]}`;
          break;
        }
      }
      return isComplete;
    },
    clearCompleteWord(letters) {
      let firstLetter = letters[0];
      let index = -1;
      for (let i = 0; i < this.data.length; i++) {
        let d = this.data[i];
        if (d.letter == firstLetter.letter && d.x == firstLetter.x && d.y == firstLetter.y) {
          index = i;
          break;
        }
      }
      if (index > -1) {
        this.data.splice(index, letters.length);
      }
    },
    toLeft() {
      if (!this.checkCollision()) {
        this.xi = this.xi > 0 ? this.xi - 1 : 0;
      }
    },
    toRight() {
      if (!this.checkCollision()) {
        this.xi = this.xi < 8 ? this.xi + 1 : 8;
      }
    },
    animate() {
      this.an = setInterval(this.render, 800);
      this.render();
    },
    render() {
      this.ctx.clearRect(0, 0, this.WIDTH, this.HEIGHT);
      if (!this.checkCollision()) {
        this.initWord(this.xi, this.yi++);
      } else {
        this.data.push({
          letter: this.letter + '',
          x: this.xi + '',
          y: this.yi + ''
        });
        this.min = this.min > this.yi ? this.yi + '' : this.min;
        console.log(this.min);
        this.checkScore();
        if (this.yi != 3) {
          this.letter = this.letters[Math.floor(this.letters.length * Math.random())];
          this.xi = Math.floor(9 * Math.random());
          this.yi = 3;
          this.initWord();
        } else {
          this.text = 'Game Over!';
          clearInterval(this.an);
        }
      }
      this.loadData();
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.main-wrapper {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.header {
  position: fixed;
  top: 0;
  width: 100%;
  background: #000;
  color: #fff;
}
.header .info,
.header .complete {
  width: 100%;
  display: flex;
  align-items: center;
}
.header .info {
  justify-content: space-between;
  font-weight: bold;
  font-family: "YaHei";
}
.header .complete {
  font-family: "YaHei";
  color: #4b5cc4;
}
.controller {
  position: fixed;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #000;
}

.controller div {
  display: flex;
  align-items: center;
}

.controller img {
  width: 50%;
}

.controller img:hover {
  transform: scale(1.05);
}

.controller .l {
  transform: rotate(180deg);
}

.modal {
  position: fixed;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-family: "YaHei";
  color: #fff;
}
</style>
