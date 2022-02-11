<style scoped>
    .webcamImage {
        width: 100%;
    }
</style>

<template>
    <video controls ref="hlsstreamer" autoplay :style="webcamStyle" class="webcamImage" v-observe-visibility="visibilityChanged" />
</template>

<script lang="ts">



import {Component, Mixins, Prop} from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import Hls from 'hls.js';

@Component
export default class Hlsstreamer extends Mixins(BaseMixin) {
    private isVisible = true
    private hlsObject : Hls | null = null

    declare $refs: {
        hlsstreamer: any
    }

    @Prop({ required: true })
    camSettings: any

    @Prop()
    printerUrl: string | undefined

    get webcamStyle() {
        let transforms = ''
        if ('flipX' in this.camSettings && this.camSettings.flipX) transforms += ' scaleX(-1)'
        if ('flipX' in this.camSettings && this.camSettings.flipY) transforms += ' scaleY(-1)'
        if (transforms.trimLeft().length) return { transform: transforms.trimLeft() }

        return ''
    }

    instantiatePlayer(){
        this.hlsObject = new Hls();
        let stream = this.camSettings.urlStream
        let video = this.$refs["hlsstreamer"];
        this.hlsObject.loadSource(stream);
        this.hlsObject.attachMedia(video);
        this.hlsObject.on(Hls.Events.MANIFEST_PARSED, function () {
            video.play();
        });
    }

    startStream(){
        let video = this.$refs["hlsstreamer"];
        if(this.hlsObject){
            video.play()
            this.seekToLive()
        }else{
            this.instantiatePlayer()
        }
    }

    seekToLive(){
        let video = this.$refs["hlsstreamer"];
        if(!isNaN(video.duration)){
            console.log("video current time: " + video.currentTime)
            console.log("video duration: " + video.duration)
            video.currentTime = video.duration - 4
        }
    }

    stopStream(){
        let video = this.$refs["hlsstreamer"];
        video.pause()
    }

    created () {
        document.addEventListener('visibilitychange', this.handleDocumentVisibility, false)
    }

    // Used to:
    //     - destroy the HLS stream when the document is no longer visible (e.g., the user has clicked on another tab)
    //     - recreate the HLS stream when the document is again visible (e.g., the user is back on the mainsail tab)
    handleDocumentVisibility(){
        // Always recreate the stream to synchronize it
        if(this.hlsObject){
            this.hlsObject.destroy()
            this.hlsObject = null
        }
        if (document.visibilityState === 'visible') {
            this.startStream()
        }
    }

    visibilityChanged(isVisible:boolean) {
        this.isVisible = isVisible

        if (isVisible) {
            this.startStream()

        } else {
            this.stopStream()
        }
    }

    beforeDestroy() {
        if(this.hlsObject){
            this.hlsObject.destroy()
        }
        document.removeEventListener('visibilitychange', this.handleDocumentVisibility, false)
    }
}
</script>