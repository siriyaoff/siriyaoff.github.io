---
layout: single
title: "Implemention of Stop-video Button for FMETP STREAM"
categories:
  - Dev
tags:
  - Unity
  - C#
  - FMETP
  - Mediapipeline
  - Toggle
  - Event
  - Event handler
---

## Decision making

### 2 ways to implement Stop video Button

**1. Transmit black screen feed for a second and stop**

- client manages all  
  → no need to edit server code

**2. Transmit a frame message**

- client send a message like the one using in connection  
  → server handles the message - server sends another frame message to other clients - other clients change the client’s screen as black individually - i can use the black screen used when screen the self cam

### Analysis

**1. Transmit black screen feed for a second and stop**

- change client-side only
- simple way in 1to1 communication

**2. Transmit a frame message**

- need to examine the framing protocol to know how to make a message with the black screen
- it can be extended to 1ton communication easily
  - 1ton communication requires server-processing anyway::ㄴㄴ  
    ~~NOT SURE~~
    - _~~proceed alignment of `n` feeds in client-side_ makes a big load(network bandwidth, computing power, …)~~
    - ~~it may be more effective to _proceed compilated feeds for each client in server_~~
- additional implementation needed in server-side
  - actually all the components(sender, server, receiver) must be considered

### Result

- I chose the first way
- list to examine
  - webcam feed initialization
  - avatar initialization
  - the way to send black screen for a moment

# Implementation

## 1. Toggle button added

- UI - Toggle used  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-1.png)
  - 잘 안보이지만, Game view 가장 아래쪽에 토글 버튼과 회색 Stop video 글자가 있음
- Inspector of Stop video toggle  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-2.png)
  - set `Is On` to inactive
  - add `Stopvideo.toggleMeshes()`, `HolisticTrackingSolution.toggleAvatar()` to Event Trigger `On Value Changed`
- `Stopvideo.cs`

  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;

  public class Stopvideo : MonoBehaviour
  {
      [SerializeField] GameObject[] meshes;
      public void toggleMeshes()
      {
          for(int i = 0; i < meshes.Length; i++)
          {
              meshes[i].SetActive(!meshes[i].activeSelf);
          }
      }
  }
  ```

  - toggle status of each mesh of the avatar

## 2. Implement mediapipe-controlling function

### Mediapipe analysis

![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-3.png)

- UML에 따라서 그린게 아님
- 실선 화살표는 의존, 점선 화살표는 Implementation을 나타냄
  - A (실선 화살표)> B : A가 B에 의존함
  - A (점선 화살표)> B : A는 B를 구현한 것임
- `ImageSource`는 피드로 줄 소스를 제공하는 인퍼페이스임
  - `WebCamSource`, `StaticImageSource`, `VideoSource` 등의 구현체가 존재함
- `Bootstrap`에서 `Glog`(global flags), `AssetLoader`(Streaming asset 등), `ImageSource`를 초기화함
- `Solution` : `Bootstrap`을 초기화하고 `ImageSource`를 읽는 메소드들을 제공
- `GraphRunner` : avatar tracking inference 실행
- `ImageSourceSolution` : `Solution`의 구현체
  - `ImageSource`에 맞춰 `GraphRunner`를 초기화하고 그 결과값을 input stream에 넣음
  - `AddTextureFrameToInputStream()`까지의 코드에서 byte code를 생성함

### Implementing `toggleAvatar()`

- `toggleAvatar()` in `ImageSourceSolution.cs`
  ```csharp
  public void toggleAvatar(){
  	if(isRunning){
  	  Stop();
  	  base.controlStopvideoBtn();
  	  int tmp=0;
  	  base.Play();
  	  StartCoroutine(Run());
  	  tmp=0;
  	  Stop();
  	}
  	else{ // screen is black
  	  base.Play();
  	  base.controlStopvideoBtn();
  	  Play();
  	}
  }
  ```
  - `isRunning`을 필드로 추가하고 `Play()`, `Pause()`, `Resume()`, `Stop()`을 실행할 때 갱신함
  - `Run()`이 은 graphrunner, imagesource이 초기화되어있지 않으면 초기화하고 TextureFrame을 생성함
- `controlStopvideoBtn()` in `Solution.cs`
  ```csharp
  public void controlStopvideoBtn()
  {
      bootstrap.toggleImageSource();
  }
  ```
  - bootstrap에서의 `toggleImageSource()`를 실행시키기 위해 구현함
- `toggleImageSource()` in `Bootstrap.cs`
  ```csharp
  public void toggleImageSource(){
    var imageSource=ImageSourceProvider.ImageSource;
    if(imageSource.isPrepared==true) imageSource.Stop();
    if(_defaultImageSource==ImageSourceType.Image) // screen is black
      _defaultImageSource=ImageSourceType.WebCamera;
    else
      _defaultImageSource=ImageSourceType.Image;
    StartCoroutine(Init());
  }
  ```
  - `ImageSource`를 바꿈

### Add Blackscreen image to `Assets`

- 최상위 경로의 `Assets` 폴더에 black screen 파일들을 넣음  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-4.png)
- `ImageSource`의 구현체 중에 하나인 `StaticImageSource`에 추가함  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-5.png)

# Result

![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-6.png)

- Stop video 버튼을 클릭하면 black screen이 1frame이 전송되고 피드가 멈춤
- 다시 클릭하면 아바타가 켜진 후 피드가 보내지면서 4프레임 안에 웹캠으로 ImageSource가 전환됨

# Discussion

- black screen이 1frame 전송되는 것과 웹캠 피드를 재시작할 때 보내지는 4프레임은 따로 설정한 값이 아님
  - 웹캠 피드 재시작은 이후 요청이 들어오면 잠시 대기 시간을 걸어서 없앨 수 있음
    - `toggleAvatar()`의 `tmp`가 원래는 `while(t++<100000000);`으로 대기시간을 만드는 용도였음
  - black screen이 1 frame만 전송되는건 이상적인 케이스인데, 아마 `toggleAvatar()`에서 `StartCoroutine(Run())`을 실행하고 바로 `Stop()`을 호출해서 현재 실행중인 iteration만 실행하고 바로 종료되는 듯

# Miscellaneous

- byteframe 생성하기 위해서 본 함수
  ```csharp
  public void SetPixels32(Color32[] pixels)
  {
    _texture.SetPixels32(pixels); // length : 307200
    Debug.Log(pixels[0].ToString());
    Debug.Log(pixels[1].ToString());
    Debug.Log(pixels[2].ToString());
    Debug.Log(pixels[3].ToString());
  ```
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-Stop-video-Button-7.png)
  - rgba 형식임
  - setpixels32에 파라미터 추가해서 `[0, 0, 0, 255]`로 바꿀 수는 있지만 Textureframe을 생성하는 iteration 하나를 독자적으로 실행할 방법을 못찾음
