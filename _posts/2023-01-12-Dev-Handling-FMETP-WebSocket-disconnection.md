---
layout: single
title: "Handling FMETP WebSocket disconnection"
categories:
  - Dev
tags:
  - Unity
  - C#
  - FMETP
  - WebSocket
  - Disconnection
  - Event
  - Event handler
  - Toggle
  - AssetLoader
---

## App configuration

### Scenes

- StartScene_General  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-1.png)
- Holistic  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-2.png)
  - StartScene_General에서 Start Session 버튼을 클릭하면 Holistic scene으로 넘어감
  - 오른쪽 버튼들은 가장 위 토글 버튼을 클릭했을 때 확장되는 버튼들임

## Control flow

- StartSession 버튼은 On Click() 이벤트에 대해 MainMenuButtons.CreateSession 메소드가 붙어있음

  ```csharp
  [Header("CreateSession")]
  [SerializeField]
  private FluentTEnums.SceneNames _sceneToLoad = FluentTEnums.SceneNames.MainMenu;

  public void CreateSession()
  {
      var _levelName = "";
      switch (_sceneToLoad)
      {
          case FluentTEnums.SceneNames.FaceTracking:
              _levelName = StaticVariables.SCENE_FACE_TRACKING;
              break;
          case FluentTEnums.SceneNames.HolisticTracking:
              _levelName = StaticVariables.SCENE_HOLISTIC_TRACKING;
              break;
          case FluentTEnums.SceneNames.MainMenu:
              _levelName = StaticVariables.SCENE_MAIN_LEVEL;
              break;
          case FluentTEnums.SceneNames.TestMainMenu:
              _levelName = StaticVariables.SCENE_TEST_MAIN_LEVEL;
              break;
          default:
              Debug.LogWarning("Something went wrong with the enum. Check what happened.");
              return;
      }

      LoadLevel(_levelName);
      Debug.Log("Starting session...");

  }
  private void LoadLevel(string levelName)
  {
      StartCoroutine(LoadSceneAsync(levelName));
  }

  IEnumerator LoadSceneAsync(string levelName)
  {
      AsyncOperation op = SceneManager.LoadSceneAsync(levelName);

      while (!op.isDone)
      {
          float progress = Mathf.Clamp01(op.progress / .9f);
          Debug.Log(op.progress);

          yield return null;
      }
  }
  ```

  - `_sceneToLoad`는 SerializeField이기 때문에 유니티 에디터에서 변경될 수 있음
  - `StartCoroutine(LoadSceneAsync(levelName))`으로 scene을 호출하는 것을 알 수 있음

- `Holistic` scene이 로드되면 `Solution` game object가 mediapipeline을 초기화함  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-3.png)

## Implement disconnect button

- `Holistic`의 토글 메뉴에 `X` 버튼을 추가함

  - 아래와 같은 스크립트를 추가함

  ```csharp
  public void GotoStartScene()
      {
          var _levelName = StaticVariables.SCENE_MAIN_LEVEL;
          StartCoroutine(LoadSceneAsync(_levelName));
          Debug.Log("Session closed");
      }

      IEnumerator LoadSceneAsync(string levelName)
      {
          AsyncOperation op = SceneManager.LoadSceneAsync(levelName);

          while (!op.isDone)
          {
              float progress = Mathf.Clamp01(op.progress / .9f);
              Debug.Log(op.progress);

              yield return null;
          }
      }
  ```

  - `On Click()` 이벤트에 아래와 같이 메소드를 붙임  
    ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-4.png)
  - `GotoStartScene()` 메소드를 붙임
    - 버튼을 눌렀을 때 `StartScene_General`으로 화면 전환이 되는 것을 확인함
  - `FMWebSocketManager.Close`도 붙여서 웹소켓도 같이 종료되도록 설정함
    - EC2에 배포한 TomatoSaas 서버로 테스트했을 때 정상 접속, 종료되는 것을 확인함

## Trouble shooting

### Start session → X → Start session으로 다시 `Holistic` scene으로 넘어갔을 때 웹캠 화면이 나오지 않음

- 처음 한 번만 실행해서 정상 작동할 때의 로그  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-5.png)
- GotoStartsession 호출 후 다시 Start session 버튼을 눌렀을 때의 로그  
  ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-6.png)
- `LoadSceneAsync`는 어셋에서 제공해주는 함수임
- 웹캠 피드를 담당하는 MediaPipeUnity 패키지의 초기화 문제라 생각하고 유니티에서 MediaPipeline을 담당하는 game object를 찾음 → `solution`
  - `solution`이 MediaPipeUnity의 스크립트들을 호출하고 있었음  
    ![Untitled](https://raw.githubusercontent.com/siriyaoff/siriyaoff.github.io/master/assets/img/FMETP-ws-disconnection-7.png)
  - `MediaPipeException` 에러의 스택 가장 처음을 보면 `Bootstrap:OnEnable()`에서 에러가 난다는 것을 알 수 있음
    - bootstrap 초기화 문제라 생각하고 로그 찍어본 결과 AssetLoader 부분에서 에러남
    ```csharp
    Logger.LogInfo(_TAG, "Initializing AssetLoader...");
    try
    {
        switch (_assetLoaderType)
        {
            case AssetLoaderType.AssetBundle:
                {
                    AssetLoader.Provide(new AssetBundleResourceManager("mediapipe"));
                    break;
                }
            case AssetLoaderType.StreamingAssets:
                {
                    AssetLoader.Provide(new StreamingAssetsResourceManager());
                    break;
                }
            case AssetLoaderType.Local:
                {
    #if UNITY_EDITOR
                    AssetLoader.Provide(new LocalResourceManager());
                    break;
    #else
    Logger.LogError("LocalResourceManager is only supported on UnityEditor");
    yield break;
    #endif
                }
            default:
                {
                    Logger.LogError($"AssetLoaderType is unknown: {_assetLoaderType}");
                    yield break;
                }
        }
    }
    catch
    {
        Logger.LogInfo(_TAG, "Asset already loaded");
    }
    ```
    - try…catch로 감싸서 해결함

### host가 종료했을 때 client들은 `Holistic` scene에 계속 머무름

- 세션이 닫히면 client들도 시작 페이지에 돌아가도록 설정해야 함
