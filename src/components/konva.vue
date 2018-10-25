<template >
  <div class="div">
    <h2><span v-if="transformation == 'on'">режим трансформации </span><span v-else>Режим мапинга</span></h2>
    <button @click="setClosed">Замкнуть фигуру</button>&nbsp;
    <button @click="setClear">Очистить</button>&nbsp;
    <!--<button @click="setTransform">Трансформировать</button>&nbsp;-->

    <label for="scale">Маштаб</label>&nbsp;
    <input type="range" id="scale" min="1" max="100" value="50" @input="scaleAll" > &nbsp;
    <label for="sizePoint">размер точки</label>&nbsp;
    <input type="number" id="sizePoint" @change="changePointSize" :value="circleRadius">
    <span>Для перетаскивания зажмите ctrl </span>
    <textarea cols="30" rows="10">{{ polygons }}</textarea>
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
        <v-group ref="groupPoly" v-for="poly in polygons">
          <v-line ref="line"  :config="poly.configPoly"></v-line>
          <v-circle v-for="c in circle"></v-circle>
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
            polygons:[
                {
                    configPoly: {
                        points: [],
                        fill: 'rgba(0,0,0,.5)',
                        stroke: 'black',
                        strokeWidth: 2,
                        closed : false,
                    },
                    circle:[]
                },
            ],

            testImg: new Image(100, 100),
            transformation: 'off',
            circleRadius: 3,
            contextMenu: true,
            deltaAllScale: 1,
            line:{},
            groupPoly:{},
            layer:{}

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
            this.groupPoly.getChildren((node)=>{
                if (node.getClassName() === 'Circle') {
                    node.radius(+e.target.value);
                    this.circleRadius = +e.target.value;
                }
            })
            this.groupPoly.draw();
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
                this.unTransformCircle()
                layer.scaleX(this.deltaAllScale);
                layer.scaleY(this.deltaAllScale);
                layer.draw();
            })
            layerImg.scaleX(this.deltaAllScale);
            layerImg.scaleY(this.deltaAllScale);
            layerImg.draw();

        },
        calculatedXY(xy){
            let
              x = (xy[0] - this.layer.x() - (this.groupPoly.x() * this.deltaAllScale)) / (this.groupPoly.scaleX() * this.deltaAllScale),///
              y = (xy[1] - this.layer.y() - (this.groupPoly.y() * this.deltaAllScale)) / (this.groupPoly.scaleY() * this.deltaAllScale);///
            return [x,y]
        },
        drawCircle(x,y,num){
                let layer = this.layer,
                groupPoly = this.groupPoly,
                circle = {
                    x: x,
                    y: y,
                    radius: this.circleRadius,
                    fill: 'red',
                    stroke: 'black',
                    strokeWidth: 1,
                    draggable: true,
                    circleNum: num
                };

            circle.on('mousedown',  (e)=> {
                this.removePointPolygonAndCircle(e);
            })
            .on('mouseup', (e)=> {
                this.transformation = 'off';
                if (e.evt.button === 2) {
                    this.contextMenu = true
                }
            })
            .on('mouseover', function() {
                document.body.style.cursor = 'pointer';
            })
            .on('mouseout', function() {
                document.body.style.cursor = 'default';
            })
            .on('dragmove',(e)=>{
                this.circleDragMove(e);
            })

            window.oncontextmenu = () => this.contextMenu;

            groupPoly.add(circle);
            this.unTransformCircle()
            layer.add(groupPoly);
            layer.draw();
        },
        removePointPolygonAndCircle(e){
            this.transformation = 'on';
            //клик правой кнопкой
            if (e.evt.button === 2) {
                this.contextMenu = false;
                let circleNum = e.target.attrs.circleNum;

//                    удаление из полигона
                let points = this.line.points();
                points.splice(circleNum * 2, 2);
                this.line.points(points);
                this.line.draw();

                e.target.parent.getChildren((node)=>{
                    if (node.getClassName() == 'Circle') {
                        if (node.attrs.circleNum > circleNum) {
                            node.attrs.circleNum--;
                        }
                    }
                });
                this.transformation = 'off';
                e.target.destroy();
            }
        },
        circleDragMove(e){
            let points = this.line.points();
            points[e.target.attrs.circleNum*2  ] = e.target.x()
            points[e.target.attrs.circleNum*2 +1] = e.target.y()
            this.line.points(points);
            this.line.draw();
        },
        //клик по всей рабочей области
        clickStage(c,e) {
            if (this.transformation === 'on' || e.evt.button === 2) {
                return false
            }

            const
              x = this.calculatedXY([e.evt.layerX,e.evt.layerX])[0],
              y = this.calculatedXY([e.evt.layerX,e.evt.layerY])[1];

            this.line.points(this.line.points().concat([x, y]));
            this.drawCircle(x, y, this.line.points().length / 2 -1)
            this.line.draw()
        },
        //замыкание фигуры
        setClosed() {
//            this.configPoly.closed = !this.configPoly.closed
//            this.$refs.line.getStage().setClosed(this.configPoly.closed)
//            this.$refs.line.getStage().draw()

            let poly = this.polygons[this.polygons.length - 1]
            this.polygons.push(JSON.parse(JSON.stringify(poly)))

            this.line = this.$refs.line[this.$refs.line.length - 1].getStage()
            this.groupPoly = this.$refs.groupPoly[this.$refs.groupPoly.length - 1].getStage()
            //this.layer = this.$refs.layer[this.$refs.layer.length - 1].getStage()
            this.layer = this.$refs.layer.getStage()


            setTimeout(()=>{ this.line = this.$refs.line[this.polygons.length - 1].getStage() }, 500)
            this.line.closed(true).draw();
//            this.groupPoly = this.$refs.groupPoly.getStage()
//            this.layer = this.$refs.layer.getStage()

        },
        destroyChildrenCircle(self){
            var children = window.Konva.Collection.toCollection(self.children);
            var child;
            for (var i = 0; i < children.length; i++) {
                child = children[i];
                if (child.getClassName() === 'Circle'){
                    delete child.parent;
                    child.index = 0;
                    child.destroy();
                }
            }
            children = null;
            self.children = new window.Konva.Collection();
            return self
        },
        setClear() {

            this.polygons.pop()
            this.destroyChildrenCircle(this.groupPoly);
            this.stage.draw();


        },
        unTransformCircle() {
            this.groupPoly.getChildren((circle)=>{
                if (circle.getClassName() === 'Circle') {
                    circle.scaleX(1 / this.groupPoly.scaleX())
                    circle.scaleY(1 / this.groupPoly.scaleY())
                }
            })
            this.groupPoly.draw();
        },
        setTransform() {
            const stage = this.$refs.stage.getStage()
            stage.find('Transformer').destroy()

            if (this.transformation == 'off') {
                this.transformation = 'on'
                let tr = new window.Konva.Transformer()

                // добавляем трансформаию толлько одноверменную по всем сторонам,
                // потому что line Stroc невозможно пересчитать
                tr.keepRatio(true)
                  .enabledAnchors(['top-left', 'top-right', 'bottom-left', 'bottom-right'])

                let layer = this.$refs.layer[this.$refs.layer.length - 1].getStage(tr)
                tr.attachTo(this.groupPoly)
                this.groupPoly.draggable(true)
                layer.add(tr)
                  .draw()
            }
            else {
                this.transformation = 'off'
                this.unTransformCircle()
                this.groupPoly.draggable(false)
                    .getChildren((node)=>{
                        if (node.getClassName() === 'Line') {
                            node.strokeWidth(this.line.strokeWidth() / this.groupPoly.scaleX() );
                            node.draw();
                        }
                    })
                stage.draw()
            }
        },
    },
    mounted() {
        this.testImg.onload = () => this.$refs.layerImg.getStage().draw()
        this.testImg.src = "http://crm.m2metr.com/uploads/img/14/53e266e6422e7a773ed8c43f0df9a86c.jpeg"

        this.line = this.$refs.line[this.$refs.line.length - 1].getStage()
        this.groupPoly = this.$refs.groupPoly[this.$refs.groupPoly.length - 1].getStage()
        //this.layer = this.$refs.layer[this.$refs.layer.length - 1].getStage()
        this.layer = this.$refs.layer.getStage()

//        this.line = this.$refs.line[this.$refs.line.length - 1].getStage()
//        this.groupPoly = this.$refs.groupPoly.getStage()

        this.layer = this.$refs.layer.getStage()

    }
}
</script>

<style scoped>

</style>
