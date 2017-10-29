```
//cv::mat to AVFrame
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

void MattoAVFrame(AVFrame *frame,cv::Mat mat,int width,int height) {
    for (uint32_t h = 0; h < height; h++) {
        memcpy(&(frame->data[0][h * frame->linesize[0]]), &(mat.data[h * mat.step]), width * 3);
    }
}


int main(int argc, char** argv)
{

    std::string input("rtsp://admin:ad53937301@49.91.240.8:554/h264/ch1/main/av_stream"), output("test2.flv");
    uint32_t framesToEncode=3000;
    uint8_t endcode[] = { 0, 0, 1, 0xb7 };

//-----------------------------------------------取图初始化-------------------------------------------------------------------
    cv::VideoCapture videoCapturer(input);
    if (videoCapturer.isOpened() == false) {
        std::cerr << "Cannot open video at '" << input << "'" << std::endl;
        exit(1);
    }

    double totalFrameCount = videoCapturer.get(CV_CAP_PROP_FRAME_COUNT);
    uint width = videoCapturer.get(CV_CAP_PROP_FRAME_WIDTH);
    uint height = videoCapturer.get(CV_CAP_PROP_FRAME_HEIGHT);

    std::cout << input << "[ Width:" << width << ", Height=" << height
              << ", FPS: " << videoCapturer.get(CV_CAP_PROP_FPS)
              << ", FrameCount: " << totalFrameCount << " ]" << std::endl;

    if (framesToEncode > totalFrameCount) {
        std::cerr << "You asked for " << framesToEncode << " but there are only " << totalFrameCount
                  << " frames, will encode as many as there is" << std::endl;
        framesToEncode = totalFrameCount;
    }
//----------------------------------------------------------------------------------------------------------------------

     //Create an encoder and open it
    avcodec_register_all();

    AVCodec *h264encoder = avcodec_find_encoder(destCodec);
    AVCodecContext *h264encoderContext = avcodec_alloc_context3(h264encoder);
    h264encoderContext->pix_fmt = destPixelFormat;
    h264encoderContext->width = width;
    h264encoderContext->height = height;
    h264encoderContext->bit_rate = 400000;
    h264encoderContext->time_base= (AVRational){1,1};


    if (avcodec_open2(h264encoderContext, h264encoder, NULL) < 0) {
        std::cerr << "Cannot open codec, exiting.." << std::endl;
        exit(1);
    }
//----------------------------------------------------------------------------------------------------------------------
//创建h264流

    AVFormatContext *cv2avFormatContext = avformat_alloc_context();
    AVStream *h264outputstream = avformat_new_stream(cv2avFormatContext, h264encoder);

    int got_frame;


    FILE* videoOutFile = fopen(output.c_str(), "wb");
    if (videoOutFile == 0) {
        std::cerr << "Cannot open output video file at '" << output << "'" << std::endl;
        exit(1);
    }

    SwsContext *bgr2yuvcontext = sws_getContext(width, height, sourcePixelFormat,
                                                width, height, destPixelFormat,
                                                SWS_BICUBIC, NULL, NULL, NULL);

//----------------------------------------------------------------------------------------------------------------------
//创建flv流
    AVIOContext *outctx=0;

    avio_open(&outctx,output.c_str(),0);
    //avformat_write_header(cv2avFormatContext, 0);



//----------------------------------------------------------------------------------------------------------------------


//-------------------------------------------------进入循环--------------------------------------------------------------
//----------------------------------------------------------------------------------------------------------------------
    /**
     * Convert and encode frames
     */
    for (uint i=0; i < framesToEncode; i++) {

        cv::Mat cvFrame;
        AVFrame *sourceAvFrame = av_frame_alloc();
        AVFrame *destAvFrame = av_frame_alloc();
        assert(videoCapturer.read(cvFrame) == true);
//----------------------------------------------------------------------------------------------------------------------
        //分配frame空间
        int re=av_image_alloc(sourceAvFrame->data, sourceAvFrame->linesize, width, height, sourcePixelFormat, 1);
        if(re<0)
        {
            fprintf(stderr,"err: %s",av_err2str(re));
            return -1;
        }

//----------------------------------------------------------------------------------------------------------------------
        // Copy image data into AVFrame from cv::Mat
        MattoAVFrame(sourceAvFrame,cvFrame,width,height);

        //转换bgr2yuv
        re=av_image_alloc(destAvFrame->data, destAvFrame->linesize, width, height, destPixelFormat, 1);
        if(re<0)
        {
            fprintf(stderr,"err: %s",av_err2str(re));
            return -1;
        }

        sws_scale(bgr2yuvcontext, sourceAvFrame->data, sourceAvFrame->linesize,
                  0, height, destAvFrame->data, destAvFrame->linesize);
//----取图end，至此图片在destAvFrame中

//--------------------------------------------解码-----------------------------------------------------------------------
        AVPacket avEncodedPacket;
        av_init_packet(&avEncodedPacket);
        avEncodedPacket.data = NULL;
        avEncodedPacket.size = 0;
        destAvFrame->pts = i;

        avcodec_encode_video2(h264encoderContext, &avEncodedPacket, destAvFrame, &got_frame);

        //printf("streamindex=%d",avEncodedPacket.stream_index);


//----解码成功
//----------------------------------------------------------------------------------------------------------------------

        if (got_frame) {
            std::cerr << "Encoded a frame of size " << avEncodedPacket.size << ", writing it.." << std::endl;
            if (fwrite(avEncodedPacket.data, 1, avEncodedPacket.size, videoOutFile) < (unsigned)avEncodedPacket.size)
                std::cerr << "Could not write all " << avEncodedPacket.size << " bytes, but will continue.." << std::endl;
            fflush(videoOutFile);
            printf("streamindex=%d",avEncodedPacket.stream_index);

        }


//cleanup
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