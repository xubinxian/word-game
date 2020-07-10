<template>
  <div class="main-wrapper">
    <div ref="header" class="header">
      <div ref="info" class="info">
        <div style="margin-left: 20px">STAGE: {{stage}}</div>
        <div style="margin-right: 20px">SCORE: {{score}}</div>
      </div>
      <div ref="complete" class="complete">
        <div style="margin-left: 20px"></div>
        <div style="margin-right: 20px">{{complete}}</div>
      </div>
    </div>
    <canvas ref="canvas">Ah! Are you ok ?</canvas>
    <div ref="controller" class="controller">
      <div ref="l" class="l">
        <img class="tl" :src="arrowSrc" @click="toLeft" />
        <img class="tr" :src="arrowSrc" @click="toRight" />
      </div>
      <div ref="r" class="r">
        <img class="tu" :src="arrowSrc" @click="toUp" />
        <img class="td" :src="arrowSrc" @click="toDown" />
      </div>
    </div>
    <div ref="modal" class="modal">{{text}}</div>
  </div>
</template>

<script>

import '@/assets/common.css'
import arrow from '@/assets/arrow.svg'
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
      fontStyle: '#666',
      fontHighLightStyle: '#fff',
      fontComleteStyle: '#ff0',
      WIDTH: 0,
      HEIGHT: 0,
      WIDTH_UNIT: 0,
      HEIGHT_UNIT: 0,
      arrowSrc: arrow,
      score: 0,
      scores: [],
      xi: 0,
      yi: 0,
      complete: "",
      text: "3",
      min: 14,
      data: [],
      stage: 1,
      total: 0,
      ALL: 108,
      lock: false,
      stages: [
        ['A', 'D', 'S'],
        ['A', 'B', 'T', 'O'],
        ['C', 'O', 'S', 'T', 'A'],
        ['N', 'B', 'W', 'S', 'K', 'E']
      ],
      completeWords: [],
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
          if (key.length <= 12 && key.length > 1) {
            json[key] = value;
          }
        });
      }
      return json;
    }
  },
  methods: {
    init() {
      this.initCanvas();
      this.initModal();
      this.initHeader();
      this.initController();
    },
    initCanvas() {
      let canvas = this.$refs.canvas;
      this.ctx = canvas.getContext('2d');
      canvas.style.background = this.background;
      this.WIDTH = document.body.clientWidth;
      this.HEIGHT = document.body.clientHeight;
      canvas.width = this.WIDTH;
      canvas.height = this.HEIGHT;
      this.WIDTH_UNIT = this.WIDTH / 9;
      this.HEIGHT_UNIT = this.HEIGHT / 16;
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
    },
    initController() {
      let controller = this.$refs.controller;
      controller.style.height = this.HEIGHT_UNIT * 2 + 'px';
      controller.style.bottom = 0;

      let left = this.$refs.l;
      left.style.width = this.WIDTH_UNIT * 1.5 + 'px';
      left.style.height = '100%';

      let right = this.$refs.r;
      right.style.width = this.WIDTH_UNIT * 1.5 + 'px';
      right.style.height = '100%';
    },
    initModal() {
      let modal = this.$refs.modal;
      modal.style.height = this.HEIGHT_UNIT * 4 + 'px';
      modal.style.top = this.HEIGHT_UNIT * 6 + 'px';
      modal.style.fontSize = this.HEIGHT_UNIT * 0.8 + 'px';
      modal.style.background = 'rgba(0,0,0,.25)';
      setTimeout(_ => {
        this.text = 2
        setTimeout(_ => {
          this.text = 1
          setTimeout(_ => {
            this.text = 'GO!'
            setTimeout(() => {
              this.text = '';
              modal.style.background = 'rgba(0,0,0,0)';
              this.loadStage();
            }, 1000);
          }, 1000);
        }, 1000);
      }, 1000);
    },
    drawWord(xi, yi, letter, fontStyle) {
      let ctx = this.initContext();
      ctx.fillStyle = fontStyle || this.fontStyle;
      ctx.fillText(letter, this.WIDTH_UNIT * xi, this.HEIGHT_UNIT * yi);
    },
    loadData(notCheck) {
      this.ctx.clearRect(0, 0, this.WIDTH, this.HEIGHT);
      this.data.sort((d1, d2) => d1.x - d2.x).sort((d1, d2) => d1.y - d2.y);
      this.data.forEach(d => {
        let filter = this.completeWords.filter(w => w.x == d.x && w.y == d.y);
        if (filter.length) {
          this.drawWord(d.x, d.y, d.letter, this.fontComleteStyle);
        } else if (d.x == this.xi && d.y == this.yi) {
          this.drawWord(d.x, d.y, d.letter, this.fontHighLightStyle);
        } else {
          this.drawWord(d.x, d.y, d.letter);
        }
      });
      !notCheck && this.checkScore();
    },
    loadStage() {
      this.score = 0;
      this.complete = '';
      this.data = [];
      this.completeWords = [];
      this.scores[this.stage - 0] = this.score;
      this.total = parseInt(this.ALL + '');
      this.xi = Math.floor(9 * Math.random());
      this.yi = 2 + Math.floor(13 * Math.random());
      for (let x = 0; x <= 8; x++) {
        for (let y = 3; y <= 14; y++) {
          let letters = this.stages[this.stage - 1];
          this.data.push({
            letter: letters[Math.floor(letters.length * Math.random())],
            x: x,
            y: y
          });
        }
      }
      this.loadData();
    },
    checkScore() {
      let rows = this.data.filter(d => d.x >= this.xi && d.y == this.yi);
      let isComplete = this.checkWord(rows);
      if (isComplete) {
        if (this.total >= this.stage * 20) {
          setTimeout(_ => {
            let letter = this.data.filter(d => d.letter)[0];
            this.xi = letter.x;
            this.yi = letter.y;
            this.loadData();
          }, 1500);
        } else {
          this.$refs.modal.style.background = 'rgba(0,0,0,.25)';
          if (this.stage < this.stages.length - 1) {
            this.text = `Stage ${this.stage} clear!`;
            setTimeout(_ => {
              this.stage++;
              this.loadStage();
              this.text = '';
              this.$refs.modal.style.background = 'rgba(0,0,0,.0)';
            }, 1500);
          } else {
            this.text = `All stages clear!`;
          }
        }
      } else {
        let cols = this.data.filter(d => d.y >= this.yi && d.x == this.xi);
        isComplete = this.checkWord(cols);
        if (isComplete) {
          if (this.total >= this.ALL * this.stage / 5) {
            setTimeout(_ => {
              let letter = this.data.filter(d => d.letter)[0];
              this.xi = letter.x;
              this.yi = letter.y;
              this.loadData();
            }, 1500);
          } else {
            this.$refs.modal.style.background = 'rgba(0,0,0,.25)';
            if (this.stage < this.stages.length - 1) {
              this.text = `Stage ${this.stage} clear!`;
              setTimeout(_ => {
                this.stage++;
                this.loadStage();
                this.text = '';
                this.$refs.modal.style.background = 'rgba(0,0,0,.0)';
              }, 1500);
            } else {
              this.text = `All stages clear!`;
            }
          }
        }
      }
    },
    checkWord(letters) {
      let isComplete;
      let word = letters.map(l => l.letter && l.letter.toLowerCase() || '1').join('');
      for (let key in this.dictJson) {
        let index = word.indexOf(key);
        if (index == 0) {
          this.completeWords = isComplete = letters.splice(index, key.length);
          this.loadData(true);
          this.complete = `${key} ${this.dictJson[key]}`;
          this.clearCompleteWord(isComplete);
          break;
        }
      }
      return isComplete;
    },
    clearCompleteWord(letters) {
      this.lock = true;
      setTimeout(_ => {
        letters.forEach(l => {
          for (let i = 0; i < this.data.length; i++) {
            let d = this.data[i];
            if (d.letter == l.letter && d.x == l.x && d.y == l.y) {
              d.letter = '';
              this.total--;
            }
          }
        });
        this.lock = false;
        this.score += letters.length;
        console.log('TOTAL', this.total);
      }, 1000);
    },
    toLeft() {
      if (this.xi > 0 && !this.lock) {
        let selected = this.data.filter(d => d.x == this.xi && d.y == this.yi)[0];
        let target = this.data.filter(d => d.x == this.xi - 1 && d.y == this.yi)[0];
        selected.x--;
        target.x++;
        this.xi--;
        this.loadData();
      }
    },
    toRight() {
      if (this.xi < 8 && !this.lock) {
        let selected = this.data.filter(d => d.x == this.xi && d.y == this.yi)[0];
        let target = this.data.filter(d => d.x == this.xi + 1 && d.y == this.yi)[0];
        selected.x++;
        target.x--;
        this.xi++;
        this.loadData();
      }
    },
    toUp() {
      if (this.yi > 3 && !this.lock) {
        let selected = this.data.filter(d => d.x == this.xi && d.y == this.yi)[0];
        let target = this.data.filter(d => d.x == this.xi && d.y == this.yi - 1)[0];
        selected.y--;
        target.y++;
        this.yi--;
        this.loadData();
      }
    },
    toDown() {
      if (this.yi < 14 && !this.lock) {
        let selected = this.data.filter(d => d.x == this.xi && d.y == this.yi)[0];
        let target = this.data.filter(d => d.x == this.xi && d.y == this.yi + 1)[0];
        selected.y++;
        target.y--;
        this.yi++;
        this.loadData();
      }
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
  flex: 1;
}

.controller img {
  flex: 1;
  height: 50%;
}

.controller img:hover {
  transform: scale(1.2);
}

.controller .l .tl {
  transform: rotate(180deg);
}

.controller .r {
  flex-direction: column;
}

.controller .r .tu {
  transform: rotate(-90deg);
}

.controller .r .td {
  transform: rotate(90deg);
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
