<template>
    <div class="player" v-on:dblclick="handleDoubleClick" @fullscreenchange="handleFullscreenChange">
        <vue3VideoPlay v-bind="options" controls refresh="reloadVideo">
            
        </vue3VideoPlay>
    </div>
</template>
  
<script>
import "vue3-video-play/dist/style.css";
import vue3VideoPlay from "vue3-video-play";
import { reactive, toRefs, onMounted } from "vue";

export default {
    components: {
        vue3VideoPlay,
    },
    data(){
        return {
            options: {
                width: "800px", //播放器高度
                height: "450px", //播放器高度
                color: "#409eff", //主题色
                title: "渔港", //视频名称
                src: "http://220.161.87.62:8800/hls/1/index.m3u8", //视频源
                // src: "//d2zihajmogu5jn.cloudfront.net/sintel/master.m3u8",
                type: 'm3u8', //视频类型
                muted: false, //静音
                webFullScreen: false,
                speedRate: ["0.75", "1.0", "1.25", "1.5", "2.0"], //播放倍速
                autoPlay: true, //自动播放
                loop: false, //循环播放
                mirror: false, //镜像画面
                ligthOff: true, //关灯模式
                volume: 0.3, //默认音量大小
                control: true, //是否显示控制
                controlBtns: [
                    "audioTrack",
                    "quality",
                    "speedRate",
                    "volume",
                    "setting",
                    "pip",
                    "pageFullScreen",
                    "fullScreen",
                    
                ], //显示所有按钮,
            },
            coordsData: [], // to store the coords data from JSON
            mycanvas:null,

        }
    },
    
    mounted() { 
        // create canvas and append to HTML
        document.addEventListener("fullscreenchange", this.fullScreen);

        const canvas = document.createElement("canvas");
        canvas.width = 800; // same size as video
        canvas.height = 450;
        canvas.style.position = "absolute";
        canvas.style.top = "0";
        canvas.style.left = "0";
        canvas.style.zIndex = "3";
        canvas.style.pointerEvents='none';

        this.mycanvas = canvas;
        this.readJson_draw_Coords();
        document.querySelector(".player").appendChild(this.mycanvas);

        const video = document.querySelector("video");
        video.addEventListener("canplay",()=>{
           
            video.addEventListener("timeupdate",this.readJson_draw_Coords);
        });

    },
    methods:{
        readJson_draw_Coords(){
            const filename="../../data/landdata.json";
             // fetch JSON data and parse the coords array from each object
            fetch(filename)
                .then((response) => response.json())
                .then((json) => {
                    this.coordsData = json.extra.data.map((d) => d.coords);
                    const video = document.querySelector("video");
                    const currentTime = video.currentTime;
                    
                    const ctx = this.mycanvas.getContext("2d");
                    ctx.clearRect(0, 0, this.mycanvas.width, this.mycanvas.height); // clear canvas
                    this.coordsData.forEach((coords) => {
                        this.drawPolygon(ctx, coords, currentTime);
                    });
            });
            
        },

        fullScreen(){
            const isFullScreen = document.fullscreenElement !=null;
            console.log(document.fullscreenElement);

            console.log(isFullScreen)
            if (isFullScreen){
                

                console.log('full screen')
            }
        },

        // helper function to draw a polygon onto a canvas
        drawPolygon(ctx, coords, currentTime) {
            ctx.strokeStyle = '#ff0000';
            ctx.lineWidth = 5;
            ctx.beginPath();
            // console.log(coords);
            const firstCoord = coords[0];
            // console.log(firstCoord.x);
            ctx.moveTo(firstCoord.x | 0, firstCoord.y | 0);
            coords.slice(1).forEach((coord) => {
                ctx.lineTo(coord.x | 0, coord.y | 0);
            });
            ctx.closePath();

            ctx.stroke();
        },

        handleDoubleClick() {
            const video = document.querySelector('video');
            document.fullscreenElement=null;
            const is_full_screen=document.fullscreenElement !=null;
            if(is_full_screen){
                console.log('how to close full screen')
            }
        },

        handleFullscreenChange(event) {
            console.log("change");
            const videoElement = event.target;
            const fullscreenElement = document.fullscreenElement;

            var convas_cache = this.mycanvas;
            if (fullscreenElement === videoElement) {
                console.log("add element");
                // 进入全屏状态，添加你的内容
                // full screen
                const canvas = document.createElement("canvas");
                canvas.width = 800; // same size as video
                canvas.height = 450;
                canvas.style.position = "absolute";
                canvas.style.top = "0";
                canvas.style.left = "0";
                canvas.style.zIndex = "3";
                canvas.style.pointerEvents='none';

                videoElement.appendChild(canvas);
                this.mycanvas=canvas;
            } else {
                console.log("remove elemnent");
                // 退出全屏状态，移除你的内容
                const fullscreenContent = videoElement.querySelector('canvas');
                if (fullscreenContent) {
                    videoElement.removeChild(fullscreenContent);
                    this.mycanvas=convas_cache;
                }
            }
        },
        reloadVideo() {
            const videoPlayer = document.querySelector('video');
            videoPlayer.currentTime(0);
            videoPlayer.play();
        },

    }
};
</script>
  
  
<style>
.player {
    position: relative;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    margin: auto;
    z-index: 0;
}

.canvas {
    position: fixed;
    top: 0;
    left: 0;
    z-index: 1000;
}
</style>
  