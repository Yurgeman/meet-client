<template>
  <div class="peer item"
       :class="{ '-maximized': state.maximized === id }"
       @click="handleClick">
    <div
      v-if="state.screenshots"
      :style="`background:url(/faces/${screenshotNumber}.jpg); background-size: cover; background-position: center;`"
      class="video"
      :class="{ '-mirrored': mirrored }"/>

    <video
      class="video"
      :class="{ '-mirrored': mirrored }"
      ref="video"
      autoplay
      playsinline
      webkit-playsinline
      :muted="muted"
      v-else-if="stream"
      :data-fit="state.fill ? 'cover' : 'contain'"/>
    <div v-else
         class="video video-placeholder -content-placeholder">
      <svg xmlns="http://www.w3.org/2000/svg"
           width="24"
           height="24"
           viewBox="0 0 24 24"
           fill="none"
           stroke="currentColor"
           stroke-width="2"
           stroke-linecap="round"
           stroke-linejoin="round"
           class="-icon-placeholder">
        <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"></polygon>
      </svg>
      <label>Ожидание подключения</label>
    </div>
    <div v-if="fingerprint"
         class="video video-placeholder video-fingerprint -content-placeholder -overlay -info"
         v-show="!state.maximized">
      <!--<label title="Verification code"-->
      <!--       class="-short"-->
      <!--       @click.stop.prevent="doToggleShow"-->
      <!--       v-show="!showCode">-->
      <!--  <svg xmlns="http://www.w3.org/2000/svg"-->
      <!--       width="24"-->
      <!--       height="24"-->
      <!--       viewBox="0 0 24 24"-->
      <!--       fill="none"-->
      <!--       stroke="currentColor"-->
      <!--       stroke-width="2"-->
      <!--       stroke-linecap="round"-->
      <!--       stroke-linejoin="round"-->
      <!--       class="feather feather-shield">-->
      <!--    <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"></path>-->
      <!--  </svg>-->
      <!--  {{ fingerprint.substr( fingerprint.length - 4, 4 ) }}-->
      <!--</label>-->
      <!--<label title="Verification code"-->
      <!--       class="-long"-->
      <!--       @click.stop.prevent="doToggleShow"-->
      <!--       v-show="showCode">If the person you see here confirms to see the same ID, you are securely-->
      <!--  connected:<br/>-->
      <!--  <tt>{{ fingerprint }}</tt>-->
      <!--</label>-->
      <!--<label title="Name"-->
      <!--       class="-short"-->
      <!--       style="right: 5rem">-->
      <!--  {{ name }}-->
      <!--</label>-->
    </div>
    <div v-if="state.muteVideo && id === 'self'"
         class="video video-placeholder -content-placeholder">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="-icon-placeholder">
        <path d="M16 16v1a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V7a2 2 0 0 1 2-2h2m5.66 0H14a2 2 0 0 1 2 2v3.34l1 1L23 7v10"></path>
        <line x1="1"
              y1="1"
              x2="23"
              y2="23"></line>
      </svg>
      <label>You turned the video off</label>
    </div>
    <div v-if="stream && showPlayButton"
         class="video video-placeholder -content-placeholder -overlay">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="-icon-placeholder"
      >
        <circle cx="12"
                cy="12"
                r="10"></circle>
        <polygon points="10 8 16 12 10 16 10 8"></polygon>
      </svg>
      <label>Click to start video</label>
    </div>

    <button class="pd-video-conf pd-video-conf--close"
            @click="callOff"
            v-html="icons.callOff"></button>

  </div>
</template>

<script>
import { trackSilentException } from '../bugs'
//import enableInlineVideo        from 'iphone-inline-video'
import { Logger }               from '../lib/logger'

const log = Logger( 'app:app-peer' )

window.screenshotNumber = 0

export default {
  name: 'app-video',

  props: {
    stream:      {
      type:    MediaStream | Object,
      default: null
    },
    active:      {
      type:    Boolean,
      default: false
    },
    muted:       {
      type:    Boolean,
      default: false
    },
    title:       {
      type: String
    },
    mirrored:    {
      type:    Boolean,
      default: false
    },
    fingerprint: {
      type: String
    },
    id:          {
      type: String
    },
    name:        {
      type: String
    }
  },

  data () {
    return {
      screenshotNumber: ++window.screenshotNumber,
      showCode:         false,
      showPlayButton:   false,
      icons:            {
        callOff:  `<svg xmlns="http://www.w3.org/2000/svg" width="44" height="44" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
  <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
  <line x1="3" y1="21" x2="21" y2="3" />
  <path d="M5.831 14.161a15.946 15.946 0 0 1 -2.831 -8.161a2 2 0 0 1 2 -2h4l2 5l-2.5 1.5c.108 .22 .223 .435 .345 .645m1.751 2.277c.843 .84 1.822 1.544 2.904 2.078l1.5 -2.5l5 2v4a2 2 0 0 1 -2 2a15.963 15.963 0 0 1 -10.344 -4.657" />
</svg>`,
        maximize:
                  `<svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-maximize" width="44" height="44" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
          <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
          <path d="M4 8v-2a2 2 0 0 1 2 -2h2" />
          <path d="M4 16v2a2 2 0 0 0 2 2h2" />
          <path d="M16 4h2a2 2 0 0 1 2 2v2" />
          <path d="M16 20h2a2 2 0 0 0 2 -2v-2" />
        </svg>`,
        minimize: `<svg xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-minimize" width="44" height="44" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" fill="none" stroke-linecap="round" stroke-linejoin="round">
          <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
          <path d="M15 19v-2a2 2 0 0 1 2 -2h2" />
          <path d="M15 5v2a2 2 0 0 0 2 2h2" />
          <path d="M5 15h2a2 2 0 0 1 2 2v2" />
          <path d="M5 9h2a2 2 0 0 0 2 -2v-2" />
        </svg>`
      }

    }
  },

  methods: {

    async callOff () {
      await window.close()
    },

    async playVideo ( video ) {
      let startPlayPromise = await video.play()
      console.log( '---- video', video )
      log( 'play', startPlayPromise )
      if ( startPlayPromise !== undefined ) {
        startPlayPromise
          .then( () => {
            // Start whatever you need to do only after playback
            // has begun.
          } )
          .catch( ( error ) => {
            if ( error.name === 'NotAllowedError' ) {
              this.showPlayButton = true
            }
            else {
              trackSilentException( error )
            }
          } )
      }
    },

    async doConnectStream ( stream ) {
      log( 'doConnectStream', this.title, stream )
      if ( stream ) {
        try {
          await this.$nextTick()
          let video = this.$refs.video
          log( 'connectStreamToVideoElement', stream, video )
          if ( stream ) {
            if ( 'srcObject' in video ) {
              video.srcObject = stream
            }
            else {
              video.src = window.URL.createObjectURL( stream ) // for older browsers
            }
            //enableInlineVideo(video, { iPad: false });

            // Keep in mind https://developers.google.com/web/updates/2017/09/autoplay-policy-changes
            // But if the user allows to access camera it should be fine
            // https://developer.mozilla.org/en-US/docs/Web/Media/Autoplay_guide
            video.onloadedmetadata = ( e ) => this.playVideo( video )
            video.onloadeddata     = ( e ) => this.playVideo( video )
          }
        }
        catch ( err ) {
          trackSilentException( err )
        }
      }
    },

    handleClick () {
      if ( this.showPlayButton ) {
        this.doPlay()
      }
      else if ( this.state.maximized === this.id ) {
        this.state.maximized = ''
      }
      else {
        this.state.maximized = this.id
      }
    },

    doToggleShow ( ev ) {
      this.showCode = !this.showCode
    },

    async doPlay () {
      try {
        log( 'force play manually' )
        await this.$refs.video.play()
        this.showPlayButton = false
      }
      catch ( err ) {
        trackSilentException( err )
      }
    }
  },

  async mounted () {
    // webrtc.on('stream', async () => {
    //   await this.$nextTick()
    //   await this.doConnectStream(this.stream)
    // })
    if ( this.stream ) {
      await this.doConnectStream( this.stream )
    }
  },

  watch: {
    stream ( value ) {
      this.doConnectStream( value )
    }
  }
}
</script>
<style lang="sass">
@import "../css/variables"
$buttonSize: 3rem

.pd-video-conf
  position: absolute
  bottom: 0
  left: 50%
  margin: 1rem
  transform: translateX(-50%)
  cursor: pointer

  &--close
    background-color: rgba(215, 62, 72, 0.8)
    border-radius: 500px
    padding: 0
    display: flex
    align-items: center
    justify-content: center
    color: rgba(255, 255, 255, 1)
    width: $buttonSize + 1rem
    height: $buttonSize + 1rem
    z-index: 30

    &:hover
      background-color: rgba(215, 62, 72, 1)

    & > *
      width: $buttonSize * .7
      height: $buttonSize * .7
</style>
