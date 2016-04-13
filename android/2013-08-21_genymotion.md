# Genymotion

Genymotion을 소개합니다.

## Android Emulator

정말 느리죠. 해보신 분들만 아는 그 답답함!

차라리 중고 후진 기기를 사는게 마음이 편할 정도죠.

요즘이야 Intel용 이미지가 있으니 그나마(?) 좀 나아지긴 했지만.. iOS Emulator와 비교대상이 못되는 것 같아요.

## Genymotion은?

[Genymotion]은 [AndroVM] 프로젝트의 새 버전이라고 합니다.

![](http://media.tumblr.com/abb231c517198e97cabf15f0b9122813/tumblr_inline_mrv0a8bH9F1qz4rgp.png)

Multi Platform을 지원하고, IntelliJ IDEA와 Eclipse 둘 다 Plugin을 지원하고 있습니다.

![](http://media.tumblr.com/3106be1497df413e16f4e2ba8c374c3e/tumblr_inline_mrv0imKZgZ1qz4rgp.png)


### 어떻게 생겼나?

일단 실행하고 로그인을 하면 아래와 같은 화면이 나오네요.

![](http://media.tumblr.com/c6cb096f92e651909eb0ebc5c89c9e96/tumblr_inline_mrv0ahe0z41qz4rgp.png)

기기 하나를 추가해봤습니다.

![](http://media.tumblr.com/3e86c121fa820dec5045c639511a28bd/tumblr_inline_mrv0ar5ovH1qz4rgp.png)

기기를 선택하고 Add를 누르면... 확인창이 뜨고

![](http://media.tumblr.com/ee8a7c8e26a4f4368a374e6ad6a45b2a/tumblr_inline_mrv0b0RgIl1qz4rgp.png)

Next를 누르면... 다운로드를 받습니다.

![](http://media.tumblr.com/3d6ad9081b69787def38223c6280a6db/tumblr_inline_mrv0be6wYy1qz4rgp.png)

기기 이름을 정하고 Create를 누르면 VM이 만들어집니다.

![](http://media.tumblr.com/cf3493da221d018da0f151dab1125683/tumblr_inline_mrv0bmJzN41qz4rgp.png)

![](http://media.tumblr.com/bbe8db979bcdde7fd6c52865f2ce565f/tumblr_inline_mrv0btzPi71qz4rgp.png)

실행을 시켜볼까 했더니... SDK Path를 지정하라고 하네요. Proxy 설정할 수 있는 부분도 맘에 듭니다. VirtualBox를 사용하니 당연히 지원할 수 있는 메뉴겠지만..

![](http://media.tumblr.com/1d071da54dbc7ecb96fc01edd35b41ea/tumblr_inline_mrv0c1uurP1qz4rgp.png)

경로 설정하고 실행하면 아래와 같이 실행이 됩니다. 속도는 얼마 안걸리네요.

![](http://media.tumblr.com/da693e1de70e9cbefdb5690a6d4c2045/tumblr_inline_mrv0c8JAtt1qz4rgp.png)

그런데 우측에 몇 가지 메뉴가 있는데... 베터리 상태도 지정할 수 있구요.

![](http://media.tumblr.com/0696b1a84197da2692e1737527437900/tumblr_inline_mrv0cnHcek1qz4rgp.png)

GPS 상태도 지정할 수 있습니다.

![](http://media.tumblr.com/63d9d6d0cb523f667c1572bcd778a5d0/tumblr_inline_mrv0d7Pi2X1qz4rgp.png)

움직임도 부드럽고, IDE와 연동해 사용하기도 편하네요.

![](http://media.tumblr.com/981bd03e444b705aedf5d6bc3ccd97b9/tumblr_inline_mrv0cgylKE1qz4rgp.png)

### 아쉬운 점은?
Android Emulator는 느리긴 하지만 CI 툴에서 Headless로 띄울 수 있는데 이 툴은 얼핏 보면 그렇게 할 수 없는 듯 합니다.

하지만 VirtualBox를 사용하고 있으니 그리 불가능해 보이진 않을 것 같은데.. (Jenkins Plugin 같은) 쉬운 방법을 제공해주면 좋을 것 같네요.

여튼 Emulator를 사용하고 계신다면 반드시 쓰셔야할 App인듯 싶네요.

[Genymotion]: http://www.genymotion.com/  "The fastest Android emulator for app testing and presentation"
[AndroVM]: http://androvm.org/blog/  "AndroVM blog"