<template>
  <div class="main-wrapper">
    <div ref="header" class="header">
      <div ref="info" class="info">
        <div style="margin-left: 20px">关卡: {{stage}}</div>
        <div style="margin-right: 20px">得分: {{score}}</div>
      </div>
      <div ref="complete" class="complete">
        <div style="margin-right: 20px;overflow-y: auto;max-height: 100%;">{{complete}}</div>
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
    <div ref="modal" class="modal">
      <div class="title">{{text}}</div>
      <div v-if="showScores" style="width: 100%">
        <div class="score" v-for="s in scores" :key="s.stage">
          <div>{{s.stage}}</div>
          <div>{{s.score}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import "@/assets/common.css";
import arrow from "@/assets/arrow.svg";
import dict from "@/common/chengyu";

export default {
  name: "Main",
  data() {
    return {
      ctx: null,
      an: null,
      background: "#222",
      strokeStyle: "#000",
      fillStyle: "#000",
      fontFamily: "Georgia",
      fontStyle: "#666",
      fontHighLightStyle: "#baec40",
      fontComleteStyle: "#baec40",
      WIDTH: 0,
      HEIGHT: 0,
      WIDTH_UNIT: 0,
      HEIGHT_UNIT: 0,
      arrowSrc: arrow,
      showScores: true,
      score: 0,
      scores: [],
      xi: 0,
      yi: 0,
      complete: "",
      min: 14,
      dictJson: {},
      data: [],
      text: "成语消消乐",
      stage: 1,
      ALL: 108,
      lock: false,
      stages: [
        { X: 4, Y: 4, W: 4 },
        { X: 8, Y: 4, W: 8 },
        { X: 8, Y: 6, W: 12 },
        { X: 9, Y: 8, W: 18 },
        { X: 8, Y: 12, W: 24 },
        { X: 9, Y: 12, W: 108 }
      ],
      words: [],
      letters: [],
      lettersFirst: [],
      lettersFirstIndex: 0,
      completeWords: [],
      X: 0,
      Y: 3,
      W: 0
    };
  },
  mounted() {
    this.init();
  },
  computed: {},
  methods: {
    init() {
      this.initCanvas();
      this.initModal();
      this.initHeader();
      this.initController();
      this.initDict();
    },
    initDict() {
      let json = {};
      if (dict) {
        dict.forEach(d => {
          let key = d.substring(0, d.indexOf("拼"));
          let value = d.substring(d.lastIndexOf("拼") + 1);
          if (key.length == 4) {
            this.words.push(key);
            json[key] = value;
          }
        });
      }
      this.dictJson = json;
      this.words = this.getRandom(this.words);
    },
    initCanvas() {
      let canvas = this.$refs.canvas;
      this.ctx = canvas.getContext("2d");
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
      let fontSize = this.HEIGHT_UNIT * 0.8 + "px";
      this.ctx.font = `bold ${fontSize} ${this.fontFamily}`;
      return this.ctx;
    },
    initHeader() {
      let header = this.$refs.header;
      header.style.height = this.HEIGHT_UNIT * 2 + "px";
      let info = this.$refs.info;
      info.style.height = this.HEIGHT_UNIT + "px";
      info.style.fontSize = this.HEIGHT_UNIT * 0.4 + "px";
      let complete = this.$refs.complete;
      complete.style.height = this.HEIGHT_UNIT + "px";
      complete.style.fontSize = this.HEIGHT_UNIT * 0.3 + "px";
    },
    initController() {
      let controller = this.$refs.controller;
      controller.style.height = this.HEIGHT_UNIT * 2 + "px";
      controller.style.bottom = 0;

      let left = this.$refs.l;
      left.style.width = this.WIDTH_UNIT * 1.5 + "px";
      left.style.height = "100%";

      let right = this.$refs.r;
      right.style.width = this.WIDTH_UNIT * 1.5 + "px";
      right.style.height = "100%";
    },
    initModal() {
      let modal = this.$refs.modal;
      modal.style.height = this.HEIGHT_UNIT * 4 + "px";
      modal.style.top = this.HEIGHT_UNIT * 6 + "px";
      modal.style.fontSize = this.HEIGHT_UNIT * 0.8 + "px";
      setTimeout(_ => {
        this.text = 3;
        setTimeout(_ => {
          this.text = 2;
          setTimeout(_ => {
            this.text = 1;
            setTimeout(_ => {
              this.text = "开始!";
              setTimeout(() => {
                this.text = "";
                this.loadStage();
              }, 1000);
            }, 1000);
          }, 1000);
        }, 1000);
      }, 2000);
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
      this.text = "";
      this.complete = "";
      this.data = [];
      this.completeWords = [];
      this.score = 0;
      this.lettersFirstIndex = 0;
      let params = this.stages[this.stage - 1];
      this.X = params.X - 1;
      this.Y = params.Y + 2;
      this.W = 4 * params.W;
      this.letters = this.words.filter(
        (w, i) => i >= 14 * params.W && i < 15 * params.W
      );
      this.lettersFirst = this.letters.map(l => l && l.substring(0, 1));
      let letters = this.letters.join("").split("");
      letters = this.getRandom(letters);
      let i = 0;
      for (let x = 0; x < params.X; x++) {
        for (let y = 3; y < params.Y + 3; y++) {
          this.data.push({
            letter: letters[i],
            x: x,
            y: y
          });
          i++;
        }
      }
      this.getSelected();
      this.loadData();
    },
    getSelected() {
      if (this.lettersFirstIndex < this.lettersFirst.length) {
        let letter = this.lettersFirst[this.lettersFirstIndex];
        let l = this.data.filter(d => d.letter == letter)[0];
        this.xi = l.x;
        this.yi = l.y;
        this.lettersFirstIndex++;
      }
    },
    getRandom(arr) {
      var len = arr.length;
      //首先从最大的数开始遍历，之后递减
      for (var i = len - 1; i >= 0; i--) {
        //随机索引值randomIndex是从0-arr.length中随机抽取的
        var randomIndex = Math.floor(Math.random() * (i + 1));
        //下面三句相当于把从数组中随机抽取到的值与当前遍历的值互换位置
        var itemIndex = arr[randomIndex];
        arr[randomIndex] = arr[i];
        arr[i] = itemIndex;
      }
      //每一次的遍历都相当于把从数组中随机抽取（不重复）的一个元素放到数组的最后面（索引顺序为：len-1,len-2,len-3......0）
      return arr;
    },
    checkScore() {
      if (this.score < this.W) {
        let rows = this.data.filter(d => d.x >= this.xi && d.y == this.yi);
        let cols = this.data.filter(d => d.y >= this.yi && d.x == this.xi);
        if (this.checkWord(rows)) {
          this.doScore();
        } else if (this.checkWord(cols)) {
          this.doScore();
        }
      } else {
        this.doFinish();
      }
    },
    doScore() {
      setTimeout(_ => {
        this.getSelected();
        this.loadData();
      }, 1500);
    },
    doFinish() {
      if (this.stage < this.stages.length) {
        this.text = `第${this.stage}关通过!`;
        this.scores[this.stage - 1] = this.score;
        setTimeout(_ => {
          this.stage++;
          this.loadStage();
          this.text = "";
        }, 1500);
      } else {
        this.scores[this.stage - 1] = this.score;
        this.complete = "";
        this.text = `恭喜!`;
        setTimeout(_ => {
          let info = [];
          for (let i = 0; i < this.scores.length; i++) {
            info.push({
              stage: `第${i + 1}关`,
              score: `${this.scores[i]}分`
            });
          }
          this.scores = info;
          this.showScores = true;
          this.ctx.clearRect(0, 0, this.WIDTH, this.HEIGHT);
        }, 1000);
      }
    },
    checkWord(letters) {
      let isComplete;
      let word = letters
        .map(l => (l.letter && l.letter.toLowerCase()) || "1")
        .join("");
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
              d.letter = "";
            }
          }
        });
        this.lock = false;
        this.score += letters.length;
      }, 1000);
    },
    toLeft() {
      if (this.xi > 0 && !this.lock) {
        let selected = this.data.filter(
          d => d.x == this.xi && d.y == this.yi
        )[0];
        let target = this.data.filter(
          d => d.x == this.xi - 1 && d.y == this.yi
        )[0];
        selected.x--;
        target.x++;
        this.xi--;
        this.loadData();
      }
    },
    toRight() {
      if (this.xi < this.X && !this.lock) {
        let selected = this.data.filter(
          d => d.x == this.xi && d.y == this.yi
        )[0];
        let target = this.data.filter(
          d => d.x == this.xi + 1 && d.y == this.yi
        )[0];
        selected.x++;
        target.x--;
        this.xi++;
        this.loadData();
      }
    },
    toUp() {
      if (this.yi > 3 && !this.lock) {
        let selected = this.data.filter(
          d => d.x == this.xi && d.y == this.yi
        )[0];
        let target = this.data.filter(
          d => d.x == this.xi && d.y == this.yi - 1
        )[0];
        selected.y--;
        target.y++;
        this.yi--;
        this.loadData();
      }
    },
    toDown() {
      if (this.yi < this.Y && !this.lock) {
        let selected = this.data.filter(
          d => d.x == this.xi && d.y == this.yi
        )[0];
        let target = this.data.filter(
          d => d.x == this.xi && d.y == this.yi + 1
        )[0];
        selected.y++;
        target.y--;
        this.yi++;
        this.loadData();
      }
    }
  }
};
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
  color: #baec40;
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
  height: 25%;
}

.controller img:hover {
  transform: scale(1.2);
}

.controller .l .tl {
  transform: rotate(180deg);
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
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-family: "YaHei";
  color: #fff;
}

.modal .score {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-around;
  font-size: 28px;
}
</style>
