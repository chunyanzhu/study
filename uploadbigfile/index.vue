<template>
    <el-upload
        class="avatar-uploader"
        :action="action"
        :headers="headers"
        :show-file-list="false"
        :on-success="handleAvatarSuccess"
        :before-upload="beforeAvatarUpload"
        :accept="accept"
        :on-change="onChange"
        :on-progress="onProgress"
    >
    </el-upload>
</template>
<script setup>
// 上传大文件由于各种原因网络突然中断，然后整个文件都需要从头继续上传。
// 对文件进行切割成小文件，逐个上传或并发上传，以文件的md5码作为文件的唯一标识，
// 这样即使上传到一半中断了，等到下次再上传的时候我们通过对比文件的md5来确认是否相同文件，确认后就可以在上次上传的基础上继续上传了


const onChange = (file)=>{
  console.log('file,', file)
  uploadVideo(file.raw, 20).then(res=>{
    console.log('总体完成')
    // 组合切片
  }).catch(err=>{
    console.log('总体失败')
  })
}
const uploadVideo = async(file, byte = 2)=>{
  var duration = await getVideoDuration(file)
  let chunkSize = byte * 1024 * 1024
  let chunkArr = []
  if(file.size < chunkSize){
    chunkArr.push(file.slice(0))
  }else{
    var start = 0, end = 0
    while(true){
      end+=chunkSize
      var blob = file.slice(start, end)
      start+= chunkSize
      if(!blob.size){
        break;
      }
      chunkArr.push(blob)
    }
  }
  console.log('chunkArr', chunkArr)


  return new Promise((ok, no)=>{
    // await 
    // requet('https:',{
    //       "video_number",//总共分为几片
    //       "size",//文件总大小
    //       "video_len",//视频长度
    //       "suffix":file.type,//文件类型
    //       "md5":d
    //     }).then((chunksArr)=>{

    // })


    const allPromie = []
    chunkArr.forEach((item, index)=>{
      let p = new Promise((resolve, reject)=>{
        var data = new FormData()
        data.append('video', item)
        data.append('name', '我哦哦哦哦哦'),
        data.append('blob_num', index+1)
        data.append('total_blob_num', chunkArr.length)
        request({
          url: `${process.env.VUE_APP_BASE_API}/common/upload`,
          method: 'POST',
          data
        })
      })
      allPromie.push(p)
    })

    Promise.all(all).then(res=>{
      ok()
    }).catch(err=>{
      no(err)
    })
  })
  
}
const getVideoDuration = (file)=>{
  return new Promise((resolve, reject)=>{
    // @todo file里面可以读到duration吗 loadedmetadata是什么事件  时间久吗
    var url = URL.createObjectURL(file)
    var audioElement = new Audio(url)
    console.time('loadedmetadata')
    console.time('loadeddata')
    console.time('canplay')
    audioElement.addEventListener('loadedmetadata', function(_event){
      console.log('audioElement loadedmetadata ', audioElement.duration)
      console.timeEnd('loadedmetadata')
      var hour = parseInt((audioElement.duration)/3600)
      if (hour<10) { hour = "0" + hour; }
          var minute = parseInt((audioElement.duration % 3600) / 60);
          if (minute<10) {  minute = "0" + minute; }
          var second = Math.ceil(audioElement.duration % 60);
          if (second<10) { second = "0" + second; }
          resolve(hour + ":" + minute + ":" + second)
    })
    audioElement.addEventListener('loadeddata', function(){
      console.timeEnd('loadeddata')
    })
    audioElement.addEventListener('canplay',function(){
      console.timeEnd('canplay')
    })
  })
}
</script>
