---
title: ios ijkplayer改造添加录屏、RTSP、HTTPS支持
date:  2019-04-02 10:29:00
tags:
  - 工具
categories: 技术分享
---

经过几天的努力现在可以总结下这几天对ijkplayer的改造。
1、在GitHub上clone ijkplayer源码，切换分支到0.8.8
```
$cd ~/file/ijkfile //目录可能不存在 ，需要自己创建
$git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
//进入ijkplayer-ios
$cd ijkplayer-ios
//切换分支
$git checkout -B latest k0.8.8
```
2、添加RTSP和HTTPS支持（以module-lite.sh为例）
```
开启支持RTSP和HTTPS
开启支持RTSP,默认不支持RTSP,需要修改module-lite.sh内容,新增对应的协议,module-lite.sh是在config目录下
目录：~/config/module-lite.sh 将这一行：
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-protocol=rtp"
修改为：
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtp"

export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtsp"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=tcp"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-decoder=mjpeg"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=mjpeg"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-openssl"
```
3、编译文件设置
注：可根据具体情况加载模块：
偏好更多的解码器/视频格式支持, 则链接module-default.sh
偏好打包出来的库体积更小 (默认格式支持), 则链接module-lite.sh
偏好打包出来的库体积更小 (包含HEVC支持), 则链接module-lite-hevc.sh
```
//进入ijkplayer/config 目录
$ cd config
//移除module.sh文件
$ rm module.sh
//替换模块
$ ln -s module-lite.sh module.sh
```
4、修改

```
//ff_play.h
int ffp_start_recording_l(FFPlayer *ffp,const char *file_name);
int ffp_record_isfinished_l(FFPlayer *ffp);
int ffp_stop_recording_l(FFPlayer *ffp);
```

```
//ff_ffplay_def.h
...
    int render_wait_start;
    AVFormatContext *m_ofmt_ctx;        // 用于输出的AVFormatContext结构体
    AVOutputFormat *m_ofmt;
    pthread_mutex_t record_mutex;       // 锁
    int is_record;                      // 是否在录制
    int record_error;
    
    int is_first;                       // 第一帧数据
    int64_t start_pts;                  // 开始录制时pts
    int64_t start_dts;                  // 开始录制时dts
...
} FFPlayer;

```

```
//ff_play.c
//插入录制方法
int ffp_start_recording_l(FFPlayer *ffp,const char *file_name)
{
    assert(ffp);
    VideoState *is = ffp->is;
    
    ffp->m_ofmt_ctx = NULL;
    ffp->m_ofmt = NULL;
    ffp->is_record = 0;
    ffp->record_error = 0;
    
    if (!file_name || !strlen(file_name)) { // 没有路径
        av_log(ffp, AV_LOG_ERROR, "filename is invalid");
        goto end;
    }
    
    if (!is || !is->ic|| is->paused || is->abort_request) { // 没有上下文，或者上下文已经停止
        av_log(ffp, AV_LOG_ERROR, "is,is->ic,is->paused is invalid");
        goto end;
    }
    
    if (ffp->is_record) { // 已经在录制
        av_log(ffp, AV_LOG_ERROR, "recording has started");
        goto end;
    }
    
    // 初始化一个用于输出的AVFormatContext结构体
    avformat_alloc_output_context2(&ffp->m_ofmt_ctx, NULL, NULL, file_name);
    if (!ffp->m_ofmt_ctx) {
        av_log(ffp, AV_LOG_ERROR, "#ly# Could not create output context filename is %s\n", file_name);
        goto end;
    }
    ffp->m_ofmt = ffp->m_ofmt_ctx->oformat;
    
    for (int i = 0; i < is->ic->nb_streams; i++) {
        // 对照输入流创建输出流通道1
        AVStream *in_stream = is->ic->streams[i];
        AVStream *out_stream = avformat_new_stream(ffp->m_ofmt_ctx, in_stream->codec->codec);
        if (!out_stream) {
            av_log(ffp, AV_LOG_ERROR, "Failed allocating output stream\n");
            goto end;
        }
        
        // 将输入视频/音频的参数拷贝至输出视频/音频的AVCodecContext结构体1
        av_log(ffp, AV_LOG_DEBUG, "in_stream->codec；%p\n", in_stream->codec);
        if (avcodec_copy_context(out_stream->codec, in_stream->codec) < 0) {
            av_log(ffp, AV_LOG_ERROR, "Failed to copy context from input to output stream codec context\n");
            goto end;
        }
        
        out_stream->codec->codec_tag = 0;
        if (ffp->m_ofmt_ctx->oformat->flags & AVFMT_GLOBALHEADER) {
            out_stream->codec->flags |= CODEC_FLAG_GLOBAL_HEADER;
        }
    }
    
    av_dump_format(ffp->m_ofmt_ctx, 0, file_name, 1);
    
    // 打开输出文件
    if (!(ffp->m_ofmt->flags & AVFMT_NOFILE)) {
        if (avio_open(&ffp->m_ofmt_ctx->pb, file_name, AVIO_FLAG_WRITE) < 0) {
            av_log(ffp, AV_LOG_ERROR, "Could not open output file '%s'", file_name);
            goto end;
        }
    }
    
    // 写视频文件头
    if (avformat_write_header(ffp->m_ofmt_ctx, NULL) < 0) {
        av_log(ffp, AV_LOG_ERROR, "Error occurred when opening output file\n");
        goto end;
    }
    
    ffp->is_record = 1;
    ffp->record_error = 0;
    pthread_mutex_init(&ffp->record_mutex, NULL);
    
    return 0;
end:
    ffp->record_error = 1;
    return -1;
}

int ffp_record_isfinished_l(FFPlayer *ffp)
{
    return 0;
}

int ffp_record_file(FFPlayer *ffp, AVPacket *packet){
    assert(ffp);
    VideoState *is = ffp->is;
    int ret = 0;
    AVStream *in_stream;
    AVStream *out_stream;
    if (ffp->is_record) {
        if (packet == NULL) {
            ffp->record_error = 1;
            av_log(ffp, AV_LOG_ERROR, "packet == NULL");
            printf("ffp_record_file return null 1");
            return -1;
        }
        
        AVPacket *pkt = (AVPacket *)av_malloc(sizeof(AVPacket)); // 与看直播的 AVPacket分开，不然卡屏
        av_new_packet(pkt, 0);
        if (0 == av_packet_ref(pkt, packet)) {
            pthread_mutex_lock(&ffp->record_mutex);
            if (!ffp->is_first) { // 录制的第一帧，时间从0开始
                ffp->is_first = 1;
                pkt->pts = 0;
                pkt->dts = 0;
            } else { // 之后的每一帧都要减去，点击开始录制时的值，这样的时间才是正确的
                
                pkt->pts = llabs(pkt->pts - ffp->start_pts);
                pkt->dts = llabs(pkt->dts - ffp->start_dts);
                printf("AVMEDIA_TYPE_VIDEO pkt->pts == %lld pkt->dts == %lld\n",pkt->pts,pkt->dts);
            }
            
            in_stream  = is->ic->streams[pkt->stream_index];
            out_stream = ffp->m_ofmt_ctx->streams[pkt->stream_index];
            
            // 转换PTS/DTS
            pkt->pts = av_rescale_q_rnd(pkt->pts, in_stream->time_base, out_stream->time_base, (AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX));
            pkt->dts = av_rescale_q_rnd(pkt->dts, in_stream->time_base, out_stream->time_base, (AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX));
            pkt->duration = av_rescale_q(pkt->duration, in_stream->time_base, out_stream->time_base);
            pkt->pos = -1;
            
            // 写入一个AVPacket到输出文件
            if ((ret = av_interleaved_write_frame(ffp->m_ofmt_ctx, pkt)) < 0) {
                av_log(ffp, AV_LOG_ERROR, "Error muxing packet\n");
            }
            
            av_packet_unref(pkt);
            pthread_mutex_unlock(&ffp->record_mutex);
        } else {
            av_log(ffp, AV_LOG_ERROR, "av_packet_ref == NULL");
            printf("ffp_record_file return null 2");
        }
    }
    printf("ffp_record_file return null 3 == %d\n",ret);
    return ret;
}


int ffp_stop_recording_l(FFPlayer *ffp){
    assert(ffp);
    if (ffp->is_record) {
        ffp->is_record = 0;
        pthread_mutex_lock(&ffp->record_mutex);
        if (ffp->m_ofmt_ctx != NULL) {
            av_write_trailer(ffp->m_ofmt_ctx);
            if (ffp->m_ofmt_ctx && !(ffp->m_ofmt->flags & AVFMT_NOFILE)) {
                avio_close(ffp->m_ofmt_ctx->pb);
            }
            avformat_free_context(ffp->m_ofmt_ctx);
            ffp->m_ofmt_ctx = NULL;
            ffp->is_first = 0;
        }
        pthread_mutex_unlock(&ffp->record_mutex);
        pthread_mutex_destroy(&ffp->record_mutex);
        av_log(ffp, AV_LOG_DEBUG, "stopRecord ok\n");
    } else {
        av_log(ffp, AV_LOG_ERROR, "don't need stopRecord\n");
    }
    return 0;
}


//上屏解码处或缓冲解码处(decoder_decode_frame、read_thread)
//decoder_decode_frame
...
}while (d->queue->serial != d->pkt_serial);
        if (!ffp->is_first && pkt.pts == pkt.dts) { // 获取开始录制前dts等于pts最后的值
            ffp->start_pts = pkt.pts;
            ffp->start_dts = pkt.dts;
        }
      #pragma  mark - 录制插入
        if (ffp->is_record) { // 可以录制时，写入文件
            if (0 != ffp_record_file(ffp, &pkt)) {
                ffp->record_error = 1;
                ffp_stop_recording_l(ffp);
                printf("avcodec_send_packet stop\n");
            }
        }
if (pkt.data == flush_pkt.data) {
...

//read_thread
...
for(::){
 if (!ffp->is_first && pkt.pts == pkt.dts) { // 获取开始录制前dts等于pts最后的值
            ffp->start_pts = pkt.pts;
            ffp->start_dts = pkt.dts;
        }
        #pragma  mark - 录制插入
        if (ffp->is_record) { // 可以录制时，写入文件
            if (0 != ffp_record_file(ffp, &pkt)) {
                ffp->record_error = 1;
                ffp_stop_recording_l(ffp);
                printf("avcodec_send_packet stop\n");
            }
        }
...
}
...
```

```
//ijkplayer.h
int ijkmp_start_recording(IjkMediaPlayer *mp,const char *filePath);
int ijkmp_stop_recording(IjkMediaPlayer *mp);;
int ijkmp_isRecording(IjkMediaPlayer *mp);
```

```
//ijkplayer.c
static int ijkmp_start_recording_l(IjkMediaPlayer *mp,const char *filePath)
{
    //  av_log(mp->ffplayer,AV_LOG_WARING,"cjz ijkmp_start_recording_l filePath %s",filePath);
    return ffp_start_recording_l(mp->ffplayer,filePath);
}

int ijkmp_start_recording(IjkMediaPlayer *mp,const char *filePath)
{
    assert(mp);
    pthread_mutex_lock(&mp->mutex);
    av_log(mp->ffplayer,AV_LOG_WARNING,"cjz ijkmp_start_recording");
    int retval = ijkmp_start_recording_l(mp,filePath);
    printf("ijkmp_start_recording return == %d\n",retval);
    pthread_mutex_unlock(&mp->mutex);
    return retval;
}

static int ijkmp_stop_recording_l(IjkMediaPlayer *mp)
{
    return ffp_stop_recording_l(mp->ffplayer);
}

int ijkmp_stop_recording(IjkMediaPlayer *mp)
{
    assert(mp);
    pthread_mutex_lock(&mp->mutex);
    av_log(mp->ffplayer,AV_LOG_WARNING,"cjz ijkmp_stop_recording");
    int retval = ijkmp_stop_recording_l(mp);
    pthread_mutex_unlock(&mp->mutex);
    return retval;
}

static int ijkmp_isRecordFinished_l(IjkMediaPlayer *mp)
{
    return ffp_record_isfinished_l(mp->ffplayer);
}

int ijkmp_isRecordFinished(IjkMediaPlayer *mp)
{
    assert(mp);
    pthread_mutex_lock(&mp->mutex);
    av_log(mp->ffplayer,AV_LOG_WARNING,"cjz ijkmp_isRecordFinished ");
    int retval = ijkmp_isRecordFinished_l(mp);
    pthread_mutex_unlock(&mp->mutex);
    return retval;
}

int ijkmp_isRecording(IjkMediaPlayer *mp) {
    return mp->ffplayer->is_record;
}
```

```
//IJKMediaPlayback.h
- (void)stopRecord;
- (void)startRecordWithFileName:(NSString *)fileName;
```

```
//IJKFFMoviePlayerController.m
- (void)stopRecord {
    ijkmp_stop_recording(_mediaPlayer);
    NSLog(@"stop record");
}

- (void)startRecordWithFileName:(NSString *)fileName {
    // 视频存储的路径
    const char *path = [fileName cStringUsingEncoding:NSUTF8StringEncoding];
    ijkmp_start_recording (_mediaPlayer, path);
    
    NSLog(@"start record fileName %@",fileName);
}

- (BOOL)isRecording {
    return ijkmp_isRecording(_mediaPlayer);
}
```
3、编译ijkplayer
```
//编译iOS
$ ./init-ios.sh
$ ./init-ios-openssl.sh （不需要https请忽略openssl）

//编译ffmpg (我这里只编译arm64)
//可能执行compile-ffmpeg.sh arm64会出错，如果是架构请检查编译器是否支持等。
$ cd ios 
$ ./compile-ffmpeg.sh clean
$ ./compile-openssl.sh arm64 （不需要https请忽略openssl）
$ ./compile-ffmpeg.sh arm64
```
4、编译framework（稀松平常略过）
5、合并framework（如果只有arm64也不需要合并，只能真机使用）
```
准备合并:
1、打开终端, 先cd到Products目录下
2、然后执行:lipo -create 真机framework路径 模拟器framework路径 -output 合并的文件路径
例如：
lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
```

经过几天的踩坑，广泛参考网上既有资料，加入了我需要用到的业务场景可完美录屏（但存在录屏时关键帧缺失会从下一个关键帧开始正常播放，看命吧）。
