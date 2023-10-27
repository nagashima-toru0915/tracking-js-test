<template>
    
    <div class="demo-frame">
        <div >
            <v-btn 
                id="video_start"
                variant="tonal"
                v-on:click="click_btn" 
                v-bind:disabled="disable_video_start">
                {{ button_name }}
            </v-btn>
            <v-btn 
                id="capture"
                variant="tonal"
                v-on:click="click_capture_btn" 
                v-bind:disabled="disalbe_capture">
                撮影
            </v-btn>
        </div>
        <div class="demo-container">
            <video 
                class="video_style"
                id="id_video"
                ref="ref_video"
                :width=video_width
                :height=video_height 
                @loadeddata="onVideoPlay"
                muted
                v-show=show_video>
            </video>
            <canvas 
                class="canvas_style"
                id="id_canvas" 
                ref="ref_canvas"
                :width=video_width
                :height=video_height>
            </canvas>
            <canvas
                class="video_canvas_style"
                id="id_videocanvas" 
                ref="ref_videocanvas"
                :width=video_width
                :height=video_height
                v-show=show_videoCanvas>
            </canvas>
            <img id="img" v-show="false">
        </div>
    </div>
</template>
<script>
    require("tracking");
    require('tracking/build/data/mouth');
    require('tracking/build/data/eye');
    require('tracking/build/data/face');

    export default {
        name:'TrackingTest',
        data() {
            const button_name_start = "映像を開始";
            const button_name_end = "映像を停止";

            return {
                button_name: button_name_start,
                button_name_start: button_name_start,
                button_name_end: button_name_end,
                canvasCtx: null,
                videoCanvasCtx: null,
                video_start: false,
                streamingSrc: null,
                ObjectTracker: null,
                videocanvas_interval: null,
                video_width: 800,
                video_height: 600,
                show_video: true,
                show_videoCanvas: false,
                disable_video_start: false,
                disalbe_capture: true,
            }
        },
        mounted() {
            console.log("開始");
        },
        methods: {
            click_btn : function () {
                console.log("click_btn");
                //this.alert("ボタン プッシュ");

                this.videoStart();
            },
            videoStart : async function() {
                if (this.streamingSrc != null) {
                    this.streamingSrc.getTracks().array.forEach(element => {
                        element.stop();
                    });
                }

                
                this.ObjectTracker = new window.tracking.ObjectTracker([ 'mouth', 'eye', 'face' ]);
                

                if (this.video_start) {
                    this.button_name = this.button_name_start;

                    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                        this.id_video.pause();
                        this.id_video.currentTime = 0;

                        await this.videoStop(this.id_video);
                    }

                    if (this.videoCanvasCtx) {
                        this.videoCanvasCtx.clearRect(0, 0, this.video_width, this.video_height);
                    }

                    this.video_start = false;
                    this.disalbe_capture = true;
                } else {
                    this.id_video = this.$refs.ref_video;

                    if (this.canvasCtx) {
                        this.canvasCtx.clearRect(0, 0, this.video_width, this.video_height);
                    }

                    console.log("videoStart: no navigator.webKitGetUserMedia");
                    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
                        navigator.mediaDevices.getUserMedia({
                            video: {
                                facingMode:"user", 
                            },
                            audio: false,
                        }).then(stream => {
                            this.handlesuccess(stream);
                        });
                    }
                }
            },
            handlesuccess: function(stream) {
                console.log("handleSuccess");

                if (this.videocanvas_interval != null) {
                    for (let i = 0; i < this.videocanvas_interval; i++) {
                        clearInterval(this.videocanvas_interval[i]);
                    }
                }
                this.videocanvas_interval = [];

                setTimeout(
                    function(){
                        console.log("handleSuccess 3");
                        this.id_video.srcObject = stream;
                        this.id_video.play();
                    }, 1000);
            },
            videoStop: async function(videoElem) {
                console.log("videoStop");
                const stream = videoElem.srcObject;
                const tracks = stream.getTracks();

                tracks.forEach(function(track) {
                    track.stop();
                });

                videoElem.srcObject = null;

                if (this.videocanvas_interval != null) {
                    for (let i = 0; i < this.videocanvas_interval; i++) {
                        clearInterval(this.videocanvas_interval[i]);
                    }
                }

                this.button_name = this.button_name_start;
                this.video_start = false;
                this.disalbe_capture = true;
            },
            onVideoPlay : async function() {
                console.log("onVideoPlay");
                this.button_name = this.button_name_end;

                this.video_start = true;
                this.disalbe_capture = false;
                this.show_videoCanvas = false;
                this.show_video = true;

                if (this.canvasCtx == null) {
                    this.id_canvas = this.$refs.ref_canvas;

                    this.canvasCtx = this.id_canvas.getContext('2d', { willReadFreququentyly: true });

                    this.canvasCtx.fillStyle = "red";
                    this.canvasCtx.strokeStyle = "red";

                    this.canvasCtx.lineWidth = 4;
                }
                if (this.videoCanvasCtx == null) {
                    this.id_videocanvas = this.$refs.ref_videocanvas;

                    this.videoCanvasCtx = this.id_videocanvas.getContext('2d', { willReadFreququentyly: true });
                }

                console.log(this.videoCanvasCtx);
            },
            click_capture_btn : async function() {
                this.disable_video_start = true;
                if (this.videoCanvasCtx) {
                    //console.log("this.videocanvasCtx clear");
                    this.videoCanvasCtx.clearRect(0, 0, this.video_width, this.video_height);
                    this.videoCanvasCtx.beginPath();
                }
                if (this.canvasCtx) {
                    this.canvasCtx.clearRect(0, 0, this.video_width, this.video_height);
                    this.canvasCtx.beginPath();
                }

                this.videoCanvasCtx.drawImage(this.id_video, 
                    0, 0, this.video_width, this.video_height);
                const img_src = this.id_videocanvas.toDataURL('image/png');

                await this.videoStop(this.id_video);
                this.show_videoCanvas = true;
                this.show_video = false;

                let img = document.getElementById("img");
                img.src = img_src;

                img.addEventListener('load', function(e) {
                    console.log("load");
                    console.log(e);
                    this.ObjectTracker.setStepSize(1.7);

                    window.tracking.track('#img', this.ObjectTracker);

                    this.ObjectTracker.on('track', this.tracker_track_event);
                }.bind(this), false);

                this.disable_video_start = false;
            },
            tracker_track_event: function(event) {
                console.log("track");
                console.log(event.data);

                for (let i = 0; i < event.data.length; i++) {
                    const rect = event.data[i];
                    console.log(rect);

                    const x = rect.x;
                    const y = rect.y;
                    const width = rect.width;
                    const height = rect.height;

                    this.canvas_mouth_stroke(x, y, width, height);
                    //this.canvasCtx.fillText(i, x, y, 18);
                }
            },
            canvas_mouth_stroke: function(x, y, width, height) {
                console.log("canvas_mouth;x:" + x + ";y:" + y);
                this.canvasCtx.strokeRect(x, y, width, height);
            }
        }
    }

</script>
<style>
    .demo-container {
        position: relative;
    }

    .video_style {
        position: absolute;
        left:  0px;                /* 右からの位置指定 */
        top: 0px;   
        z-index: 1;
    }
    
    .video_canvas_style {
        position: absolute;
        left: 0px;
        top: 0px;
        z-index: 2;
    }

    .canvas_style {
        position: absolute;
        left: 0px;
        top: 0px;
        z-index: 3;
    }
</style>