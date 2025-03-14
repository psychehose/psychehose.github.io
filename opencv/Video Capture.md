
### Video Capture
`cv::VideoCapture` 클래스는 OpenCV에서 카메라 장치나 비디오 파일을 처리하기 위한 클래스다.

#### 생성자

생성자는 다양한 비디오 소스에 연결하는 기능을 제공

```cpp
VideoCapture();

// Camera 연결
explicit VideoCapture(int index, int apiPreference = CAP_ANY);

// 파일
explicit VideoCapture(const String& filename, int apiPreference = CAP_ANY);

VideoCapture(const Ptr<IStreamReader>& source, int apiPreference, const std::vector<int>& params);

```


* `apiPreference` - API 백엔드
	- **용도**: 비디오 캡처에 사용할 특정 백엔드 API를 지정합니다.
	- **주요 옵션**:
	    - `cv::CAP_ANY`: 자동 선택 (기본값)
	    - `cv::CAP_DSHOW`: Windows DirectShow
	    - `cv::CAP_AVFOUNDATION`: macOS AVFoundation
	    - `cv::CAP_V4L2`: Linux Video4Linux2
	    - `cv::CAP_GSTREAMER`: GStreamer 프레임워크
	    - `cv::CAP_FFMPEG`: FFmpeg 라이브러리

- `const Ptr<IStreamReader>& source`
	- **`Ptr<>`**:  OpenCV에서 사용하는 스마트 포인터 클래스입니다. 자동 메모리 관리를 제공
	- **`IStreamReader`**: 데이터 스트림을 읽기 위한 인터페이스 클래스
	- **용도**: 사용자 정의 데이터 소스에서 비디오 프레임을 읽을 수 있게 합니다


### VideoWriter

OpenCV에서 비디오 파일을 생성하고 인코딩하기 위한 클래스

#### 생성자

```cpp
CV_WRAP VideoWriter(const String& filename, int fourcc, double fps,
                Size frameSize, bool isColor = true);

    CV_WRAP VideoWriter(const String& filename, int apiPreference, int fourcc, double fps,
                Size frameSize, bool isColor = true);

    CV_WRAP VideoWriter(const String& filename, int fourcc, double fps, const Size& frameSize,
                        const std::vector<int>& params);

    CV_WRAP VideoWriter(const String& filename, int apiPreference, int fourcc, double fps,
                        const Size& frameSize, const std::vector<int>& params);


```

* `int fourcc` 
	* Four Character Code - 비디오 코덱을 4개의 문자로 식별하는 방법
	* `a','v','c','1'` : H.264 코덱
	* `'M','J','P','G'`: Motion JPEG
	- `'X','V','I','D'`: XVID MPEG-4
	- `'m','p','4','v'`: MPEG-4
	- `'D','I','V','X'`: DIVX MPEG-4
	  
* `double fps`: Frame rate 
* Size& frameSize: 해상도

예시

```cpp
cv::VideoWriter writer(videoFilename,
                               cv::VideoWriter::fourcc('a', 'v', 'c', '1'),
                               FPS,
                               frame.size());
```

#### 파일에 프레임 쓰기 & 닫기

```cpp
for (const auto& f : allFrames) {
    writer.write(f); // 쓰기
}

writer.release(); // 닫기
```



### 화면에 프레임 업데이트

```cpp
// 창 미리 생성
cv::namedWindow("카메라", cv::WINDOW_NORMAL);

while (true) {
    // 카메라에서 새 프레임 읽기
    camera.read(frame);
    
    // "카메라" 창에 새 프레임 표시/업데이트
    cv::imshow("카메라", frame);
    
    // GUI 이벤트 처리 및 키 입력 대기
    int key = cv::waitKey(1);
    if (key == 'q') break; // q키 누르면 종료
}
```





### Full code

#### VideoBuffer.h
```cpp

#include <deque>
#include <mutex>
#include <opencv2/opencv.hpp>
#include <vector>

class VideoBuffer {
 private:
  std::deque<cv::Mat> frames;
  const int maxFrames;
  std::mutex mtx;

 public:
  VideoBuffer(int capacity) : maxFrames(capacity) {}

  void addFrame(const cv::Mat& frame) {
    std::lock_guard<std::mutex> lock(mtx);
    frames.push_back(frame.clone());
    if (frames.size() > maxFrames) {
      frames.pop_front();
    }
  }

  std::vector<cv::Mat> getAllFrames() {
    std::lock_guard<std::mutex> lock(mtx);
    return std::vector<cv::Mat>(frames.begin(), frames.end());
  }

  size_t size() {
    std::lock_guard<std::mutex> lock(mtx);
    return frames.size();
  }
};
```

#### main.cpp

```cpp
#include <atomic>
#include <iostream>
#include <opencv2/opencv.hpp>

#include "VideoBuffer.h"

int main() {
  // 설정값
  const int FPS = 30;
  const int PRE_SECONDS = 3;
  const int POST_SECONDS = 3;
  const int BUFFER_SIZE = FPS * PRE_SECONDS;

  // OpenCV 카메라 초기화 (메인 스레드에서)
  cv::VideoCapture camera(0);

  if (!camera.isOpened()) {
    std::cerr << "카메라를 열 수 없습니다." << std::endl;
    return -1;
  }

  // 윈도우 생성 (메인 스레드에서)
  cv::namedWindow("카메라", cv::WINDOW_NORMAL);

  // 링 버퍼 초기화
  VideoBuffer preBuffer(BUFFER_SIZE);
  std::vector<cv::Mat> postBuffer;

  // 상태 변수
  std::atomic<bool> isRecording(false);
  std::atomic<bool> shouldQuit(false);
  std::atomic<int> postFrameCount(0);

  bool videoSaved = false;
  std::string videoFilename = "captured_video.mp4";

  std::cout << "시작되었습니다. 's'를 눌러 캡처, 'q'를 눌러 종료, 'p'를 눌러 재생" << std::endl;

  // 메인 루프 (UI 작업을 포함)
  cv::Mat frame;
  while (!shouldQuit) {
    // 카메라에서 프레임 읽기
    if (!camera.read(frame)) {
      std::cerr << "프레임을 읽을 수 없습니다." << std::endl;
      break;
    }

    // 현재 프레임을 미리 버퍼에 추가
    preBuffer.addFrame(frame);

    // 녹화 중이면 사후 버퍼에 추가
    if (isRecording) {
      postBuffer.push_back(frame.clone());
      postFrameCount++;

      // 사후 녹화 완료 확인
      if (postFrameCount >= FPS * POST_SECONDS) {
        isRecording = false;

        // 비디오 저장
        std::vector<cv::Mat> preFrames = preBuffer.getAllFrames();
        std::vector<cv::Mat> allFrames;
        allFrames.insert(allFrames.end(), preFrames.begin(), preFrames.end());
        allFrames.insert(allFrames.end(), postBuffer.begin(), postBuffer.end());

        cv::VideoWriter writer(videoFilename,
                               cv::VideoWriter::fourcc('a', 'v', 'c', '1'),
                               FPS,
                               frame.size());

        if (!writer.isOpened()) {
          std::cerr << "비디오 파일을 저장할 수 없습니다." << std::endl;
        } else {
          for (const auto& f : allFrames) {
            writer.write(f);
          }
          writer.release();
          std::cout << "비디오가 저장되었습니다: " << videoFilename << std::endl;
          videoSaved = true;
        }

        postBuffer.clear();
        postFrameCount = 0;
      }
    }

    // 프레임 표시 (메인 스레드에서)
    cv::imshow("카메라", frame);

    // 키 입력 처리
    int key = cv::waitKey(1);
    if (key == 'q' || key == 'Q') {
      shouldQuit = true;
    } else if (key == 's' || key == 'S') {
      if (!isRecording) {
        std::cout << "캡처 시작됨..." << std::endl;
        isRecording = true;
        postBuffer.clear();
        postFrameCount = 0;
      }
    } else if (key == 'p' || key == 'P' && videoSaved) {
      std::cout << "녹화된 비디오 재생 중..." << std::endl;

      cv::VideoCapture player(videoFilename);
      if (!player.isOpened()) {
        std::cerr << "비디오 파일을 재생할 수 없습니다." << std::endl;
      } else {
        cv::namedWindow("재생", cv::WINDOW_NORMAL);
        cv::Mat playFrame;
        while (player.read(playFrame)) {
          cv::imshow("재생", playFrame);
          if (cv::waitKey(1000 / FPS) >= 0) {
            break;
          }
        }
        cv::destroyWindow("재생");
        player.release();
      }
    }
  }

  // 정리
  camera.release();
  cv::destroyAllWindows();

  return 0;
}
```