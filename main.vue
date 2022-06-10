    
	
	<div style="height: 800px; width: 100%" id="element-to-record">
          <video class="prism-player" style="height: 730px; width: 100%" id="my-video"  ref="videoElement" controls autoplay muted></video>
          <canvas id="background-canvas" width="100%"   height="800" style="position: absolute; top: -99999999px; left: -9999999999px"  />
          <el-button style="margin-left: 35%; width: 150px; height: 50px" type="primary"@click="startRecording" :disabled="adisabled" size="" >开始录像</el-button>
          <el-button style="width: 150px; height: 50px" type="danger"  @click="stopRecording" :disabled="bdisabled" size="" id="btn-stop-recording" >停止录像 </el-button>
          <el-button
            @click="saveRecord()"
            icon="el-icon-finished"
            style="width: 150px; height: 50px"
            type="success"
            >保存
          </el-button>
        </div>
		
		
//JS 部分

//播放视频
    startPlay() {


      setTimeout(() => {
        if (flvjs.isSupported()) {
          let player = null;
          let videoElement = document.getElementById("my-video");
          console.log("play___________________" + videoElement);
          player = flvjs.createPlayer({
            type: "flv", //=> 媒体类型 flv 或 mp4
            isLive: true, //=> 是否为直播流
            hasAudio: false, //=> 是否开启声音
            url: "https://sf1-hscdn-tos.pstatp.com/obj/media-fe/xgplayer_doc_video/flv/xgplayer-demo-360p.flv", //=> 视频流地址
          });
          player.attachMediaElement(videoElement); //=> 绑DOM
          player.load();
          player.play();
          this.elementToRecord = document.getElementById("element-to-record");
          this.canvas2d = document.getElementById("background-canvas");
          this.context = this.canvas2d.getContext("2d");
          const w = window.innerWidth;
          const h = (window.innerWidth / 16) * 9;
          this.canvas2d.width = w;
          this.canvas2d.height = h;
        } else {
          console.log("flvjs不支持");
        }
      }, 1);
    },

//开始录制
    startRecording() {
      this.isStoppedRecording = false;
      this.isRecordingStarted = true;
      console.log(this.currentRow);

      this.recorder = RecordRTC(this.canvas2d, {
        type: "canvas",
      });
      this.recorder.startRecording();
      this.adisabled = true;
      this.bdisabled = false;
      this.looper();
    },
//停止录制
   stopRecording() {
      const _this = this;
      this.recorder.stopRecording(function () {
        _this.isRecordingStarted = false;
        _this.isStoppedRecording = true;

        _this.bdisabled = true;
        _this.adisabled = false;
        const blob = _this.recorder.getBlob();
        document.getElementById("my-video").src = URL.createObjectURL(blob);
        document.getElementById("my-video").parentNode.style.display = "block";

        //视频流下载保存到本地

        var url = window.URL.createObjectURL(blob);
        var a = document.createElement("a");
        a.href = url;
        a.style.display = "none";
        a.download = _this.currentRow.changeShiftsCode + ".mp4";
        _this.uploadVideo(blob, a.download);
 
      });
    },
	
//blob文件流转MP4上传
 uploadVideo(blob, fileName) {
      this.currentVideo=process.env.VUE_APP_BASE_API_URL+"/profile/upload/"+fileName
      //
      let param = new FormData(); //创建form对象
      let file = new File([blob], fileName, {
        type: "video/mp4",
        lastModified: Date.now(),
      });
      param.append("file", file);

      axios({
        headers: { Authorization: "Bearer " + getToken() },
        responseType: "blob",
        method: "post", // post 方式
        url: process.env.VUE_APP_BASE_API + "/common/upload", // 接口地址
        data: param, // 参数
      })
        .then((data) => {
          // 成功
        })
        .catch((err) => {
          // 异常
        });
    },
