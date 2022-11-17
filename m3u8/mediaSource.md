
最近看到网上大多数的video标签的src并不是一个mp4的文件，而是一个blob地址。
###1.这个blob地址是什么生成的
[MediaSource](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaSource/MediaSource)
```
var video = document.querySelector('video');

var assetURL = 'frag_bunny.mp4';
// Need to be specific for Blink regarding codecs
// ./mp4info frag_bunny.mp4 | grep Codec
var mimeCodec = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';

if ('MediaSource' in window && MediaSource.isTypeSupported(mimeCodec)) {
  var mediaSource = new MediaSource;
  //console.log(mediaSource.readyState); // closed
  video.src = URL.createObjectURL(mediaSource);
  mediaSource.addEventListener('sourceopen', sourceOpen);
} else {
  console.error('Unsupported MIME type or codec: ', mimeCodec);
}

function sourceOpen (_) {
    //console.log(this.readyState); // open
    var mediaSource = this;
    var sourceBuffer = mediaSource.addSourceBuffer(mimeCodec);
    fetchAB(assetURL, function (buf) {
        sourceBuffer.addEventListener('updateend', function (_) {
        mediaSource.endOfStream();
        video.play();
        //console.log(mediaSource.readyState); // ended
        });
        sourceBuffer.appendBuffer(buf);
    });
};
function fetchAB (url, cb) {
    console.log(url);
    var xhr = new XMLHttpRequest;
    xhr.open('get', url);
    xhr.responseType = 'arraybuffer';
    xhr.onload = function () {
        cb(xhr.response);
    };
    xhr.send();
};

```
通过上面例子，我们可以看出视频的blob地址是MediaSource的实例，通过addSourceBuffer/appendBuffer可以添加视频内容。不缺分是什么格式的视频。
###2.采用blob地址的优势：
1）对视频资源的地址做了一层”加密“处理，不支持暴露资源文件，不会轻易被爬虫获取，不会被轻易下载。
2）可以通过addSourceBuffer/appendBuffer可以对视频做下一步处理，比如添加广告。

注意：blob:URL是session级别的对象，当你刷新、打开新的tab时，就会生成新的blob对象地址，原有的blob:URL地址就失效了