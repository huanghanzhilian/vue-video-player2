<template>
  <div class="video-player">
    <video class="video-js" controls crossorigin="anonymous">
      <!-- <source src="//vjs.zencdn.net/v/oceans.mp4" type='video/mp4'>
      <source src="//vjs.zencdn.net/v/oceans.webm" type='video/webm'> -->
      <!-- <track kind="subtitles" src="http://test.huanghanlian.com/timedtext.vtt" srclang="en" label="English" default> -->
    </video>
  </div>
</template>
<script>
window.videojs = require('video.js');
window.m3u8Parser = require('m3u8-parser');
require('videojs-contrib-quality-levels');
//window.libjass = require('libjass')
//require('videojs-settings-menu');
//var sssswwo = require('videojs-settings-menu/dist/videojs-settings-menu');

videojs = videojs.default || videojs
export default {
  name: 'video-player',
  data() {
    return {

    }
  },
  props: {
    options: {
      type: Object,
      required: true
    },
    start: {
      type: Number,
      default: function() {
        return 0
      }
    },
    playsinline: {
      type: Boolean,
      default: function() {
        return false
      }
    },
    customEventName: {
      type: String,
      default: function() {
        return 'statechanged'
      }
    }
  },
  mounted: function() {
    if (this.player) {
      this.initialize()
    }
  },
  beforeDestroy: function() {
    if (this.player) {
      this.dispose()
    }
  },
  methods: {
    initialize: function() {

      // init
      var self = this
      this.player = null

      // videojs options 参数
      var videoOptions = Object.assign({
        autoplay: false,
        controls: true,
        preload: 'auto',
        fluid: false,
        muted: false,
        width: '100%',
        height: '100%',
        language: 'en',
        "nativeControlsForTouch": true, //是否使用默认播放器控件
        customControlsOnMobile: true, //移动端
        controlBar: {
          children: {
            'playToggle': {},
            'muteToggle': {},
            'volumeControl': {},
            'currentTimeDisplay': {},
            'timeDivider': {},
            'durationDisplay': {},
            'liveDisplay': {},
            'flexibleWidthSpacer': {},
            'progressControl': {},
            //'captionsButton': true,
            'subtitlesButton': true,
            //'chaptersButton': true,
            // 'settingsMenuButton': {
            //   entries: [
            //     'subtitlesButton',
            //     //'playbackRateMenuButton'
            //     // 'subtitlesButton',
            //     //'captionsButton',
            //     // 'playbackRateMenuButton',
            //     // 'qualityMenuButton'
            //   ]
            // },
            'fullscreenToggle': {}
          }
        },
        techOrder: ['html5'],
        plugins: {},
        // hls:{
        //   overrideNative:true
        // },
        html5: {
          nativeAudioTracks: false,
          nativeVideoTracks: false,
          nativeTextTracks: true,
          hls: {
            //withCredentials: true,
            //debug: false,
            overrideNative: true
          }
        },
      }, this.options)

      //console.log(videoOptions)


      // check sources
      /*
      if (!videoOptions.sources || !videoOptions.sources.length) {
        console.warn('Missing required option: "sources".')
        return false
      }
      */

      // ios fullscreen
      var playsinline = this.playsinline
      if (playsinline) {
        this.$el.children[0].setAttribute('playsinline', playsinline)
        this.$el.children[0].setAttribute('webkit-playsinline', playsinline)
      }

      // emit event
      var emitPlayerState = function(event, value) {
        if (event) {
          self.$emit(event, self.player)
        }
        if (value) {
          var values = {}
          values[event] = value
          self.$emit(self.customEventName, values)
        }
      }

      // avoid error "VIDEOJS: ERROR: Unable to find plugin: __ob__"
      if (videoOptions.plugins) {
        delete videoOptions.plugins.__ob__
      }
      var vjs_ass;

      this.$el.children[0].crossOrigin = "anonymous";

      this.player = videojs(this.$el.children[0], videoOptions, function() {
        //修改视频src
        this.on('ready',function(){
          // this.src({
          //   src: 'http://djyjlsv34fx05.cloudfront.net/10minscat/10minscat.m3u8',
          //   type: "application/x-mpegURL",
          //   withCredentials: false
          // })
        })
        //加入请求头
        // this.hls.xhr.beforeRequest = function (options) {
        //     //console.log(options)
        //     //options.headers = options || {};
        //     //options.headers['token'] = localStorage.getItem('token');
        // }
        
        /*if (videoOptions.ass) {
          vjs_ass = this.ass(videoOptions.ass);
        }*/
        //this.nativeTextTracks = true;
        if (videoOptions.ass) {
          //console.log(videoOptions.ass)
          var track1 = {
            kind: 'subtitles',
            src: videoOptions.ass,
            srclang: 'en',
            label: 'chinasub',
            default: 'default'
          };
          console.log(track1)
          this.addRemoteTextTrack(track1, true);

          var tracks = this.textTracks();
          tracks.on('change', function() {
            for (let i = 0; i < this.length; i++) {
              let track = this[i];
              //alert(JSON.stringify(track))
              // find the captions track that's in english
              if (track.kind == 'subtitles' && track.label == "chinasub") {
                track.mode = 'showing'; //showing   disabled
              }
            }
          })
        }

        

        // player readied
        var _this = this
        self.$emit('ready', self.player)
        // events
        var events = ['loadeddata',
          'progress',
          'canplay',
          'canplaythrough',
          'play',
          'pause',
          'waiting',
          'playing',
          'ended',
          'error'
        ]
        for (var i = 0; i < events.length; i++) {
          (function(event) {
            _this.on(event, function() {
              emitPlayerState(event, true)
            })
          })(events[i])
        }

        this.on('timeupdate', function() {
          emitPlayerState('timeupdate', this.currentTime())
        })
        this.on('progress',function(){
          // var that=this
          // if(videoOptions.sources){
          //   //console.log(this.tech_.hls.playlists.master.playlists)
          //   var current_list = this.tech_.hls.playlists.master.playlists;
          //   //console.log(current_list)
          //   var lsee=this.tech_.hls.representations();
          //   var tsjow;
          //   for (var i = 0; i < lsee.length; i++) {
          //     if(lsee[i].height){
          //       if(lsee[i].height==480){
          //         tsjow=480
          //         break;
          //       }else if(lsee[i].height==720){
          //         tsjow=720
          //         break;
          //       }else if(lsee[i].height==1080){
          //         tsjow=1080
          //         break;
          //       }else if(lsee[i].height==1440){
          //         tsjow=1440
          //         break;
          //       }else if(lsee[i].height==2160){
          //         tsjow=2160
          //         break;
          //       }

          //     }
          //   }
          //   lsee.forEach(function(rep,index) {
          //     if(rep.height === tsjow){
          //       that.tech_.hls.playlists.media(current_list[index]);
          //       rep.enabled(true);
          //     }else{
          //       rep.enabled(false);
          //     }
          //   })
          //   this.handleTechPlaying_();
          // }
        })

        //
        this.on('loadeddata', function() {
          // var that=this
          // if(videoOptions.sources){
          //   //console.log(this.tech_.hls.playlists.master.playlists)
          //   var current_list = this.tech_.hls.playlists.master.playlists;
          //   //console.log(current_list)
          //   var lsee=this.tech_.hls.representations();
          //   var tsjow;
          //   for (var i = 0; i < lsee.length; i++) {
          //     if(lsee[i].height){
          //       if(lsee[i].height==480){
          //         tsjow=480
          //         break;
          //       }else if(lsee[i].height==720){
          //         tsjow=720
          //         break;
          //       }else if(lsee[i].height==1080){
          //         tsjow=1080
          //         break;
          //       }else if(lsee[i].height==1440){
          //         tsjow=1440
          //         break;
          //       }else if(lsee[i].height==2160){
          //         tsjow=2160
          //         break;
          //       }

          //     }
          //   }
          //   lsee.forEach(function(rep,index) {
          //     if(rep.height === tsjow){
          //       that.tech_.hls.playlists.media(current_list[index]);
          //       rep.enabled(true);
          //     }else{
          //       rep.enabled(false);
          //     }
          //   })
          //   console.log(tsjow)
          //   this.handleTechPlaying_();
          // }
        })


      })
      //修改已经初始化了的video
      // this.player.src({
      //   src: videoOptions.m3u8Urisave,
      //   type: "application/x-mpegURL",
      //   withCredentials: false
      // });

    },
    dispose: function() {
      if (this.player && videojs) {
        if (this.player.techName_ !== 'Flash') {
          this.player.pause && this.player.pause()
        }
        videojs(this.$el.children[0]).dispose()
        if (!this.$el.children.length) {
          var video = document.createElement('video')
          video.className = 'video-js'
          this.$el.appendChild(video)
        }
        this.player = null
      }
    }
  },
  watch: {
    options: {
      deep: true,
      handler: function(options, oldOptions) {
        this.dispose()
        if (options && options.sources && options.sources.length) {
          this.initialize()
        }
      }
    }
  }
}

</script>
