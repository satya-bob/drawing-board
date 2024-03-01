<template>
  <div class="container">
    <section class="tools-board">
      <!-- Shapes -->
      <div class="row">
        <label class="title">Shapes</label>
        <ul class="options">
          <li v-for="shape in shapes" :key="shape.id" class="option tool" :id="shape.id" @click="selectTool(shape.id)">
            <img :src="shape.icon" :alt="shape.name">
            <span>{{ shape.name }}</span>
          </li>
          <li class="option">
            <input type="checkbox" id="fill-color" v-model="fillColor">
            <label for="fill-color">Fill color</label>
          </li>
        </ul>
      </div>

      <!-- Options -->
      <div class="row">
        <label class="title">Options</label>
        <ul class="options">
          <li v-for="option in options" :key="option.id" :class="{ 'active': option.id === selectedTool }" class="option tool" :id="option.id" @click="selectTool(option.id)">
            <img :src="option.icon" :alt="option.name">
            <span>{{ option.name }}</span>
          </li>
          <li class="option">
            <input type="range" id="size-slider" min="1" max="30" v-model="brushWidth">
          </li>
        </ul>
      </div>

      <!-- Colors -->
      <div class="row colors">
        <label class="title">Colors</label>
        <ul class="options">
          <li v-for="(color, index) in colors" :key="index" class="option" :style="{ backgroundColor: color }" @click="selectColor(color)"></li>
          <li class="option">
            <input type="color" id="color-picker" v-model="selectedColor">
          </li>
        </ul>
      </div>

      <!-- Buttons -->
      <div class="row buttons">
        <button class="clear-canvas" @click="clearCanvas">Clear Canvas</button>
        <button class="save-img" @click="saveImage">Save As Image</button>
      </div>
    </section>
    <div class="line"></div>

    <!-- Drawing Board -->
    <section class="drawing-board">
      <canvas ref="canvas"></canvas>
    </section>
  </div>
</template>


<script>
export default {
  data() {
    return {
      brushWidth: 5,
      selectedColor: '#000',
      fillColor: false,
      colors: ['#000', '#ff0000', '#00ff00', '#0000ff'],
      selectedTool: 'brush',
      isDrawing: false,
      prevMouseX: null,
      prevMouseY: null,
      canvas: null,
      ctx: null,
      shapes: [
        { id: 'rectangle', name: 'Rectangle', icon: 'icons/rectangle.svg' },
        { id: 'circle', name: 'Circle', icon: 'icons/circle.svg' },
        { id: 'triangle', name: 'Triangle', icon: 'icons/triangle.svg' }
      ],
      options: [
        { id: 'brush', name: 'Brush', icon: 'icons/brush.svg' },
        { id: 'eraser', name: 'Eraser', icon: 'icons/eraser.svg' }
      ]
    };
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.ctx = this.canvas.getContext('2d');
    this.initCanvas();
    this.addEventListeners();
  },
  methods: {
    initCanvas() {
      const rect = this.canvas.getBoundingClientRect();
      this.canvas.width = rect.width;
      this.canvas.height = rect.height;
      this.setCanvasBackground();
    },
    setCanvasBackground() {
      this.ctx.fillStyle = '#fff';
      this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
      this.ctx.fillStyle = this.selectedColor;
    },
    drawRect(e) {
      if (!this.isDrawing) return;
      this.ctx.putImageData(this.snapshot, 0, 0);
      this.ctx.strokeStyle = this.selectedColor;
      this.ctx.strokeRect(e.offsetX, e.offsetY, this.prevMouseX - e.offsetX, this.prevMouseY - e.offsetY);
    },
    drawCircle(e) {
      if (!this.isDrawing) return;
      this.ctx.putImageData(this.snapshot, 0, 0);
      this.ctx.strokeStyle = this.selectedColor;
      const radius = Math.sqrt(Math.pow((this.prevMouseX - e.offsetX), 2) + Math.pow((this.prevMouseY - e.offsetY), 2));
      this.ctx.beginPath();
      this.ctx.arc(this.prevMouseX, this.prevMouseY, radius, 0, 2 * Math.PI);
      this.ctx.stroke();
    },
    drawTriangle(e) {
      if (!this.isDrawing) return;
      this.ctx.putImageData(this.snapshot, 0, 0);
      this.ctx.strokeStyle = this.selectedColor;
      this.ctx.beginPath();
      this.ctx.moveTo(this.prevMouseX, this.prevMouseY);
      this.ctx.lineTo(e.offsetX, e.offsetY);
      this.ctx.lineTo(this.prevMouseX * 2 - e.offsetX, e.offsetY);
      this.ctx.closePath();
      this.ctx.stroke();
    },
    startDraw(e) {
      this.isDrawing = true;
      this.prevMouseX = e.offsetX;
      this.prevMouseY = e.offsetY;
      this.ctx.lineWidth = this.brushWidth;
      this.snapshot = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    },
    drawing(e) {
  if (!this.isDrawing) return; // if isDrawing is false return from here
  this.ctx.putImageData(this.snapshot, 0, 0); // adding copied canvas data on to this canvas
  if (this.selectedTool === "brush" || this.selectedTool === "eraser") {
    // if selected tool is eraser then set strokeStyle to white 
    // to paint white color on to the existing canvas content else set the stroke color to selected color
    this.ctx.strokeStyle = this.selectedTool === "eraser" ? "#fff" : this.selectedColor;
    this.ctx.lineTo(e.offsetX, e.offsetY); // creating line according to the mouse pointer
    this.ctx.stroke(); // drawing/filling line with color
  } else if (this.selectedTool === "rectangle") {
    this.drawRect(e);
  } else if (this.selectedTool === "circle") {
    this.drawCircle(e);
  } else {
    this.drawTriangle(e);
  }
},


    addEventListeners() {
      this.canvas.addEventListener('mousedown', this.startDraw);
      this.canvas.addEventListener('mousemove', this.drawing);
      this.canvas.addEventListener('mouseup', () => this.isDrawing = false);
    },
    selectColor(color) {
      this.selectedColor = color;
    },
    selectTool(tool) {
      this.selectedTool = tool;
    },
    clearCanvas() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.setCanvasBackground();
    },
    saveImage() {
      const link = document.createElement('a');
      link.download = `${Date.now()}.png`;
      link.href = this.canvas.toDataURL();
      link.click();
    }
  }
};
</script>





<style scoped>
/* Import Google font - Poppins */
/* Import Google font - Poppins */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&display=swap');
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Poppins', sans-serif;
}
body{
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: #4A98F7;
}
.container{
  display: flex;
  width: 100%;
  gap: 10px;
  padding: 10px;
  max-width: 1050px;
}
section{
  background: #fff;
  border-radius: 7px;
}
.tools-board{
  width: 210px;
  padding: 15px 22px 0;
}
.tools-board .row{
  margin-bottom: 20px;
}
.row .options{
  list-style: none;
  margin: 10px 0 0 5px;
}
.row .options .option{
  display: flex;
  cursor: pointer;
  align-items: center;
  margin-bottom: 10px;
}
.option:is(:hover, .active) img{
  filter: invert(17%) sepia(90%) saturate(3000%) hue-rotate(900deg) brightness(100%) contrast(100%);
}
.option :where(span, label){
  color: #5A6168;
  cursor: pointer;
  padding-left: 10px;
}
.option:is(:hover, .active) :where(span, label){
  color: #4A98F7;
}
.option #fill-color{
  cursor: pointer;
  height: 14px;
  width: 14px;
}
#fill-color:checked ~ label{
  color: #4A98F7;
}
.option #size-slider{
  width: 100%;
  height: 5px;
  margin-top: 10px;
}
.colors .options{
  display: flex;
  justify-content: space-between;
}
.colors .option{
  height: 20px;
  width: 20px;
  border-radius: 50%;
  margin-top: 3px;
  position: relative;
}
.colors .option:nth-child(1){
  background-color: #fff;
  border: 1px solid #bfbfbf;
}
.colors .option:nth-child(2){
  background-color: #000;
}
.colors .option:nth-child(3){
  background-color: #E02020;
}
.colors .option:nth-child(4){
  background-color: #6DD400;
}
.colors .option:nth-child(5){
  background-color: #4A98F7;
}
.colors .option.selected::before{
  position: absolute;
  content: "";
  top: 50%;
  left: 50%;
  height: 12px;
  width: 12px;
  background: inherit;
  border-radius: inherit;
  border: 2px solid #fff;
  transform: translate(-50%, -50%);
}
.colors .option:first-child.selected::before{
  border-color: #ccc;
}
.option #color-picker{
  opacity: 0;
  cursor: pointer;
}
.buttons button{
  width: 100%;
  color: #fff;
  border: none;
  outline: none;
  padding: 11px 0;
  font-size: 0.9rem;
  margin-bottom: 13px;
  background: none;
  border-radius: 4px;
  cursor: pointer;
}
.buttons .clear-canvas{
  color: #6C757D;
  border: 1px solid #6C757D;
  transition: all 0.3s ease;
}
.clear-canvas:hover{
  color: #fff;
  background: #6C757D;
}
.buttons .save-img{
  background: #4A98F7;
  border: 1px solid #4A98F7;
}
.drawing-board{
  flex: 1;
  overflow: hidden;
}
.drawing-board canvas{
  width: 100%;
  height: 100%;
}
.line {
  width: 1px; /* Adjust the width of the line */
  height: calc(100% - 30px); /* Adjust the height of the line (considering padding) */
  background-color: #ccc; /* Adjust the color of the line */
  margin: 15px auto; /* Adjust the position of the line */
}

</style>