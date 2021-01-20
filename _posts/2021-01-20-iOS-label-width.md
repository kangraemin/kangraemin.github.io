---
title: iOS - UILabel 크기를 텍스트에 맞추기
date: 2021-01-20
categories:
 - iOS
tags:
 - iOS
---

> iOS의 UILabel의 크기를 text에 알맞게 조정하는 방법에 대해 공유합니다. 

<!-- more -->

## 문제점

```swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet weak var testLabel: UILabel!
    
    override func viewDidLoad() {
        testLabel.text = "테스트트트트"
    }
}
```
![pic1.png](/assets/images/posts/2021-01-20-iOS-label-width/pic1.png)

위 경우, UILabel의 width가 텍스트의 길이보다 짧아 텍스트가 잘리는 현상이 발생하였습니다.

UILabel의 크기를 텍스트의 크기에 맞춰 변경하는 방법을 알아봅니다. 

### 1. sizeToFit() 함수 사용

```swift
import UIKit

class ViewController: UIViewController {
    
    @IBOutlet weak var testLabel: UILabel!
    
    override func viewDidLoad() {
        testLabel.text = "테스트트트트"
        testLabel.sizeToFit()
    }
}
```

![pic2.png](/assets/images/posts/2021-01-20-iOS-label-width/pic2.png)
- sizeToFit 함수는, Subview를 감쌀 수 있도록 해당 뷰의 크기를 재조정 하고, 위치를 재조정 해 줍니다. ((sizeToFit() Docs)[https://developer.apple.com/documentation/uikit/uiview/1622630-sizetofit])
- 해당 메소드를 활용하여, UILabel의 크기를 위와 같이 텍스트 크기에 맞출 수 있습니다. 

### 2. sizeThatFits() 함수 사용

```swift
import UIKit

class CalculatorViewController: UIViewController {
    
    @IBOutlet weak var testLabel: UILabel!
    
    override func viewDidLoad() {
        testLabel.text = "테스트트트트"
        let newSize = testLabel.sizeThatFits( CGSize(width: testLabel.frame.width, height: CGFloat.greatestFiniteMagnitude))
        print(testLabel.frame.size.width)
        print(testLabel.frame.size.height)
        print(newSize.width)
        print(newSize.height)
        testLabel.frame.size.width = newSize.width
        testLabel.frame.size.height = newSize.height
    }
}
```

![pic3.png](/assets/images/posts/2021-01-20-iOS-label-width/pic3.png)
- sizeThatFits함수는, 해당 뷰를 그릴 때 가장 알맞은 크기를 계산하여 리턴 해 줍니다. ((sizeThatFits() Docs)[https://developer.apple.com/documentation/uikit/uiview/1622625-sizethatfits])
- 해당 메소드를 활용하여, 본 포스트의 경우 Text를 모두 표기해 주는 UILabel의 크기를 알아내고, 알아낸 크기에 UILabel의 크기를 맞춤으로써 화면의 모든 텍스트를 표기 할 수 있습니다.


### 3. intrinsicContentSize 값 사용

```swift
import UIKit

class CalculatorViewController: UIViewController {
    
    @IBOutlet weak var testLabel: UILabel!
    
    override func viewDidLoad() {
        testLabel.text = "테스트트트트"
        testLabel.frame.size = testLabel.intrinsicContentSize
    }
}
```

![pic4.png](/assets/images/posts/2021-01-20-iOS-label-width/pic4.png)
- intrinsicContentSize 프로퍼티는, 뷰 안의 컨텐츠( 본 포스트의 UILabel에선 text에 해당 ) 의 실질적인 크기를 리턴 해 줍니다. ((intrinsicContentSize() Docs)[https://developer.apple.com/documentation/uikit/uiview/1622600-intrinsiccontentsize])
- 해당 프로퍼티를 활용하여, UILabel의 크기를 뷰 안의 컨텐츠의 크기를 알아내고, 해당 컨텐츠의 크기에 UILabel의 크기를 맞춤으로써 화면의 모든 텍스트를 표기 할 수 있습니다.