---
description: Dots Per Inch / Scale Factor
---

# 💩 DPI/ScaleFactor

### 맥에서 scale factor를 감지하는 방법

Mac의 화면은 레티나 디스플레이를 사용하기에, 통상적으로 하드웨어적인 해상도를 1/2 다운 스케일링한 디스플레이가 된다. 즉, 화면의 스케일이 2배를 나타나게 되는데, 이를 해결하기 위한 방법으로 scale factor를 불러올 수 있다. 이는 추가적인 디스플레이 연결시 생기는 scale factor간의 차이를 처리하기 위함이다.

아래와 같이 X-code의 Appkit 라이브러리를 사용하여, 현재 화면의 scale factor를 가져올 수 있다. 이는 전체 화면 정보를 다 가져와서, 이 중 현재 화면에 마우스 커서가 위치한 화면을 찾아내고, 그 화면중에서 scale factor를 가져오는 방법이다. 이를 c로 extern 시키면, C/C++에서 사용이 가능하다.

```objectivec
// GetScaleFactor.mm

#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>

extern "C" {
    float getScaleFactor() {
        @autoreleasepool {
            // 현재 포커스된 화면 가져오기
            // NSScreen을 통해 macOS에서 디스플레이의 화면 정보를 가져올 수 있는데, 이 중mainScreen의 정보를 가져온다.
            NSScreen *currentScreen = [NSScreen mainScreen];
            // 마우스 커서의 현재 위치를 가져옴.
            NSPoint mouseLocation = [NSEvent mouseLocation];
            // 현재 연결된 전체 스크린의 정보를 모두 가져온다.(NSScreen screens)
            for (NSScreen *screen in [NSScreen screens]) {
                NSRect frame = [screen frame];
                // 이 중에서 현재 마우스 커서가 위치한 프레임을 찾아, 마우스가 있으면 해당 화면의 정보를 가져온다.
                if (NSPointInRect(mouseLocation, frame)) {
                    currentScreen = screen;
                    break;
                }
            }
            // 현재 화면의 스케일 팩터 가져오기
            return [currentScreen backingScaleFactor];
        }
    }
}
```

이후 cmake에서 컴파일 시에, 아래와 같이 해당 소스파일을 링킹하기 전에 변수로 세팅을 진행한다. 변수로 세팅하는 이유는 컴파일 단에서 해당 파일이 없을 시 문제를 만들지 않기 위함이다. 이후 링킹을 아래와 같이 진행한다.

```cmake
if (APPLE)
    set(MACOS_MM_FILE src/GetScaleFactor.mm)
endif()

add_executable(${PROJECT_NAME}
    ${MACOS_MM_FILE}
}
```

이후, 쓰고자 하는 C/C++ 파일에서 해당 함수를 불러와 사용이 가능하다.

```cpp
#ifdef __APPLE__
    extern "C" {
        float getScaleFactor(); // Objective-C++에서 정의한 함수를 선언
    }
#endif
```
