<style scoped>
    .webcamImage {
        width: 100%;
    }
</style>

<template>
    <video ref="hlsstreamer" autoplay :style="webcamStyle" class="webcamImage" v-observe-visibility="visibilityChanged" />
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

    mounted() {
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
        video.play()
        console.log("hls stream started")
    }

    stopStream(){
        let video = this.$refs["hlsstreamer"];
        video.pause()
        console.log("hls stream stopped")
    }

    visibilityChanged(isVisible:boolean) {
        this.isVisible = isVisible

        if (isVisible) {
            this.startStream()
        } else {
            this.stopStream()
        }
    }
}
</script>