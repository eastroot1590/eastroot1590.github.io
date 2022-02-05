---
layout: post
title: "OCR Scanner 만들어보기"
data: 2021-07-28 13:49:00 +0900
categories: iOS
---

# OCR Scanner

인공지능 기술이 발달하면서 여러 분야에서 인공지능 기술이 사용되고 있다. 사진에 있는 글자를 인식하는 OCR(Optical Character Recognition)도 인공지능이 기술이 적용된 사례 중 하나다. 이 기술은 직접 인공지능을 학습시켜서 만들어낼 수도 있지만 이미 여러 대기업에서 자신들이 학습시킨 인공지능을 사용할 수 있도록 서비스를 제공하고 있다. 다양한 종류의 OCR 기술을 사용해보고 차이점을 분석해보자.

보통 대기업에서 제공하는 사진 인식 인공지능 서비스는 **Vision**이라는 이름을 가지고 있다. 이름이 Vision인 만큼 OCR 뿐 아니라 얼굴 인식, 바코드 인식 등 다양한 사진 인식 기능을 제공한다. 

## Firebase MLVision

구글 **Firebase**에서 제공하는 **MLVision**은 구글 클라우드를 통해 구글 서버로 이미지를 전송하지 않고, 디바이스에서 직접 연산하는 **onDevice** 솔루션도 제공한다. 단, onDevice 모델은 알파벳 기반의 문자만 인식할 수 있고 한글은 인식할 수 없다.

### 개발

Google MLVision 사이트에 굉장히 잘 설명되어 있고, 실제로 사용하는 법도 굉장히 쉽다. Cocoapods을 통해 MLVision을 설치한다.

Firebase API를 호출하기 위해서는 구글 클라우드 플랫폼에 앱을 등록시키고 앱을 실행할 때 마다 Firebase에 앱을 인증해야 한다.

`Vision`이라는 인터페이스를 통해 `VisionTextRecognizer`를 생성할 수 있고, 이 recognizer에 인식할 언어를 옵션으로 추가하고 `VisionImage`를 만들어서 인식 요청을 보낼 수 있다.

```swift
textRecognizer.process(VisionImage(image: sourceImage)) { result, error in
	if let error = error {
		debugPrint(error)
	}

	// 처리...
}
```

### 결과

인식된 결과는 `VisionText`라는 결과로 응답받을 수 있으며, 이를 통해 블록 단위로 인식된 문자와 블록 범위를 얻을 수 있다. `cornerPoints`라는 값도 있지만 **CloudVision**을 사용할 경우 이 값은 없고, frame으로 박스 범위를 얻을 수 있다.

인식률이 굉장히 높다. 괜히 구글이 아니다 싶다. 그런데 박스를 그려보니 `CGRect`로 얻기 때문에 무조건 직사각형 형태가 나오는데 **Kakao Vision**의 경우 네 꼭지점을 얻어서 실제 기울어진 글자는 기울어진 박스를 그릴 수 있는 점과는 차이가 있다.

## Kakao Vision

카카오에서 제공하는 **Kakao Vision**은 구글과 달리 프레임워크 형태로 제공하는 것이 아니라 오픈된 API만 제공한다. 해당 API로 인식을 원하는 이미지를 전송하면 인식된 결과를 응답으로 주기 때문에 온전히 네트워크 환경에서만 사용할 수 있다.

### 개발

일반적인 HTTP 요청과 다르게 **multipart/form-data** 형식으로 요청해야 하기 때문에 애를 좀 먹었다. form-data 요청은 크게 두가지 방법으로 요청할 수 있는데 전송할 파일의 크기가 작을 경우 요청 바디에 파일 바이너리 데이터를 그대로 요청하고, 파일의 크기가 너무 클 경우 업로드 스트림을 생성해서 요청한다. 

iOS에서 주로 사용하는 `Alamofire` 프레임워크의 경우 `upload()`라는 form-data 요청 함수를 제공하는데 내부적으로 파일의 크기가 2MB가 넘어가면 스트림형식으로, 그보다 작을 경우 바디에 데이터를 넣어서 요청한다.

Kakao Vision은 이미지 크기를 1MB, 1,024px로 제한하고 있기 때문에 애초에 전송할 파일이 2MB를 넘을 일이 없어서 데이터를 바디에 넣는 `URLTask`를 사용하기로 했다.

이름이 form-data인 만큼 바디에 데이터를 입력하는 양식이 중요한데 이 양식의 개행(`\r\n`) 하나라도 틀리게 입력하면 정상적인 요청이 안된다. 

```swift
// multipart/form-data body 생성
let boundary: String = "\(UUID().uuidString)"

var body: Data = Data()
body.append("\r\n--\(boundary)\r\n".data(using: .utf8)!)
body.append("Content-Disposition: form-data; name=\"image\"; filename=\"sample.jpeg\"\r\n".data(using: .utf8)!)
body.append("Content-Type: image/jpeg\r\n\r\n".data(using: .utf8)!)
body.append(imageBinary)
body.append("\r\n--\(boundary)--\r\n".data(using: .utf8)!)
```

### 결과

결과는 MLVision과 비슷하게 인식된 문자와 범위를 얻을 수 있다. 차이점은 범위를 `CGRect`가 아니라 `[[Int]]` 즉, 네 꼭지점 좌표를 얻을 수 있다는 점인데, 이 덕분에 실제로 박스를 그렸을 때 좀 더 확실하게 알아볼 수 있다.

인식률은 MLVision에 비해서 좀 떨어지지만 사용할 수 없는 정도는 아니다.

## TesseractOCRiOS

C#으로 개발된 **TesseractOCR**을 Objective-C에서 사용할 수 있게 마이그레이션 시킨 프레임워크로 학습 데이터(.traineddata)를 가지고 디바이스 내에서 OCR 연산을 할 수 있다. MLVision, KakaoVision과 다르게 OCR만 제공하기 때문에 좀 더 가볍다고 할 수 있지만 생각해보면 Kakao Vision은 API를 호출하는 형태라서 Kakao Vision이 앱 입장에서는 가장 가벼울 것 같다.

### 개발

traineddata를 구하는게 쉽지가 않았다. 현재 사용하는 프레임워크 버전에 맞는 traineddata를 사용해야 인식이 시도라도 되기 때문에 무조건 최신 데이터를 구하는게 능사가 아니었고, 여러 파일을 시도한 결과 무려 7년 전에 만들어진 데이터를 사용해서 실행해볼 수 있었다.

CocoaPods으로 프레임워크를 설치한 후 사용할 수 있는데, 원래 Swift로 개발된 프레임워크가 아니다보니 다른 솔루션과는 사용하는 방식이 조금 다르다.

```swift
guard let tesseract = G8Tesseract(language: "kor+eng") else {
	return
}

tesseract.engineMode = .tesseractOnly
tesseract.pageSegmentationMode = .auto
tesseract.image = sourceImage

var ocrResult: [OCRResult] = []

if tesseract.recognize(),
	 let recognizedText = tesseract.recognizedText {
	// 처리...
}
```

### 결과

처참한 인식률을 보여줬다. 그나마 영어는 인식이 잘 되는데 한글은 이상한 글자로 인식되는 경우가 많아서 실제 사용하기는 힘들 것 같다.

## Sample

실제로 테스트 해 볼 수 있는 프로젝트를 만들어봤다. 시작 화면에서 원하는 솔루션을 선택하고 카메라 또는 사진 앱에서 인식할 사진을 선택하면 사진에서 특정 영역을 선택할 수 있도록 에디팅할 수 있는 화면이 표시되고, 영역을 선택하면 해당 영역을 인식한 결과를 볼 수 있다

[Github 링크](https://github.com/eastroot1590/OCRScanner/tree/v1.0)