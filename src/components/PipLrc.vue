<template>
  <div class="piplyc">
    <canvas ref="piplrc-cvs" style="display: none"></canvas>
    <video
      ref="piplrc-video"
      style="display: none"
      @leavepictureinpicture="quit"
    ></video>
  </div>
</template>
<script>
import { mapState } from 'vuex'
import NotifyMixin from 'src/mixins/Notification'

/**
 * 桌面字幕
 */
// 初始化 = 初始化video + 初始化canvas + 连接 +

export default {
  name: 'PipLrc',
  mixins: [NotifyMixin],
  data() {
    return { pipWindow: null }
  },
  computed: {
    ...mapState('AudioPlayer', ['currentLyric', 'playing', 'desktopLrc'])
  },
  watch: {
    currentLyric(val) {
      this.desktopLrc && this.draw(val, this.$refs['piplrc-cvs'])
    },
    playing(val) {
      /**
       * @type {HTMLVideoElement}
       */
      const video = this.$refs['piplrc-video']
      this.desktopLrc && (val ? video.play() : video.pause())
    },
    desktopLrc(val) {
      val ? this.enablePip() : this.exitPip()
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    enablePip() {
      /**
       * @type {HTMLVideoElement}
       */
      const video = this.$refs['piplrc-video']
      /**
       * @type {HTMLCanvasElement}
       */
      const canvas = this.$refs['piplrc-cvs']
      if (
        typeof video.requestPictureInPicture === 'function' &&
        document.pictureInPictureEnabled
      ) {
        video.requestPictureInPicture().catch((err) => {
          this.showErrNotif(err.message)
          this.quit()
        })
      } else if (typeof video.webkitPresentationMode === 'function') {
        video.webkitPresentationMode('picture-in-picture')
      }
      this.draw(this.currentLyric, canvas)
      video.play()
    },
    exitPip() {
      /**
       * @type {HTMLVideoElement}
       */
      const video = this.$refs['piplrc-video']
      if (typeof document.exitPictureInPicture === 'function') {
        document.pictureInPictureElement && document.exitPictureInPicture()
      } else if (typeof video.webkitPresentationMode === 'function') {
        video.webkitSetPresentationMode('inline')
      }
      video.pause()
    },
    init() {
      // 初始化
      /**
       * @type {HTMLVideoElement}
       */
      const video = this.$refs['piplrc-video']
      /**
       * @type {HTMLCanvasElement}
       */
      const canvas = this.$refs['piplrc-cvs']
      this.draw('', canvas)
      canvas.width = window.innerWidth * 2
      canvas.height = (canvas.width / 20) * 3
      video.srcObject = canvas.captureStream()

      this.playing ? video.play() : video.pause()
    },
    quit() {
      console.warn('[PipLrc] QUIT!')
      this.$store.commit('AudioPlayer/TOGGLE_DESKTOPLRC', false)
      this.$store.commit('AudioPlayer/PAUSE')
    },
    /**
     * @param {string} str 字幕
     * @param {HTMLCanvasElement} cvs
     */
    draw(str, cvs) {
      const ctx = cvs.getContext('2d')
      const fontsize = cvs.width / 40
      ctx.clearRect(0, 0, cvs.width, cvs.height)
      ctx.fillStyle = 'rgba(255,255,255)'
      ctx.fillRect(0, 0, cvs.width, cvs.height)
      ctx.font = `bold ${
        fontsize * 0.9
      }px "-apple-system", "BlinkMacSystemFont", "Segoe UI", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "Helvetica Neue", "Helvetica", "Arial", "sans-serif", "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol"`
      ctx.fillStyle = '#9c27b0'

      // 可绘制参数
      const padding = [10, 15]
      const allowLines = Math.floor((cvs.height - padding[0] * 2) / fontsize)
      const allowWidth = cvs.width - padding[1] * 2

      // 首次测量全部字符串
      const allTxt = ctx.measureText(str)
      // 预计行数
      const actualLine = Math.ceil(allTxt.width / allowWidth)

      // 小于可写宽度 => 就地绘制
      if (allTxt.width <= allowWidth) {
        const ltx = ((cvs.width - allTxt.width) >> 1) + padding[1]
        const lty = ((cvs.height - fontsize) >> 1) + padding[0]
        ctx.fillText(str, ltx, lty + fontsize)
        return
      }
      // 大于可写宽度 => 换行绘制
      const chars = str.split('')
      let line = ''
      let x = padding[0],
        y =
          padding[1] +
          fontsize +
          (actualLine >= allowLines
            ? 0
            : (cvs.height - actualLine * fontsize) / 2)

      for (
        let n = 0, curLine = 1;
        n < chars.length && allowLines >= curLine;
        n++
      ) {
        const newline = line + chars[n]
        const metrics = ctx.measureText(newline)
        const nlineWidth = metrics.width
        // 超出 ?
        if (nlineWidth > allowWidth && n > 0) {
          // 非尾行超出
          if (curLine < allowLines) {
            ctx.fillText(line, x, y)
            curLine++
            line = chars[n]
            y += fontsize
          } else {
            ctx.fillText(line + '...', x, y)
            curLine++
          }
        } else {
          line = newline
          if (n === chars.length - 1) {
            const lastLine = ctx.measureText(line)
            ctx.fillText(line, (cvs.width - lastLine.width) >> 1, y)
          }
        }
      }
    }
  }
}
</script>
<style>
.piplrc-lrc {
  font-size: 16px;
  align-items: center;
  padding: 4px 10px;
  color: #fbc2eb;
}
</style>
