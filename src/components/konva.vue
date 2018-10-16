<template>
  <div class="div">
    <h2><span v-if="transformation == 'on'">режим трансформации </span><span v-else>Режим мапинга</span></h2>
    <button @click="setClosed">Замкнуть фигуру</button>
    <button @click="setClear">Очистить</button>
    <button @click="setTransform">Трансформировать</button>
    <v-stage @mousedown="clickStage" ref="stage" :config="configKonva">
      <v-layer ref="layer">
        <v-image :config="configImg"></v-image>
      </v-layer>
      <v-layer ref="layer">
        <v-line ref="line"  :config="configPoly"></v-line>
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
                points: [23, 20, 23, 160, 70, 93, 150, 109, 290, 139, 270, 93, 117, 15],
                fill: '#00D2FF',
                stroke: 'black',
                strokeWidth: 2,
                closed : false,
            },
            cirles:[],
            testImg: new Image(100, 100),
            transformation: 'off',
        }
    },
    computed: {
        configImg: function() {
            return {
                x: 20,
                y: 20,
                image: this.testImg,
                width: 200,//1280,
                height: 200, //810,
            }
        }
    },

    methods: {
        drawCircle(x,y){

        },

        clickStage(c,e) {
            if (this.transformation == 'on') {
                return false
            }
            let line = this.$refs.line.getStage();
            line.points(line.points().concat([e.evt.layerX, e.evt.layerY]));
            this.drawCircle(e.evt.layerX, e.evt.layerY)
            this.$refs.line.getStage().draw()
        },

        setClosed() {
            this.configPoly.closed = !this.configPoly.closed
            this.$refs.line.getStage().setClosed(this.configPoly.closed)
            this.$refs.line.getStage().draw()
        },
        setClear() {
            this.configPoly.points = [];
            let line = this.$refs.line.getStage(),
                layer = this.$refs.layer.getStage();
            line.points([]);
            layer.clear()
        },
        setTransform() {
            let stage = this.$refs.stage.getStage()
            stage.find('Transformer').destroy()
            let line = this.$refs.line.getStage();

            if (this.transformation == 'off') {
                this.transformation = 'on'
                // create new transformer
                var tr = new window.Konva.Transformer()
                let layer = this.$refs.layer.getStage(tr)
                tr.attachTo(this.$refs.line.getStage())
                layer.add(tr);
                line.draggable(true);
                layer.draw()
            }
            else {
                this.transformation = 'off'
                line.draggable(false);
                stage.draw();
            }
        },
    },
    mounted(){
        this.testImg.src = "http://crm.m2metr.com/uploads/img/14/53e266e6422e7a773ed8c43f0df9a86c.jpeg"
        console.log(123)
    }
}
</script>

<style scoped>

</style>
