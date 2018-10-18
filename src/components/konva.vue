<template>
  <div class="div"  >
    <h2><span v-if="transformation == 'on'">режим трансформации </span><span v-else>Режим мапинга</span></h2>
    <button @click="setClosed">Замкнуть фигуру</button>
    <button @click="setClear">Очистить</button>
    <button @click="setTransform">Трансформировать</button>

    <label for="scale">Маштаб</label>
    <input type="range" id="scale" min="1" max="100" value="50" @input="scaleAll" >

    <v-stage @click="clickStage" ref="stage" :config="configKonva" @wheel="scaleAll">
      <v-layer ref="layerImg">
        <v-image :config="configImg"></v-image>
      </v-layer>
      <v-layer ref="layer" >
        <v-group ref="groupPoly">
          <v-line ref="line"  :config="configPoly"></v-line>
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
            configPoly: {
                points: [],
                fill: 'rgba(0,0,0,.5)',
                stroke: 'black',
                strokeWidth: 2,
                closed : false,
            },
            cirles: [],
            testImg: new Image(100, 100),
            transformation: 'off',
            circleRadius: 3,
            contextMenu: true,
            deltaAllScale: 1
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
        scaleAll(c,e){
            const layer = this.$refs.layer.getStage();
            const layerImg = this.$refs.layerImg.getStage();
            let delta = 0;
            if (c instanceof Event) {
                this.deltaAllScale =  c.target.value * 25 * 0.001;
            }
            else {
                this.deltaAllScale = layer.scaleX() + -e.evt.deltaY * 0.001;
            }

            this.unTransformCircle()
            layer.scaleX(this.deltaAllScale);
            layer.scaleY(this.deltaAllScale);
            layer.draw();

            layerImg.scaleX(this.deltaAllScale);
            layerImg.scaleY(this.deltaAllScale);
            layerImg.draw();

            console.log(layer.scale())
        },
        calculatedXY(xy){
            const   groupPoly = this.$refs.groupPoly.getStage(),

              x = (xy[0] - (groupPoly.x() * this.deltaAllScale)) / (groupPoly.scaleX() * this.deltaAllScale),///
              y = (xy[1] - (groupPoly.y() * this.deltaAllScale)) / (groupPoly.scaleY() * this.deltaAllScale);///

            return [x,y]

        },
        drawCircle(x,y,num){
            const   layer = this.$refs.layer.getStage(),
                groupPoly = this.$refs.groupPoly.getStage(),
                circle = new window.Konva.Circle({
                    x: x,
                    y: y,
                    radius: this.circleRadius,
                    fill: 'red',
                    stroke: 'black',
                    strokeWidth: 1,
                    draggable: true,
                    circleNum: num
                });

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
                const line = this.$refs.line.getStage()
                let points = line.points();
                points.splice(circleNum * 2, 2);
                line.points(points);
                line.draw();

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
            const line = this.$refs.line.getStage()
            let points = line.points();
            points[e.target.attrs.circleNum*2  ] = e.target.x()
            points[e.target.attrs.circleNum*2 +1] = e.target.y()
            line.points(points);
            line.draw();
        },
        //клик по все йрабочей области
        clickStage(c,e) {
            if (this.transformation === 'on' || e.evt.button === 2) {
                return false
            }

            const line = this.$refs.line.getStage(),
              x = this.calculatedXY([e.evt.layerX,e.evt.layerX])[0],
              y = this.calculatedXY([e.evt.layerX,e.evt.layerY])[1];
            line.points(line.points().concat([x, y]));
            this.drawCircle(x, y, line.points().length / 2 -1)

            this.$refs.line.getStage().draw()
        },
        //замыкание фигуры
        setClosed() {
            this.configPoly.closed = !this.configPoly.closed
            this.$refs.line.getStage().setClosed(this.configPoly.closed)
            this.$refs.line.getStage().draw()
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
            this.configPoly.points = [];
            let line = this.$refs.line.getStage(),
                layer = this.$refs.layer.getStage();
            let groupPoly = this.$refs.groupPoly.getStage()
            this.destroyChildrenCircle(groupPoly);
            this.configPoly.closed = true;
            this.setClosed()
            line.points([]);
            layer.draw();
        },
        unTransformCircle() {
            const groupPoly = this.$refs.groupPoly.getStage()
            groupPoly.getChildren((circle)=>{
                if (circle.getClassName() === 'Circle') {
                    circle.scaleX(1 / groupPoly.scaleX())
                    circle.scaleY(1 / groupPoly.scaleY())
                }
            })
        },
        setTransform() {
            const stage = this.$refs.stage.getStage()
            stage.find('Transformer').destroy()

            const groupPoly = this.$refs.groupPoly.getStage()
            if (this.transformation == 'off') {
                this.transformation = 'on'
                var tr = new window.Konva.Transformer()
                let layer = this.$refs.layer.getStage(tr)

                tr.attachTo(groupPoly)
                layer.add(tr)
                groupPoly.draggable(true)
                layer.draw()
            }
            else {
                this.transformation = 'off'
                this.unTransformCircle()
                groupPoly.draggable(false)
                stage.draw()
            }
        },
    },
    mounted() {
        this.testImg.onload = () => this.$refs.layerImg.getStage().draw()
        this.testImg.src = "http://crm.m2metr.com/uploads/img/14/53e266e6422e7a773ed8c43f0df9a86c.jpeg"
    }
}
</script>

<style scoped>

</style>
