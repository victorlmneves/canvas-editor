<template>
  <section class="container">
    <aside class="images">
      <ul class="images__list">
        <li class="images__item">
          <img
            draggable
            @dragstart="startDrag($event)"
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
            draggable
            @dragstart="startDrag($event)"
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
      @dropend="handlesBoardClick($event)"
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
      imageWidth: 0,
      imageHeight: 0,
      textLabel: '',
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
    const vm = this
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

    const container = this.stage.container()

    container.addEventListener('dragover', function (e) {
      e.preventDefault()
    })

    container.addEventListener('drop', function (e) {
      e.preventDefault()
      vm.addItemToBoard(e)
    })
  },

  methods: {
    startDrag (evt) {
      evt.dataTransfer.dropEffect = 'move'
      evt.dataTransfer.effectAllowed = 'move'
      this.imageUrl = evt.target.src
      this.imageWidth = evt.target.width
      this.imageHeight = evt.target.height
      this.textLabel = evt.target.nextSibling.outerText
    },

    addItemToBoard(evt) {
      let positionX = 50
      let positionY = 50
      let selectedImage = evt.target.src
      let imageWidth = evt.target.width ? evt.target.width : 0
      let imageHeight = evt.target.height ? evt.target.height : 0
      let textLabel = evt.target.nextSibling ? evt.target.nextSibling.outerText : ''

      if (evt.dataTransfer) {
        positionX = evt.layerX - 150
        positionY = evt.layerY - 100
        selectedImage = this.imageUrl
        imageWidth = this.imageWidth
        imageHeight = this.imageHeight
        textLabel = this.textLabel
      }

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

      const imageObj = new Image()
      imageObj.src = selectedImage

      imageObj.onload = function () {
        const maxSize = 100
        const ratio = (imageWidth > imageHeight ? (imageWidth / maxSize) : (imageHeight / maxSize))

        const image = new Konva.Image({
          x: positionX,
          y: positionY,
          image: imageObj,
          width: imageWidth/ratio,
          height: imageHeight/ratio,
          name: `image-${evt.target.id}`
        })

        const labelText = new Konva.Text({
          x: positionX,
          y: positionY + 80,
          text: textLabel,
          fontSize: 14,
          fontFamily: 'Avenir',
          fill: 'black',
          cornerRadius: 10,
          width: 100,
          padding: 5,
          align: 'center'
        })

        const textLabelBg = new Konva.Rect({
          x: positionX,
          y: positionY + 80,
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
        
      this.selectedShapeName = `image-${evt.target.id}`
      this.stage.add(this.layer)
    },

    handlesBoardClick() {
      const stage = this.stage
      const transformer = this.transformer
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
