<template>
  <div>
    <div>
      <div class="text-h4">TEXT_DETECTION</div>
    </div>
    <div>
      <form>
        <v-file-input
          accept="image/*"
          label="Image File"
          @change="changeImageFileHandler"
        />
        <v-btn :disabled="imageFile === null" @click="clickLoadHander"
          >LOAD</v-btn
        >
      </form>
    </div>
    <div v-show="isDrawedImage">
      <canvas ref="canvas" class="canvas my-2" />
      <v-btn class="my-2" @click="clickReadTextHander">READ TEXT</v-btn>
      <v-textarea v-model="result" class="my-2" label="結果" readonly />
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

@Component
export default class TextDetectionIndexPage extends Vue {
  imageFile: File | null = null
  isDrawedImage: boolean = false
  result: string = ''

  changeImageFileHandler(file: File) {
    this.imageFile = file
  }

  clickLoadHander() {
    const fileReader = new FileReader()
    console.log(process.env)

    if (!this.imageFile) {
      return
    }

    fileReader.onload = (event) => {
      if (event.target) {
        const result = event.target.result as string
        const canvas = this.$refs.canvas as HTMLCanvasElement
        const context = canvas.getContext('2d')
        if (context) {
          const image = new Image()
          image.src = result
          image.onload = () => {
            canvas.width = image.width
            canvas.height = image.height
            context.drawImage(image, 0, 0)
            this.isDrawedImage = true
          }
        }
      }
    }
    fileReader.readAsDataURL(this.imageFile)
  }

  async clickReadTextHander() {
    const canvas = this.$refs.canvas as HTMLCanvasElement
    const imageContent = canvas
      .toDataURL('image/jpeg')
      .replace(/^data:image\/jpeg;base64,/, '')
    const url = `https://vision.googleapis.com/v1/images:annotate?key=${process.env.apiKey}`
    const data = {
      requests: [
        {
          image: {
            content: imageContent,
          },
          features: [
            {
              type: 'TEXT_DETECTION',
            },
          ],
        },
      ],
    }
    const config = {
      headers: {
        'Content-Type': 'application/json; charset=utf-8',
      },
    }

    const response = await this.$axios.$post(url, data, config)
    console.log(response)
    this.result = response.responses[0].fullTextAnnotation.text
  }
}
</script>

<style scoped lang="scss">
.canvas {
  width: 100%;
}
</style>
