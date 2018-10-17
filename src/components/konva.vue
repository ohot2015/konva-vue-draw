<template>
  <div class="div">
    <h2><span v-if="transformation == 'on'">режим трансформации </span><span v-else>Режим мапинга</span></h2>
    <button @click="setClosed">Замкнуть фигуру</button>
    <button @click="setClear">Очистить</button>
    <button @click="setTransform">Трансформировать</button>
    <v-stage @click="clickStage" ref="stage" :config="configKonva">
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
                fill: '#00D2FF',
                stroke: 'black',
                strokeWidth: 2,
                closed : false,
            },
            cirles: [],
            testImg: new Image(100, 100),
            transformation: 'off',
            circleRadius: 7,
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
        calculatedXY(xy){
            const   groupPoly = this.$refs.groupPoly.getStage(),
              x = (xy[0]  - groupPoly.x()) / groupPoly.scaleX(),
              y = (xy[1]  - groupPoly.y()) / groupPoly.scaleY();
            return [x,y]
        },
        drawCircle(x,y){
            const   layer = this.$refs.layer.getStage(),
                groupPoly = this.$refs.groupPoly.getStage(),
                circle = new Konva.Circle({
                    x: x,
                    y: y,
                    radius: this.circleRadius,
                    fill: 'red',
                    stroke: 'black',
                    strokeWidth: 1,
                    draggable: true,
                });

            circle.on('mousedown',  (e)=> {
                this.transformation = 'on';
            })
            .on('mouseup',  (e)=> {
                this.transformation = 'off';
            })
            // add cursor styling
            .on('mouseover', function() {
                document.body.style.cursor = 'pointer';
            })
            .on('mouseout', function() {
                document.body.style.cursor = 'default';
            })
            .on('dragstart', ()=>{
                const line = this.$refs.line.getStage()
                let points = line.points();
                let pointsXY = [];
                let tmp = [];
                points.forEach((point,i) => {
                    tmp.push(point);
                    if (i % 2 !== 0) {
                        pointsXY.push(tmp.join(','))
                        tmp = [];
                    }
                })

                this.circles = pointsXY.indexOf(`${x},${y}`)
            })
            .on('dragmove',(e)=>{
                const line = this.$refs.line.getStage()
                let points = line.points();
                points[this.circles * 2 ] = e.target.x()
                points[this.circles * 2 + 1] = e.target.y()
                line.points(points);
                line.draw();
            });

            groupPoly.add(circle);
            this.unTransformCircle()
            layer.add(groupPoly);
            layer.draw();
        },
        clickStage(c,e) {
            if (this.transformation === 'on') {
                return false
            }
            const line = this.$refs.line.getStage(),
              x = this.calculatedXY([e.evt.layerX,e.evt.layerX])[0],
              y = this.calculatedXY([e.evt.layerX,e.evt.layerY])[1];
            this.drawCircle(x, y)
            line.points(line.points().concat([x, y]));

            this.$refs.line.getStage().draw()
        },
        setClosed() {
            this.configPoly.closed = !this.configPoly.closed
            this.$refs.line.getStage().setClosed(this.configPoly.closed)
            this.$refs.line.getStage().draw()
        },
        destroyChildrenCircle(self){
            var children = Konva.Collection.toCollection(self.children);
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
            self.children = new Konva.Collection();
            return self
        },
        setClear() {
            this.configPoly.points = [];
            let line = this.$refs.line.getStage(),
                layer = this.$refs.layer.getStage();
            let groupPoly = this.$refs.groupPoly.getStage()
            groupPoly = this.destroyChildrenCircle(groupPoly);

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
