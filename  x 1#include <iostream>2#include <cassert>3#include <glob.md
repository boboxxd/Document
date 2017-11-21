```
#include <iostream>
#include <cassert>
#include <glob.h>

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/opencv.hpp>

extern "C" {
#include <libavcodec/avcodec.h>
#include <libswscale/swscale.h>
#include <libavformat/avformat.h>
#include <libavutil/imgutils.h>
}

/**
 * Pixel formats and codecs
 */
static const AVPixelFormat sourcePixelFormat = AV_PIX_FMT_BGR24;
static const AVPixelFormat destPixelFormat = AV_PIX_FMT_YUV420P;
static const AVCodecID destCodec = AV_CODEC_ID_H264;


int main(int argc, char** argv)
{

    std::string input("/Users/xuxudong/Temp/1.rmvb"), output("test.h264");
    uint32_t framesToEncode=30000;
    uint8_t endcode[] = { 0, 0, 1, 0xb7 };

    /**
     * Create cv::Mat
     */
    cv::VideoCapture videoCapturer(input);
    if (videoCapturer.isOpened() == false) {
        std::cerr << "Cannot open video at '" << input << "'" << std::endl;
        exit(1);
    }

    /**
     * Get some information of the video and print them
     */
    double totalFrameCount = videoCapturer.get(CV_CAP_PROP_FRAME_COUNT);
    uint width = videoCapturer.get(CV_CAP_PROP_FRAME_WIDTH),
            height = videoCapturer.get(CV_CAP_PROP_FRAME_HEIGHT);

    std::cout << input << "[ Width:" << width << ", Height=" << height
              << ", FPS: " << videoCapturer.get(CV_CAP_PROP_FPS)
              << ", FrameCount: " << totalFrameCount << " ]" << std::endl;

    /**
     * Be sure we're not asking more frames than there is
     */
    if (framesToEncode > totalFrameCount) {
        std::cerr << "You asked for " << framesToEncode << " but there are only " << totalFrameCount
                  << " frames, will encode as many as there is" << std::endl;
        framesToEncode = totalFrameCount;
    }

    /**
     * Create an encoder and open it
     */
    avcodec_register_all();
    AVCodec *h264encoder = avcodec_find_encoder(destCodec);
    AVCodecContext *h264encoderContext = avcodec_alloc_context3(h264encoder);
    h264encoderContext->pix_fmt = destPixelFormat;
    h264encoderContext->width = width;
    h264encoderContext->height = height;
    h264encoderContext->bit_rate = 400000;
    h264encoderContext->time_base= (AVRational){1,25};


    if (avcodec_open2(h264encoderContext, h264encoder, NULL) < 0) {
        std::cerr << "Cannot open codec, exiting.." << std::endl;
        exit(1);
    }

    /**
     * Create a stream
     */
    AVFormatContext *cv2avFormatContext = avformat_alloc_context();
    AVStream *h264outputstream = avformat_new_stream(cv2avFormatContext, h264encoder);



    int got_frame;

    FILE* videoOutFile = fopen(output.c_str(), "wb");
    if (videoOutFile == 0) {
        std::cerr << "Cannot open output video file at '" << output << "'" << std::endl;
        exit(1);
    }

    /**
     * Prepare the conversion context
     */
    SwsContext *bgr2yuvcontext = sws_getContext(width, height, sourcePixelFormat,
                                                width, height, destPixelFormat,
                                                SWS_BICUBIC, NULL, NULL, NULL);




//-------------------------------------------------进入循环--------------------------------------------------------------
//----------------------------------------------------------------------------------------------------------------------
    /**
     * Convert and encode frames
     */
    for (uint i=0; i < framesToEncode; i++) {
        /**
         * Get frame from OpenCV
         */
        cv::Mat cvFrame;
        AVFrame *sourceAvFrame = av_frame_alloc();
        AVFrame *destAvFrame = av_frame_alloc();
        assert(videoCapturer.read(cvFrame) == true);


//---------------------------------------
   /**
    * Allocate source frame, i.e. input to sws_scale()
    */
        int re=av_image_alloc(sourceAvFrame->data, sourceAvFrame->linesize, width, height, sourcePixelFormat, 1);
        if(re<0)
        {
            fprintf(stderr,"err: %s",av_err2str(re));
            return -1;
        }
//---------------------------------------
        /**
         * Copy image data into AVFrame from cv::Mat
         */
        for (uint32_t h = 0; h < height; h++)
        {
            memcpy(&(sourceAvFrame->data[0][h*sourceAvFrame->linesize[0]]), &(cvFrame.data[h*cvFrame.step]), width*3);
    }
        /**
         * Allocate destination frame, i.e. output from sws_scale()
         */
        re=av_image_alloc(destAvFrame->data, destAvFrame->linesize, width, height, destPixelFormat, 1);
        if(re<0)
        {
            fprintf(stderr,"err: %s",av_err2str(re));
            return -1;
        }

        sws_scale(bgr2yuvcontext, sourceAvFrame->data, sourceAvFrame->linesize,
                  0, height, destAvFrame->data, destAvFrame->linesize);
//
//
        /**
         * Prepare an AVPacket and set buffer to NULL so that it'll be allocated by FFmpeg
         */
        AVPacket avEncodedPacket;
        av_init_packet(&avEncodedPacket);
        avEncodedPacket.data = NULL;
        avEncodedPacket.size = 0;

        destAvFrame->pts = i;
        avcodec_encode_video2(h264encoderContext, &avEncodedPacket, destAvFrame, &got_frame);

        if (got_frame) {
            std::cerr << "Encoded a frame of size " << avEncodedPacket.size << ", writing it.." << std::endl;

            if (fwrite(avEncodedPacket.data, 1, avEncodedPacket.size, videoOutFile) < (unsigned)avEncodedPacket.size)
                std::cerr << "Could not write all " << avEncodedPacket.size << " bytes, but will continue.." << std::endl;

            fflush(videoOutFile);
        }

        /**
         * Per-frame cleanup
         */
        av_packet_free_side_data(&avEncodedPacket);
        av_free_packet(&avEncodedPacket);
        av_freep(sourceAvFrame->data);
        av_frame_free(&sourceAvFrame);
        av_freep(destAvFrame->data);
        av_frame_free(&destAvFrame);
    }
    fwrite(endcode, 1, sizeof(endcode), videoOutFile);
    fclose(videoOutFile);

    /**
     * Final cleanup
     */
    sws_freeContext(bgr2yuvcontext);
    avformat_free_context(cv2avFormatContext);
    avcodec_close(h264encoderContext);
    avcodec_free_context(&h264encoderContext);
}
```













```
#include <libavutil/timestamp.h>
#include <libavformat/avformat.h>

static void log_packet(const AVFormatContext *fmt_ctx, const AVPacket *pkt, const char *tag)
{
    AVRational *time_base = &fmt_ctx->streams[pkt->stream_index]->time_base;

    printf("%s: pts:%s pts_time:%s dts:%s dts_time:%s duration:%s duration_time:%s stream_index:%d\n",
           tag,
           av_ts2str(pkt->pts), av_ts2timestr(pkt->pts, time_base),
           av_ts2str(pkt->dts), av_ts2timestr(pkt->dts, time_base),
           av_ts2str(pkt->duration), av_ts2timestr(pkt->duration, time_base),
           pkt->stream_index);
}

int main(int argc, char **argv)
{
    AVOutputFormat *ofmt = NULL;
    AVFormatContext *ifmt_ctx = NULL, *ofmt_ctx = NULL;
    AVPacket pkt;
    const char *in_filename, *out_filename;
    int ret, i;
    int stream_index = 0;
    int *stream_mapping = NULL;
    int stream_mapping_size = 0;

    if (argc < 3) {
        printf("usage: %s input output\n"
               "API example program to remux a media file with libavformat and libavcodec.\n"
               "The output format is guessed according to the file extension.\n"
               "\n", argv[0]);
        return 1;
    }

    in_filename  = argv[1];
    out_filename = argv[2];

    av_register_all();

    if ((ret = avformat_open_input(&ifmt_ctx, in_filename, 0, 0)) < 0) {
        fprintf(stderr, "Could not open input file '%s'", in_filename);
        goto end;
    }

    if ((ret = avformat_find_stream_info(ifmt_ctx, 0)) < 0) {
        fprintf(stderr, "Failed to retrieve input stream information");
        goto end;
    }

    av_dump_format(ifmt_ctx, 0, in_filename, 0);

    avformat_alloc_output_context2(&ofmt_ctx, NULL, NULL, out_filename);
    if (!ofmt_ctx) {
        fprintf(stderr, "Could not create output context\n");
        ret = AVERROR_UNKNOWN;
        goto end;
    }

    stream_mapping_size = ifmt_ctx->nb_streams;
    stream_mapping = av_mallocz_array(stream_mapping_size, sizeof(*stream_mapping));
    if (!stream_mapping) {
        ret = AVERROR(ENOMEM);
        goto end;
    }

    ofmt = ofmt_ctx->oformat;

    for (i = 0; i < ifmt_ctx->nb_streams; i++) {
        AVStream *out_stream;
        AVStream *in_stream = ifmt_ctx->streams[i];
        AVCodecParameters *in_codecpar = in_stream->codecpar;

        if (in_codecpar->codec_type != AVMEDIA_TYPE_AUDIO &&
            in_codecpar->codec_type != AVMEDIA_TYPE_VIDEO &&
            in_codecpar->codec_type != AVMEDIA_TYPE_SUBTITLE) {
            stream_mapping[i] = -1;
            continue;
        }

        stream_mapping[i] = stream_index++;

        out_stream = avformat_new_stream(ofmt_ctx, NULL);
        if (!out_stream) {
            fprintf(stderr, "Failed allocating output stream\n");
            ret = AVERROR_UNKNOWN;
            goto end;
        }

        ret = avcodec_parameters_copy(out_stream->codecpar, in_codecpar);
        if (ret < 0) {
            fprintf(stderr, "Failed to copy codec parameters\n");
            goto end;
        }
        out_stream->codecpar->codec_tag = 0;
    }
    av_dump_format(ofmt_ctx, 0, out_filename, 1);

    if (!(ofmt->flags & AVFMT_NOFILE)) {
        ret = avio_open(&ofmt_ctx->pb, out_filename, AVIO_FLAG_WRITE);
        if (ret < 0) {
            fprintf(stderr, "Could not open output file '%s'", out_filename);
            goto end;
        }
    }

    ret = avformat_write_header(ofmt_ctx, NULL);
    if (ret < 0) {
        fprintf(stderr, "Error occurred when opening output file\n");
        goto end;
    }

    while (1) {
        AVStream *in_stream, *out_stream;

        ret = av_read_frame(ifmt_ctx, &pkt);
        if (ret < 0)
            break;

        in_stream  = ifmt_ctx->streams[pkt.stream_index];
        if (pkt.stream_index >= stream_mapping_size ||
            stream_mapping[pkt.stream_index] < 0) {
            av_packet_unref(&pkt);
            continue;
        }

        pkt.stream_index = stream_mapping[pkt.stream_index];
        out_stream = ofmt_ctx->streams[pkt.stream_index];
        log_packet(ifmt_ctx, &pkt, "in");

        /* copy packet */
        pkt.pts = av_rescale_q_rnd(pkt.pts, in_stream->time_base, out_stream->time_base, AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX);
        pkt.dts = av_rescale_q_rnd(pkt.dts, in_stream->time_base, out_stream->time_base, AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX);
        pkt.duration = av_rescale_q(pkt.duration, in_stream->time_base, out_stream->time_base);
        pkt.pos = -1;
        log_packet(ofmt_ctx, &pkt, "out");

        ret = av_interleaved_write_frame(ofmt_ctx, &pkt);
        if (ret < 0) {
            fprintf(stderr, "Error muxing packet\n");
            break;
        }
        av_packet_unref(&pkt);
    }

    av_write_trailer(ofmt_ctx);
end:

    avformat_close_input(&ifmt_ctx);

    /* close output */
    if (ofmt_ctx && !(ofmt->flags & AVFMT_NOFILE))
        avio_closep(&ofmt_ctx->pb);
    avformat_free_context(ofmt_ctx);

    av_freep(&stream_mapping);

    if (ret < 0 && ret != AVERROR_EOF) {
        fprintf(stderr, "Error occurred: %s\n", av_err2str(ret));
        return 1;
    }

    return 0;
}
```

