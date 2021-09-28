
---
Title: raw video data
Description:
Platform: Android
UpdatedAt: Tue Apr 28 2020 04:48:08 GMT+0800 (CST)
---
# Original video data
## Function description

In the process of audio and video transmission, we can pre-process and post-process the collected audio and video data to obtain the desired playback effect.

For scenarios where there is a need to process audio and video data on your own, Agora provides the original data function. You can modify the captured voice signal or video frame before sending the data to the encoder, or you can modify the received voice signal or video frame after sending the data to the decoder.

Native SDK realizes the function of capturing and modifying the original video data by providing` IVideoFrameObserver` class.

## Realization method

Before using the raw data function, please make sure that you have completed the basic real-time audio and video functions in the project. For more information, please see [one-to-one call ](../../cn/Interactive%20Broadcast/start_call_android.md)or [ILVB](../../cn/Interactive%20Broadcast/start_live_android.md).

Refer to the following steps to implement the original video data function in your project:

1. Before joining the channel, call the `registerVideoFrameObserver` method to` register` the video observer, and implement an `IVideoFrameObserver` class in this method.
2. After successful registration, SDK will send the original video data obtained through `onCaptureVideoFrame`, `onPreEncodeVideoFrame` or `onRenderVideoFrame` callback when each video frame is captured.
3. After getting the video data, the user processes the video data on his own according to the needs of the scene. The processed video data is then sent to SDK through the above callback.

### API call timing

The following figure shows the timing of API calls using the original video data:

![](https://web-cdn.agora.io/docs-files/1577090428042)


### Sample code

You can compare the API timing diagram and refer to the following sample code snippet to implement the original video data function in the project:

```c++
# include < jni.h >
# include < android/log.h >
# include < cstring >

# include "agora/IAgoraRtcEngine.h"
# include "agora/IAgoraMediaEngine.h"

# include "video_preprocessing_plugin_jni.h"

Class AgoraVideoFrameObserver: public agora::media::IVideoFrameObserver
{
Public:
    / / obtain the video frames captured by the local camera
    Virtual bool onCaptureVideoFrame (VideoFrame& videoFrame) override
    {
        Int width = videoFrame.width
        Int height = videoFrame.height

        Memset (videoFrame.uBuffer, 128, videoFrame.uStride * height / 2)
        Memset (videoFrame.vBuffer, 128, videoFrame.vStride * height / 2)

        Return true
    }

    / / get the video frame sent by the remote user
    Virtual bool onRenderVideoFrame (unsigned int uid, VideoFrame& videoFrame) override
    {
        Return true
    }
	/ / get the video frame before local video coding
    Virtual bool onPreEncodeVideoFrame (VideoFrame& videoFrame) override
    {
        Return true
    }
}

Class IVideoFrameObserver
{
     Public:
         Enum VIDEO_FRAME_TYPE {
         FRAME_TYPE_YUV420 = 0, / / Video frame format is YUV420
         }
     Struct VideoFrame {
         VIDEO_FRAME_TYPE type
         Width of int width; / / video frame
         Height of int height; / / video frame
         Line span of Y buffer in int yStride; / / YUV data
         The row span of the U buffer in int uStride; / / YUV data
         Line span of V buffer in int vStride; / / YUV data
         Pointer to the Y buffer in void* yBuffer; / / YUV data
         Pointer to the U buffer in void* uBuffer; / / YUV data
         Pointer to the V buffer in void* vBuffer; / / YUV data
         Int rotation; / / rotation information of this frame, which can be set to 0,90,180,270
         Int64_t renderTimeMs; / / timestamp of the frame
         }
     Public:
         Virtual bool onCaptureVideoFrame (VideoFrame& videoFrame) = 0
         Virtual bool onRenderVideoFrame (unsigned int uid, VideoFrame& videoFrame) = 0
		 Virtual bool onPreEncodeVideoFrame (VideoFrame& videoFrame) {return true;}
}
```

At the same time, we provide an open source[ Video encrypt](https://github.com/AgoraIO/Advanced-Video/tree/master/Android/sample-video-encrypt) sample project at GitHub. You can download it or refer to the code in the [`IAgoraMediaEngine.h`](https://github.com/AgoraIO/Advanced-Video/blob/master/Android/sample-video-encrypt/src/main/cpp/include/agora/IAgoraMediaEngine.h) file.

### API referenc

- [`RegisterVideoFrameObserver`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_media_engine.html#a5eee4dfd1fd46e4a865feba163f3c5de)
- [`OnCaptureVideoFrame`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_video_frame_observer.html#a915c673aec879dcc2b08246bb2fcf49a)
- [`OnRenderVideoFrame`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_video_frame_observer.html#a966ed2459b6887c52112af638bc27c14)
- [`OnPreEncodeVideoFrame`](https://docs.agora.io/cn/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1media_1_1_i_video_frame_observer.html#a2be41cdde19fcc0f365d4eb14a963e1c)

## Matters needing attention in development

The raw data interface used in this paper is C++ interface. If you are developing on the Android platform, please refer to the following steps to register the video data observer using the JNI of the SDK library and the plug-in manager.

1. Before joining the channel, create a shared library project. The project must be prefixed with `libapm-` and suffixed with `so`, such as `libapm-encryption.so`.
2. After successful creation, RtcEngine automatically loads the `libapm-encryption.so` file as a statistical catalog plug-in.
3. After SDK terminates the engine, call the `unloadAgoraRtcEnginePlugin` API, and RtcEngine will automatically uninstall the statistics directory plug-in.

```java
Static AgoraVideoFrameObserver s_videoFrameObserver
Static agora::rtc::IRtcEngine* rtcEngine = NULL


# ifdef _ _ cplusplus
Extern "C" {
# endif

Int _ attribute__ ((visibility ("default") loadAgoraRtcEnginePlugin (agora::rtc::IRtcEngine* engine)
{
    _ _ android_log_print (ANDROID_LOG_ERROR, "plugin", "plugin loadAgoraRtcEnginePlugin")
    RtcEngine = engine
    Return 0
}

Void _ attribute__ ((visibility ("default") unloadAgoraRtcEnginePlugin (agora::rtc::IRtcEngine* engine)
{
    _ _ android_log_print (ANDROID_LOG_ERROR, "plugin", "plugin unloadAgoraRtcEnginePlugin")
    RtcEngine = NULL
}

JNIEXPORT void JNICALL Java_io_agora_propeller_preprocessing_VideoPreProcessing_enablePreProcessing
  (JNIEnv * env, jobject obj, jboolean enable)
{
    If (! rtcEngine)
        Return
    Agora::util::AutoPtr < agora::media::IMediaEngine > mediaEngine
    MediaEngine.queryInterface (rtcEngine, agora::AGORA_IID_MEDIA_ENGINE)
    If (mediaEngine) {
        If (enable) {
            MediaEngine- > registerVideoFrameObserver (& s_videoFrameObserver)
        } else {
            MediaEngine- > registerVideoFrameObserver (NULL)
        }
    }
}

# ifdef _ _ cplusplus
}
# endif
```

## Related documents

If you also want to implement the original audio data function in the project, please refer to [the original audio data](../../cn/Interactive%20Broadcast/raw_data_audio_android.md).
