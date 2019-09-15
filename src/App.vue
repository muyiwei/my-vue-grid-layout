<template>
  <div id="app">
    <h1 style="text-align: center">Vue Grid Layout</h1>
    <!--<pre>{{ layout | json }}</pre>-->
    <div>
      <div class="layoutJSON">
        Displayed as
        <code>[x, y, w, h]</code>:
        <div class="columns">
          <div class="layoutItem" v-for="item in layout" :key="item.i">
            <b>{{item.i}}</b>
            : [{{item.x}}, {{item.y}}, {{item.w}}, {{item.h}}]
          </div>
        </div>
      </div>
      <!--<div class="layoutJSON">
                Displayed as <code>[x, y, w, h]</code>:
                <div class="columns">
                    <div class="layoutItem" v-for="item in layout2">
                        <b>{{item.i}}</b>: [{{item.x}}, {{item.y}}, {{item.w}}, {{item.h}}]
                    </div>
                </div>
      </div>-->
    </div>

    <div>
      <span draggable="true" @dragstart="dragstart($event)" @dragend="dragend($event)">拖动到图中</span>
    </div>

    <div id="content">
      <button @click="decreaseWidth">Decrease Width</button>
      <button @click="increaseWidth">Increase Width</button>
      <button @click="addItem">Add an item</button>
      <!-- Add to show rtl support -->
      <button @click="changeDirection">Change Direction</button>
      <input type="checkbox" v-model="draggable" /> Draggable
      <input type="checkbox" v-model="resizable" /> Resizable
      <input type="checkbox" v-model="mirrored" /> Mirrored
      <input type="checkbox" v-model="responsive" /> Responsive
      <input type="checkbox" v-model="preventCollision" /> Prevent Collision
      <div style="margin-top: 10px;margin-bottom: 10px;">
        Row Height:
        <input type="number" v-model="rowHeight" /> Col nums:
        <input type="number" v-model="colNum" />
      </div>
      <div @dragover="dragover($event)" @dragleave="dragleave($event)" @drop="drop($event)">
        <grid-layout
           ref="gridLayout"
          :layout.sync="layout"
          :col-num="parseInt(colNum)"
          :row-height="rowHeight"
          :is-draggable="draggable"
          :is-resizable="resizable"
          :is-mirrored="mirrored"
          :prevent-collision="preventCollision"
          :vertical-compact="true"
          :use-css-transforms="true"
          :responsive="responsive"
          @layout-created="layoutCreatedEvent"
          @layout-before-mount="layoutBeforeMountEvent"
          @layout-mounted="layoutMountedEvent"
          @layout-ready="layoutReadyEvent"
          @layout-updated="layoutUpdatedEvent"
        >
          <grid-item
            v-for="item in layout"
            :key="item.i"
            :static="item.static"
            :x="item.x"
            :y="item.y"
            :w="item.w"
            :h="item.h"
            :i="item.i"
            @resize="resize"
            @move="move"
            @resized="resized"
            @moved="moved"
            ref="layout"
          >
            <!--<custom-drag-element :text="item.i"></custom-drag-element>-->
            <test-element :text="item.i"></test-element>
            <!--<button @click="clicked">CLICK ME!</button>-->
          </grid-item>
        </grid-layout>
      </div>
      <hr />
    </div>
  </div>
</template>

<script>
import GridItem from "./components/GridItem.vue";
import $ from "jquery"
import GridLayout from "./components/GridLayout.vue";
// import ResponsiveGridLayout from './components/ResponsiveGridLayout.vue';
import TestElement from "./components/TestElement.vue";
import CustomDragElement from "./components/CustomDragElement.vue";
import { getDocumentDir, setDocumentDir } from "./helpers/DOM";
import { setInterval } from 'timers';
//var eventBus = require('./eventBus');

let testLayout = [
  {
    x: 0,
    y: 0,
    w: 2,
    h: 2,
    i: "0",
    resizable: true,
    draggable: true,
    static: false
  },
  {
    x: 2,
    y: 0,
    w: 2,
    h: 4,
    i: "1",
    resizable: null,
    draggable: null,
    static: true
  },
  {
    x: 4,
    y: 0,
    w: 2,
    h: 5,
    i: "2",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 6,
    y: 0,
    w: 2,
    h: 3,
    i: "3",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 8,
    y: 0,
    w: 2,
    h: 3,
    i: "4",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 10,
    y: 0,
    w: 2,
    h: 3,
    i: "5",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 0,
    y: 5,
    w: 2,
    h: 5,
    i: "6",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 2,
    y: 5,
    w: 2,
    h: 5,
    i: "7",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 4,
    y: 5,
    w: 2,
    h: 5,
    i: "8",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 6,
    y: 3,
    w: 2,
    h: 4,
    i: "9",
    resizable: false,
    draggable: false,
    static: true
  },
  {
    x: 8,
    y: 4,
    w: 2,
    h: 4,
    i: "10",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 10,
    y: 4,
    w: 2,
    h: 4,
    i: "11",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 0,
    y: 10,
    w: 2,
    h: 5,
    i: "12",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 2,
    y: 10,
    w: 2,
    h: 5,
    i: "13",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 4,
    y: 8,
    w: 2,
    h: 4,
    i: "14",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 6,
    y: 8,
    w: 2,
    h: 4,
    i: "15",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 8,
    y: 10,
    w: 2,
    h: 5,
    i: "16",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 10,
    y: 4,
    w: 2,
    h: 2,
    i: "17",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 0,
    y: 9,
    w: 2,
    h: 3,
    i: "18",
    resizable: false,
    draggable: false,
    static: false
  },
  {
    x: 2,
    y: 6,
    w: 2,
    h: 2,
    i: "19",
    resizable: false,
    draggable: false,
    static: false
  }
];

export default {
  name: "app",
  components: {
    // ResponsiveGridLayout,
    GridLayout,
    GridItem,
    TestElement,
    CustomDragElement
  },
  data() {
    return {
      layout: JSON.parse(JSON.stringify(testLayout)),
      layout2: JSON.parse(JSON.stringify(testLayout)),
      draggable: true,
      resizable: true,
      mirrored: false,
      responsive: true,
      preventCollision: false,
      colNum: 12,
      originW: 4,
      originH: 6,
      rowHeight: 30,
      maxRows: 10000,
      margin: [10, 10],
      index: 0
    };
  },
  mounted: function() {
    this.index = this.layout.length;
    var last= this.layout[this.index-1]
    this.moveEl = $("<div class='move-el text'></div>");
    $(".vue-grid-layout").append(this.moveEl);
    this.moveEl.width(this.calcColWidth()*this.originW);
    this.moveEl.height(this.originH*this.rowHeight);
    
  },
  computed: {
    colItemCount() {
      return this.colNum / this.originW;
    }
  },
  methods: {
    drop: function(event) {
     this.tempItem=null;
     this.moveEl.hide();
     console.log($(this.$refs.layout[this.layout.length-1].$el).attr("class"))
     $(this.$refs.layout[this.layout.length-1].$el).removeClass("move-shaw")
    },
    dragover: function(ev) {
      var pageX = ev.pageX;
      var pageY = ev.pageY;
      var containerX = $(".vue-grid-layout").offset().left;
      var containerY = $(".vue-grid-layout").offset().top;
      var x = pageX - containerX-this.originW*this.calcColWidth()/2;
      var y = pageY - containerY-this.originH*this.rowHeight/2;
    
      ev.preventDefault();
      
      this.addItem(x, y);
    },
    dragleave:function(){
         // this.layout.splice(this.layout.length-1,1)
    },
    dragstart: function() {},
    dragend:function(){
      //if()
    },
    calcXY(top, left) {
      var colWidth = this.calcColWidth();

      var x = Math.round((left - this.margin[0]) / (colWidth + this.margin[0]));
      var y = Math.round(
        (top - this.margin[1]) /
          (this.originH * this.rowHeight + this.margin[1])
      );

      // Capping
      x = Math.max(Math.min(x, this.colNum - this.originW), 0);
      y = Math.max(Math.min(y, this.maxRows - this.originH), 0);

      return { x, y };
    },
    // Helper for generating column width
    calcColWidth() {
      var colWidth =
        ($(".vue-grid-layout").width() - (this.colNum + 1)*this.margin[0]) /
        this.colNum;
      // console.log("### COLS=" + this.cols + " COL WIDTH=" + colWidth + " MARGIN " + this.margin[0]);
      return colWidth;
    },
    clicked: function() {
      window.alert("CLICK!");
    },
    increaseWidth: function() {
      let width = document.getElementById("content").offsetWidth;
      width += 20;
      document.getElementById("content").style.width = width + "px";
    },
    decreaseWidth: function() {
      let width = document.getElementById("content").offsetWidth;
      width -= 20;
      document.getElementById("content").style.width = width + "px";
    },
    removeItem: function(item) {
      //console.log("### REMOVE " + item.i);
      this.layout.splice(this.layout.indexOf(item), 1);
    },
    addItem: function(left, top) {
      var that = this;
       
      var offset = this.calcXY(top, left);
      // let self = this;
      //console.log("### LENGTH: " + this.layout.length);
    var x = offset.x;
   var  y = offset.y;
    this.moveEl.css("left",left+"px")
    this.moveEl.css("top",top+"px");
      if (!this.tempItem || !this.tempItem.isTemp) {
        this.tempItem = {
          x: x,
          y: y,
          w: this.originW,
          h: this.originH,
          i: this.layout.length + 100+"",
          isTemp: true
        };
        this.layout.push(this.tempItem);
        this.moveEl.show();

      }
      else
      {
        this.moveFlag = true
        if(this.moveFlag){
            this.moveFlag = false;
        setTimeout(()=>{
        this.layout[this.layout.length-1].x=x;
        this.layout[this.layout.length-1].y=y;
        $(this.$refs.layout[this.layout.length-1].$el).addClass("move-shaw")
        this.$refs.gridLayout.dragEvent("dragmove",x,y,this.originW,this.originH);
        this.moveFlag = true;
        },200)
        }


       
      }
      
      
    },
    move: function(i, newX, newY) {
      console.log("MOVE i=" + i + ", X=" + newX + ", Y=" + newY);
    },
    resize: function(i, newH, newW, newHPx, newWPx) {
      console.log(
        "RESIZE i=" +
          i +
          ", H=" +
          newH +
          ", W=" +
          newW +
          ", H(px)=" +
          newHPx +
          ", W(px)=" +
          newWPx
      );
    },
    moved: function(i, newX, newY) {
      console.log("### MOVED i=" + i + ", X=" + newX + ", Y=" + newY);
    },
    resized: function(i, newH, newW, newHPx, newWPx) {
      console.log(
        "### RESIZED i=" +
          i +
          ", H=" +
          newH +
          ", W=" +
          newW +
          ", H(px)=" +
          newHPx +
          ", W(px)=" +
          newWPx
      );
    },
    /**
     * Add change direction button
     */
    changeDirection() {
      let documentDirection = getDocumentDir();
      let toggle = "";
      if (documentDirection === "rtl") {
        toggle = "ltr";
      } else {
        toggle = "rtl";
      }
      setDocumentDir(toggle);
      //eventBus.$emit('directionchange');
    },

    layoutCreatedEvent: function(newLayout) {
      console.log("Created layout: ", newLayout);
    },
    layoutBeforeMountEvent: function(newLayout) {
      console.log("beforeMount layout: ", newLayout);
    },
    layoutMountedEvent: function(newLayout) {
      console.log("Mounted layout: ", newLayout);
    },
    layoutReadyEvent: function(newLayout) {
      console.log("Ready layout: ", newLayout);
    },
    layoutUpdatedEvent: function(newLayout) {
      console.log("Updated layout: ", newLayout);
    }
  }
};
</script>


<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*text-align: center;*/
  color: #2c3e50;
  /*margin-top: 60px;*/
}
.move-el{
  position:absolute;
  display:none;
  font-size: 24px;
    text-align: center;
    background: #ccc;
    border: 1px solid black;
}
.move-shaw{
background:rgba(255,255,0,0.5)!important;
border:none !important;
}

</style>
