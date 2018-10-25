<template >
  <div class="mappigKonva">
    <button @click="setClosed">Замкнуть фигуру</button>&nbsp;
    <button @click="setClear">Очистить</button>&nbsp;
    <!--<button @click="setTransform">Трансформировать</button>&nbsp;-->
    <label for="scale">Маштаб</label>&nbsp;
    <input type="range" id="scale" min="1" max="100" value="50" @input="scaleAll" > &nbsp;
    <!--<label for="sizePoint">размер точки</label>&nbsp;-->
    <!--<input type="number" id="sizePoint" @change="changePointSize" :value="defaultConfigCircle.radius">-->
    <span>Для перетаскивания зажмите ctrl </span>
      <input type="hidden" value="">
    <v-stage
            ref="stage"
            :config="configKonva"
            @click="clickStage"
            @mousemove="moveImage"
            @wheel="scaleAll"
            @mouseup="stopDragImage"
    >
      <v-layer ref="layerImg" >
        <v-image :config="configImg"></v-image>
      </v-layer>
      <v-layer ref="layer" >
        <v-group ref="groupPoly" v-for="poly in polygons" :config="defaultGroup">
          <v-line ref="line" :config="poly.poly"></v-line>
          <v-circle
                  v-for="c in poly.circle"
                  :config="c"
                  @mousedown="mousedownCircle"
                  @mouseover="mouseoverCircle"
                  @mouseout="mouseoutCircle"
                  @dragmove="dragmoveCircle"
          >
          </v-circle>
        </v-group>
      </v-layer>
    </v-stage>
  </div>
</template>

<script>

export default {
    data() {
        return {
            configKonva: {
                width: 1817,
                height: 648
            },
            polygons:[],
            defaultConfigPoly:{
                points: [],
                fill: 'rgba(0,0,0,.5)',
                stroke: 'black',
                strokeWidth: 1,
                closed : true,
                saved:false
            },
            defaultGroup:{
                x:0,
                y:0,
                scaleX:1,
                scaleY:1,
                poly:'',
                circle:[],
            },
            defaultConfigCircle:{
                x:0,
                y:0,
                radius: 2,
                fill: 'red',
                stroke: 'black',
                strokeWidth: 1,
                draggable: true,
                num: 0
            },
            testImg: new Image(100, 100),
            deltaAllScale: 1,
            eventClick:{},
            currentGroup:{},
        }
    },
    computed: {
        configImg: function() {
            return {
                x: 0,
                y: 0,
                image: this.testImg,
                width: 1280,
                height: 810,
            }
        }
    },
    methods: {
        changePointSize(e) {
            this.defaultConfigCircle.radius = e.target.value;
            this.currentGroup.circle.forEach((obj)=>{obj.radius = e.target.value})
        },
        moveImage(c,event) {
            const e = event.evt;
            if (e.ctrlKey) {
                const layerImg = this.$refs.layerImg.getStage();
                layerImg.draggable(true)
                layerImg.on('dragmove', (node)=>{
                    this.layer.x(node.target.x());
                    this.layer.y(node.target.y());
                    this.layer.draw()
                })
            }
        },
        stopDragImage(){
            const layerImg = this.$refs.layerImg.getStage();
            layerImg.draggable(false)
        },
        scaleAll(c,e){

            const layerImg = this.$refs.layerImg.getStage();
            if (c instanceof Event) {
                this.deltaAllScale =  c.target.value * 25 * 0.001;
            }
            else {
                this.deltaAllScale = this.layer.scaleX() + -e.evt.deltaY * 0.001;
            }
            this.$refs.stage.getStage().getChildren((layer)=>{
            //    this.unTransformCircle()
                layer.scaleX(this.deltaAllScale);
                layer.scaleY(this.deltaAllScale);
                layer.draw();
            })
            layerImg.scaleX(this.deltaAllScale);
            layerImg.scaleY(this.deltaAllScale);
            layerImg.draw();

        },
        calculatedXY(xy,group){
            let
              x = (xy[0] - this.layer.x() - (group.x * this.deltaAllScale)) / (group.scaleX * this.deltaAllScale),///
              y = (xy[1] - this.layer.y() - (group.y * this.deltaAllScale)) / (group.scaleY * this.deltaAllScale);///
            return {x:x,y:y}
        },
        dragmoveCircle(c,e){
            this.currentGroup.poly.points[e.target.attrs.num * 2] = e.target.x();
            this.currentGroup.poly.points[e.target.attrs.num * 2 + 1] = e.target.y();
        },
        mouseoutCircle(){
            document.body.style.cursor = 'default';
        },
        mouseoverCircle(){
            document.body.style.cursor = 'pointer';
        },
        mousedownCircle(c,e){
            if (e.evt.button === 2) {
                window.oncontextmenu = () => false;
                this.currentGroup.poly.points.splice(c.config.num * 2, 2);
                this.currentGroup.circle.splice(c.config.num,1);
                this.currentGroup.circle.forEach((circle) => {
                    if (circle.num > c.config.num) {
                        circle.num --;                        }
                });
            }
        },
        circleDragMove(e){

        },
        clone(obj){
            return JSON.parse(JSON.stringify(obj));
        },
        editGroup(e) {
            //создаем новую группу если полигон пустой
            if (!this.polygons.length) {
                const index = this.polygons.push(this.clone(this.defaultGroup));
                this.polygons[index -1].poly = this.clone(this.defaultConfigPoly);
            }
            else if (this.polygons[this.polygons.length -1].poly.saved) {
                const index = this.polygons.push(this.clone(this.defaultGroup));
                this.polygons[index -1].poly = this.clone(this.defaultConfigPoly);
            }
            this.currentGroup = this.polygons[this.polygons.length -1];
            //создаем новый круг
            this.currentGroup.circle.push(this.clone(this.defaultConfigCircle));

            //последний круг
            const circle = this.currentGroup.circle[this.currentGroup.circle.length - 1],
                  XY = this.calculatedXY([e.evt.layerX, e.evt.layerY], this.currentGroup);

            //задаем координаты круга
            circle.x = XY.x;
            circle.y = XY.y;
            //задаем координаты полигона линии
            this.currentGroup.poly.points.push(XY.x);
            this.currentGroup.poly.points.push(XY.y);

            // сохраняем позицию точек в полигоне
            circle.num = this.currentGroup.poly.points.length / 2 -1
        },
        //клик по всей рабочей области
        clickStage(c,e) {
            if (e.evt.button === 2) {
                window.oncontextmenu = () => true;
                return false
            }
            this.editGroup(e);
        },
        //завершение отрисовки фигуры
        setClosed() {
            if (this.currentGroup.poly) {
                this.currentGroup.poly.saved = true;
            }
        },
        setClear() {
            this.polygons.pop()
        },
    },
    mounted() {
        this.testImg.onload = () => this.$refs.layerImg.getStage().draw()
        this.testImg.src = "http://crm.m2metr.com/uploads/img/14/53e266e6422e7a773ed8c43f0df9a86c.jpeg"
        this.layer = this.$refs.layer.getStage()
    }
}
</script>

<style scoped>
    textarea {
        position: absolute;
        top: 0;
        right:0;
    }
</style>
