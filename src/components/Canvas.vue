<template>
  <section class="container">
    <aside class="images">
      <ul class="images__list">
        <li class="images__item">
          <img
            alt="uma"
            src="https://konvajs.org/assets/yoda.jpg"
            crossorigin="anonymous"
            @click="addItemToBoard($event)"
            id="i1"
          />
          <label>image 1</label>
        </li>
        <li class="images__item">
          <img
            alt="duas"
            src="https://konvajs.org/assets/darth-vader.jpg"
            crossorigin="anonymous"
            @click="addItemToBoard($event)"
            id="i2"
          />
          <label>image 2</label>
        </li>
      </ul>
      <button @click="deleteImage($event)">Delete</button>
    </aside>
    <div
      id="foamboard"
      ref="foamboard"
      @click="handlesBoardClick($event)"
    >
    </div>
  </section>
</template>

<script>
import Konva from 'konva'

export default {
  name: 'Canvas',

  data() {
    return {
      stage: null,
      context: null,
      layer: [],
      stageSize: {
        stage: 'stage',
        width: 1000,
        height: 700
      },
      isDrawing: false,
      imageUrl: '',
      dragok: false,
      startX: 0,
      startY: 0,
      points: [],
      images: [],
      transformer: [],
      selectedShapeName: "",
      group: []
    };
  },

  mounted() {
    this.stage = new Konva.Stage({ // this line till the stage.add() line renders the draggable square
      container: 'foamboard',
      width: 1000,
      height: 700
    })

    this.layer = new Konva.Layer()

    this.group = new Konva.Group({
      x: 120,
      y: 40,
      draggable: true
    })
  },

  methods: {
    addItemToBoard(evt) {
      const vm = this
  
      const transformer = new Konva.Transformer()
      this.transformer = transformer
      this.layer.add(transformer)

      const group = new Konva.Group({
        x: 120,
        y: 40,
        draggable: true
      })

      let images = this.images

      const imageObj = new Image();
      imageObj.onload = function () {
        const imageWidth = evt.target.width
        const imageHeight = evt.target.height
        const maxSize = 100
        const ratio = (evt.target.width > evt.target.height ? (evt.target.width / maxSize) : (evt.target.height / maxSize))

        const image = new Konva.Image({
          x: 50,
          y: 50,
          image: imageObj,
          width: imageWidth/ratio,
          height: imageHeight/ratio,
          name: `image-${evt.target.id}`
        })

        const labelText = new Konva.Text({
          x: 50,
          y: 140,
          text: evt.target.nextSibling.outerText,
          fontSize: 14,
          fontFamily: 'Calibri',
          fill: 'black',
          cornerRadius: 10,
          width: 100,
          padding: 5,
          align: 'center'
        })

        const textLabelBg = new Konva.Rect({
          x: 50,
          y: 140,
          fill: '#ddd',
          width: labelText.width(),
          height: labelText.height(),
          cornerRadius: 10,
          draggable: true,
          name: 'label text'
        })

        group.add(image, textLabelBg, labelText)

        vm.layer.add(group)
        vm.layer.batchDraw()

        image.on('mouseover', function () {
          document.body.style.cursor = 'pointer'
        })

        image.on('mouseout', function () {
          document.body.style.cursor = 'default'
        })

        vm.stage.on('dragstart', function (e) {
          const x = e.target.attrs.x
          const y = e.target.attrs.y
          vm.startX = x
          vm.startY = y
        })

        group.on('dragend', function () {
          // const mousePos = vm.stage.getPointerPosition();
          const x = group.attrs.x + 50
          const y = group.attrs.y + 50

          console.log(group.attrs.x)
          console.log(group.attrs.y)

          if (x > 900 || x < 0 || y > 590 || y < 0) {
            group.absolutePosition({
              x: vm.startX,
              y: vm.startY
            })
          }
        })

        transformer.nodes([image, textLabelBg, labelText])
        images[evt.target.id] = image
      }

      imageObj.src = evt.target.src
        
      this.selectedShapeName = `image-${evt.target.id}`
      this.stage.add(this.layer)
    },

    handlesBoardClick() {
      const stage = this.stage
      const transformer = this.transformer
      console.log(this.layer.getContext())
      const vm = this

      this.group.remove()
      this.layer.draw()

      stage.on('click', function (e) {
        if (e.target.parent === null) {
          this.selectedShapeName = ''
          transformer.nodes([])
        } else {
          this.selectedShapeName = e.target.attrs.name
          transformer.nodes([e.target])
          e.target.moveUp()
          vm.layer.draw()
        }
      })
    },

    deleteImage() {
      this.layer.find('Transformer').toArray().find(item => {

        if (!item.nodes()[0]) {
          return
        }

        console.log(item.nodes()[0].parent)
        item.nodes()[0].parent.remove()
        item.detach()
      })
      
      this.layer.draw()
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  display: flex;
}

.konvajs-content {
  border: 1px solid gray;
}

.images {
  width: 200px;
  margin-right: 20px;

  &__list {
    display: flex;
    list-style: none;
    margin: 0;
    padding: 0;
  }

  &__item {
    max-width: 100px;
    padding: 0;
    margin: 0 5px 5px;

    & img {
      padding: 0;
      width: 100%;
    }
  }
}

.dragging {
  background-color: #e2e7ff;
}
</style>

<style>
.konvajs-content {
  border: 1px solid gray;
}
</style>
