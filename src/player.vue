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
//require('videojs-settings-menu/dist/videojs-settings-menu.css');
//plugins
//window.videojs = require('src/plugins/video.js')
//require('src/plugins/videojs-settings-menu');
//require('src/plugins/videojs-settings-menu.css');
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

      // videojs options
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
        // nativeCaptions: true,
        // nativeTextTracks: true,
        // controlBar: {
        //   // remainingTimeDisplay: false,
        //   subtitlesButton: true, //字幕
        //   captionsButton: true, //字幕带设置
        //   chaptersButton: true,
        //   // liveDisplay: false,
        //   // playbackRateMenuButton: false,
        //   // playToggle: {},
        //   // progressControl: {},
        //   // fullscreenToggle: {},
        //   // subtitlesButton: {},
        //   // volumeMenuButton: {
        //   //   inline: false,
        //   //   vertical: true
        //   // }
        // },
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
        html5: {
          nativeAudioTracks: false,
          nativeVideoTracks: false,
          nativeTextTracks: true
        },
      }, this.options)

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

      // videoOptions
      //console.log(videoOptions)
      // var xmlhttp;
      // var listak = [];
      // if (videoOptions.sources) {
      //   if (window.XMLHttpRequest) { // code for IE7+, Firefox, Chrome, Opera, Safari
      //     xmlhttp = new XMLHttpRequest();
      //   } else { // code for IE6, IE5
      //     xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
      //   }
      //   xmlhttp.onreadystatechange = function() {
      //     if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
      //       //console.log(xmlhttp.responseText)
      //       var parser = new m3u8Parser.Parser();

      //       parser.push(xmlhttp.responseText);
      //       parser.end();

      //       var parsedManifest = parser.manifest;
      //       listak = parsedManifest.playlists;
      //       siso();
      //       //console.log(parsedManifest)
      //       //console.log(listak)
      //       //document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
      //     }
      //   }
      //   xmlhttp.open("GET", videoOptions.sources[0].src, true);
      //   xmlhttp.send();
      // }

      function siso() {
        
        if (listak.length) {
          var current_list = self.player.hls.playlists.master.playlists;
        console.log(current_list)



          videojs.Hls.xhr.beforeRequest = function(options) {
            //console.log(options)
            if (listak.length) {
              var opsk = listak[0].uri.slice(0, -2); //获取多码率子m3u8共同有的
              var sowkls = options.uri.indexOf(opsk)
              // //console.log(sowkls)
              var dlp = options.uri.slice(30);
              var dlasp = listak[0].uri;
              var ops = listak[1].uri;
              //console.log(options.uri)
              //console.log(dlp)

              if (sowkls != -1) {
                // console.log(options.uri)
                // console.log(dlasp)
                // console.log(ops)
                options.uri = options.uri.replace(dlasp, dlasp); //(dlasp, ops);
              }
              // console.log(options.uri.slice(32))
              // console.log(listak[1].uri)
              // console.log(options.uri)
              return options;
            }

          };
        }
      };




      







      // avoid error "VIDEOJS: ERROR: Unable to find plugin: __ob__"
      if (videoOptions.plugins) {
        delete videoOptions.plugins.__ob__
      }
      var vjs_ass;

      this.$el.children[0].crossOrigin = "anonymous";

      this.player = videojs(this.$el.children[0], videoOptions, function() {
        
        /*if (videoOptions.ass) {
          vjs_ass = this.ass(videoOptions.ass);
        }*/
        //this.nativeTextTracks = true;
        if (videoOptions.ass) {
          var track1 = {
            kind: 'subtitles',
            src: videoOptions.ass,
            srclang: 'en',
            label: 'chinasub',
            default: 'default'
          };
          this.addRemoteTextTrack(track1, false);

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
          if(videoOptions.sources){
            //console.log(this.tech_.hls.playlists.master.playlists)
            var current_list = this.tech_.hls.playlists.master.playlists;
            //console.log(current_list)
            var lsee=this.tech_.hls.representations();
            for (var i = 0; i < lsee.length; i++) {
              if(lsee[i].height){
                if(lsee[i].height==480){
                  //this.tech_.hls.playlists.media(current_list[0]);
                  //lsee[i].enabled(true);
                  //break;
                }else if(lsee[i].height==720){
                  this.tech_.hls.playlists.media(current_list[1]);
                  lsee[i].enabled(true);
                  break;
                }else if(lsee[i].height==1080){
                  this.tech_.hls.playlists.media(current_list[0]);
                  lsee[i].enabled(true);
                  break;
                }else if(lsee[i].height==1440){
                  this.tech_.hls.playlists.media(current_list[0]);
                  lsee[i].enabled(true);
                  break;
                }else if(lsee[i].height==2160){
                  this.tech_.hls.playlists.media(current_list[0]);
                  lsee[i].enabled(true);
                  break;
                }

              }
            }
            this.handleTechPlaying_();
          }
        })

        //console.log(this)
        this.on('loadeddata', function() {
          if(videoOptions.sources){
            //console.log(this.tech_.hls.playlists.master.playlists)
            var current_list = this.tech_.hls.playlists.master.playlists;
            //console.log(current_list)
            var lsee=this.tech_.hls.representations();
            for (var i = 0; i < lsee.length; i++) {
              if(lsee[i].height){
                if(lsee[i].height==480){
                  //this.tech_.hls.playlists.media(current_list[0]);
                  //lsee[i].enabled(true);
                  //break;
                }else if(lsee[i].height==720){
                  this.tech_.hls.playlists.media(current_list[1]);
                  lsee[i].enabled(true);
                  break;
                }else if(lsee[i].height==1080){
                  this.tech_.hls.playlists.media(current_list[0]);
                  lsee[i].enabled(true);
                  break;
                }else if(lsee[i].height==1440){
                  this.tech_.hls.playlists.media(current_list[0]);
                  lsee[i].enabled(true);
                  break;
                }else if(lsee[i].height==2160){
                  this.tech_.hls.playlists.media(current_list[0]);
                  lsee[i].enabled(true);
                  break;
                }

              }
            }
            this.handleTechPlaying_();
          }
        })






      })
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
