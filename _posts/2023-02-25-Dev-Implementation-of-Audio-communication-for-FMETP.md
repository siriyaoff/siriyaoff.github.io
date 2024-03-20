---
layout: single
title: "Implemention of Audio communication for FMETP"
toc: true
toc_label: "Index"
toc_icon: "columns"
categories:
  - Dev
tags:
  - Unity
  - C#
  - FMETP
  - Mediapipeline
  - AudioSource
  - Microphone
  - Event
  - Event handler
---

## Analysis of audio pipeline

- 결국 화면이 나올 때 음성이 같이 나와야 하기 때문에, 화면 처리를 담당하는 game object들에 audio 관련 스크립트들을 붙임
  - `FaceCamera` : bytedata로의 화면 인코딩을 담당하는 game object
  - `StreamVideoRelated-WebCameDecoder` : 수신한 bytedata를 decode하는 game object
- audio 관련 스크립트들은 명확하게 두 개만 사용함
  - `FMETP_STREAM/FMCore/Scripts/Mapper/MicEncoder.cs` : 마이크를 Audio Source로 사용해 인코딩을 구현함
  - `FMETP_STREAM/FMCore/Scripts/Mapper/AudioDecoder.cs` : 수신한 bytedata가 음성 데이터일 경우 decode

### Framing Protocol

- format은 `FMPCM16`을 사용함
  - `PMC16`은 metadata가 포함되지 않음
- `MicEncoder.cs`에서 audio packet을 생성하는 부분:

  ```csharp
  Buffer.BlockCopy(_meta_label, 0, SendByte, 0, 2);
  Buffer.BlockCopy(_meta_id, 0, SendByte, 2, 2);
  Buffer.BlockCopy(_meta_length, 0, SendByte, 4, 4);
  Buffer.BlockCopy(_meta_offset, 0, SendByte, 8, 4);
  SendByte[12] = (byte)(GZipMode ? 1 : 0);
  SendByte[13] = (byte)0;//not used, but just keep one empty byte for standard

  Buffer.BlockCopy(dataByteTemp, _offset, SendByte, 14, dataByteLength);
  AppendQueueSendByteFMPCM16.Enqueue(SendByte);
  ```

  - label : image 패킷인지 audio 패킷인지 구별
    - image : 2001
    - audio : 1001
  - id : data의 번호(chunksize에 맞춰서 오디오를 분할하는데, 이때의 넘버링)
  - length : 데이터의 길이
  - offset : 기존 오디오 데이터의 몇 번째부터 전송하는지
    - chunksize에 맞춰서 분할하기 때문에 시작점이 다를 수 있음

- 오디오 패킷 예시:
  ```
  label::
  209,7
  id::
  49,0
  length::
  8,8
  offset::
  0,0,0,0
  dataByte::
  -64,93,0,0,1,0,0,0,-6,-1,-7,-1,-7,-1,-8,-1,-8,-1,-10,-1,-11,-1,-15,-1,-18,-1,-22,-1,-26,-1,-29,-1,-33,-1,-36,-1,-40,-1,-42,-1,-46,-1,-48,-1,-49,-1,-50,-1,-51,-1,...
  ```
  - little endian이라서 바꿔서 계산해야 함  
    ![FMETP-audio-comm-1](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-audio-comm-1.png)
    - 7, 209 순서로 이진수로 변환하면 2001임

## Implementation

- `FaceCamera` object에 `MicEncoder.cs` 스크립트를 추가함  
  ![FMETP-audio-comm-2](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-audio-comm-2.png)
  - `Stream Game Sound` : 컴퓨터에서 출력되는 오디오를 같이 녹음할지 선택
- `WebCameDecoder` object에 `AudioDecoder.cs` 스크립트를 추가함  
  ![FMETP-audio-comm-3](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-audio-comm-3.png)
  - `AudioDecoder.cs`에서 재생할 Audio Source가 필요하기 때문에 `Audio Source`가 자동으로 추가됨
  - `Audio Source` properties
    - Mute : default가 true이기 때문에 확인해야 함
    - Volume : default가 0이기 때문에 확인해야 함
  - `AudioDecoder` properties
    - Playback - `Volume` : Playback이 자신의 오디오를 다시 들려줄 때의 볼륨일 줄 알았는데, 그냥 수신한 오디오의 볼륨을 조절하는 것임
- `StreamVideoRelated-SocketManager` object의 `OnReceivedByteDataEvent(Byte[])`에 오디오 이벤트 핸들러 `AudioDecoder.Action_ProcessFMPCM16Data`를 추가함  
  ![FMETP-audio-comm-4](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-audio-comm-4.png)
  - `GameViewDecoder.Action_ProcessImageData`, `AudioDecoder.Action_ProcessFMPCM16Data` 모두 패킷의 label을 확인하고 다르면 처리하지 않기 때문에 따로 label에 따른 처리를 해줄 필요가 없음
- `MicEncoder.cs`, `AudioDecoder.cs`의 samplingRate, StreamFPS를 동기화함
  - samplingRate : 48000
  - StreamFPS : 30
- `AudioDecoder.cs`의 코드 수정

  ```csharp
  private void CreateClip()
  {
      if (true || samplerate != SourceSampleRate || channel != SourceChannels) // true
      {
          samplerate = SourceSampleRate;
          channel = SourceChannels;

          if (Audio != null) Audio.Stop();
          if (audioClip != null) DestroyImmediate(audioClip);

          audioClip = AudioClip.Create("StreamingAudio", samplerate * SourceChannels, SourceChannels, samplerate, true, OnAudioRead, OnAudioSetPosition);
          Audio = GetComponent<AudioSource>();
          Audio.clip = audioClip;
          Audio.loop = true;
          Audio.Play();
          // Debug.Log("audio played***********");
      }
  }
  ```

  - 직관적으로 봤을 때 `samplerate`, `channel`을 source와 맞추고 오디오 스트리밍을 시작하는 것 같은데, 한 번만 실행됐을 때는 오디오 재생이 되지 않음
    - 첫 조건문에 `true`를 추가해서 계속 초기화되도록 설정함

## Restart audio pipeline when Mic device changed

### Issues

- Whenever Mic device changed, media pipeline couldn’t detect the new one, so audio packet wasn’t created
  - Furthermore, opening airpod’s case also affected to generating the audio packet such as if the default Mic is set to MacBook Pro Mic and the case is opened, then feeding stopped immediately.
  - The whole audio pipeline starts and ends in `MicEncoder.cs`
    - `CaptureMic()` is invoked only once when `StartAll()` executed, so I couldn’t find the relation of available Mic list and the audio source of `AddMicData()`

<img src="https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-audio-comm-5.png" width="50%" height="50%">

- `Encoded Size(byte)` fixed to 8 when the audiobyte wasn’t put
  - So whenever I tried to change the Mic device, it was fixed to 8
  - Objective is to maintain reading audio source even if the Mic changed so that we can change Mic freely.

### Troubleshooting

- Modifying the order of the coroutines or the control of the coroutines were no effect.
  - Coroutines::
    - `CaptureMic()` : Initialize the `AudioSource`, `Microphone`
    - `SenderCOR()` : Read audio clip from `AudioSource`, add it to the packet queue
  - What I tried::
    - coroutine 초기화말고 아예 오디오 클립만 바꿔주면 될지도
      - `AudioMic` Stop → `Microphone` End → `Microphone` Start → `AudioMic` Play
        - 이전 microphone이 끝났는지 확인하고 끝났을 때 변경
        - 새로운 마이크 감지는 안됨
        - Mic A, Mic B가 있을 때::
          - A → B : 안됨
          - A → B → A : 안됨
          - A → B → A → B : 안됨
          - A → B → A → B → A : 됨
    - coroutine 손안대고 `AudioMic`, `Microphone`만 손댐 : 안됨
      - A → M → M → A : 안됨
      - M → M : 안됨
    - coroutine만 초기화 : 안됨
    - 다른 함수 호출해서 그곳에서 마이크 변경 : 안됨

### Implementation

```csharp
private void OnAudioConfigurationChanged(bool deviceWasChanged)
{
    string[] MicNames = Microphone.devices;
    AudioMic.Stop();
    Microphone.End(CurrentDeviceName);
    CurrentDeviceName = MicNames[0];
    AudioMic.clip = Microphone.Start(CurrentDeviceName, true, 1, OutputSampleRate);
    AudioMic.loop = true;
    AudioMic.Play();
    AudioMic.volume = 0f;
    OutputChannels = AudioMic.clip.channels;
    Debug.Log("start mic:: Mic changed to " + CurrentDeviceName);
}

IEnumerator CaptureMic()
{
...
    AudioMic.volume = 0f;

    OutputChannels = AudioMic.clip.channels;
    AudioSettings.OnAudioConfigurationChanged+=OnAudioConfigurationChanged;
...
}
```

- It is more stable to add `AudioSettings.OnAudioConfigurationChanged+=OnAudioConfigurationChanged;` rather than to control coroutines and `Microphone`, `AudioSource` object directly
  - In `OnAudioConfigurationChanged`, I added the procedure of adapting audiosource to current Mic.

## Discussion

- 이후 에러가 발생할 수 있는 지점
  - `AudioDecoder.cs/CreateClip()`에서 조건문에 `true`를 넣어놓음
    - 오디오가 밀리는 이유일 수 있음
- 유니티 앱끼리 통신할 때 서버가 종료하면 클라이언트는 잠깐동안 재연결 요청을 보냄
  - OnServerDisconnect 이벤트 받으면 start scene으로 돌아가도록 구현 필요
