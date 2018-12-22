<template>
  <div class="ui container">
    <p>hoge</p>
    <div class="webcam-ui-container">
      <div ref="webcamWrapper0" class="webcam-wrapper" style="display:flex;">
        <canvas ref="canvas0" width="416" height="416"/>
      </div>
      <div ref="webcamWrapper1" class="webcam-wrapper" style="display:flex;">
        <canvas ref="canvas1" width="416" height="416"/>
      </div>
      <div ref="webcamWrapper2" class="webcam-wrapper" style="display:flex;">
        <canvas ref="canvas2" width="416" height="416"/>
      </div>
    </div>
  </div>
</template>

<script>

  // import yolo, { downloadModel } from 'tfjs-yolo-tiny';
  // import yolo from './yolo';
  import yolo from 'tfjs-yolo';

  export default {
    name: 'app',
    data () {
      return {
        imgUrls: [
          "./data/cafe1.jpg",
          "./data/cafe2.jpg",
          "./data/cafe3.jpg"
        ],
        model: null,
        ctx: null
      }
    },
    async mounted(){
      const self = this;

      self.model = await yolo.v3tiny("./model_data/wada/model.json");
      self.ctx = [
        self.$refs.canvas0.getContext('2d'),
        self.$refs.canvas1.getContext('2d'),
        self.$refs.canvas2.getContext('2d')
      ]
      self.clearRects();

      self.imgUrls.forEach((imgUrl, i) => {
        const webcamElem = self.$refs["webcamWrapper" + i];
        const img = new Image();
        img.onload = function(){
          console.log("onload", img);
          console.log("h", img.height);
          console.log("w", img.width);
          self.exec(img, self.ctx[i], webcamElem);
        }
        img.src = imgUrl;
      });
    },
    methods:{
      async exec(img, ctx, webcamElem) {
        const self = this;
        const vw = img.width;
        const vh = img.height;

        ctx.drawImage(img, vw/2 - vh/2, 0, vh, vh, 0, 0, 416, 416);

        const boxes = await self.model.predict(ctx.canvas);

        // inputImage.dispose();

        console.log("boxes", boxes);

        boxes.forEach(box => {

          self.drawRect(webcamElem, box.left, box.top, box.width, box.height,
            `${box.class} Confidence: ${Math.round(box.score * 100)}%`)
        });
      },
      drawRect(webcamElem, x, y, w, h, text = '', color = 'red') {

        console.log("x,y,w,h",x, y, w, h);

        const rect = window.document.createElement('div');
        rect.classList.add('rect');
        rect.style.width = `${parseInt(w)}px`;
        rect.style.height = `${parseInt(h)}px`;
        rect.style.top = `${parseInt(y)}px`;
        rect.style.left = `${parseInt(x)}px`;
        rect.style["border-color"] = color;
        // rect.style.cssText = `border-color:${color}`;

        // const label = window.document.createElement('div');
        // label.classList.add('label');
        // label.innerText = text;
        // rect.appendChild(label);

        webcamElem.appendChild(rect);
      },
      clearRects() {
        const rects = window.document.getElementsByClassName('rect');
        while(rects[0]) {
          rects[0].parentNode.removeChild(rects[0]);
        }
      }
    }
  }
</script>

<style lang="css">
  .webcam-wrapper {
    border-radius: 5px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
    margin: 0 auto;
    width: 416px;
    height: 416px;
    position: relative;
    display: flex;
    justify-content: center;
    overflow: hidden;
  }
  .rect {
    position: absolute;
    border: 1px solid red;
    font-size: 24px;
  }
  .rect .label {
    background: white;
    color: black;
    display: inline;
    opacity: 0.8;
    font-size: 12px;
    padding: 3px;
    text-transform: capitalize;
    white-space: nowrap;
  }

</style>
