---
layout: post
title: "Estimator: BLE를 사용한 Planning Poker 애플리케이션"
author: 전수열
author-email: jeon@stylesha.re
description: BLE(Bluetooth Low Enery) 기술을 사용한 Planning Poker 애플리케이션 Estimator를 개발하게 된 배경과 경험을 공유합니다.
publish: true
---

## 1. Planning Poker

StyleShare 개발팀에서는 스크럼을 활용하여 일을 진행하고 있습니다. 스크럼에는 일감의 크기를 추정<sup>estimate</sup>하는 과정이 있는데요. 구성원들 모두가 일감에 대해 이해하고 일감의 크기가 어느정도인지 함께 논의하여 합의에 이르는 과정입니다. 스프린트 회의에서 일감을 등록한 사람(리포터)이 일감에 대해 설명하고 나서 전체 구성원들이 일감의 크기를 추정하는데, 이 때 사용하는 것이 바로 Planning Poker입니다.

Planning Poker는 0.5부터 시작해서  1, 3, 5, 8, 13, 20, ... 100과 같이 피보나치 수열로 증가하는 숫자를 가진 카드 덱입니다. 리포터의 설명이 끝난 뒤 스크럼 마스터가 하나, 둘, 셋을 외치면 각자 생각한 일감의 크기에 맞는 카드를 꺼내고, 스크럼 마스터는 구성원들의 추정치가 최대한 가까워지도록 부가설명이나 질문을 유도합니다.[^1]

<figure>
  <img src="http://blog.controlgroup.com/wp-content/uploads/2015/07/planningpoker.jpg">
  <figcaption>
  &#9650; Planning Poker는 이렇게 생겼다. (출처: <a href="http://blog.controlgroup.com/2015/07/13/a-modest-proposal-for-planning-poker-alternatives/">Control Group 블로그</a>)
  </figcaption>
</figure>

하지만 개발팀이 커지면서 불편함이 생기기 시작했습니다. 회의에 참여하는 인원이 7-8명씩 되다 보니, 각자가 어떤 카드를 들고있는지 한눈에 보기가 어려워진 것입니다. StyleShare에서 자칭 아이디어 뱅크 역할을 담당하고 있는 저는 획기적인 방법이 필요하다고 생각했고, 굳이 카드를 꺼내들지 않아도 각자가 무슨 카드를 선택했는지를 쉽게 볼 수 있는 애플리케이션을 만들기로 결심했습니다.

## 2. BLE (Bluetooth Low Energy)

불편함을 덜기 위한 애플리케이션이므로, 사용자 경험이 굉장히 직관적이고 단순해야 했습니다. N:N 통신이 가능해야하고, 사용자를 귀찮게 하는 페어링<sup>Pairing</sup>이나 네트워크 접속 과정이 없어야 했습니다. 한마디로, **카드를 꺼내들고 눈으로 확인하는 것보다 더 편한 무언가**를 만들어야 했습니다!

처음에는 근거리 무선 통신을 위한 기술로 스타벅스에서 사이렌 오더 개발에 사용한 고주파 인식 기술을 생각했습니다.[^2] 각자의 기기에서 선택한 카드에 맞는 소리를 내보내고, 다른 기기에서는 고주파를 읽겠다는 것이었는데요. [Soundlly](http://www.soundl.ly/)(구 aircast.me)와 같은 상업용 SDK를 쓰지 않는 이상, 사운드 프로그래밍을 한 번도 해본 적 없는 저에게는 데이터가 실린 고주파를 만드는 것부터 소리를 인식해서 데이터를 읽어내는 과정이 마치 화성에서 감자 키우는 이야기처럼 들렸습니다.

그러다 문득 생각난 것이 바로 비콘<sup>Beacon</sup>입니다. 언젠가 소비자가 오프라인 매장에 방문하면 BLE를 이용해서 매장 위치를 파악하는 기술이 있다는 이야기를 들은 적이 있었습니다. 찾아보니 시중에 나와있는 대부분의 모바일 기기에서는 BLE를 위한 최소 조건인 블루투스 4.0을 지원했고, 페어링이나 네트워크 접속 과정도 불필요했습니다. 무엇보다, 화성에서 감자 키우는 것보다는 쉬워보였습니다.

## 3. Swift로 BLE 개발하기

그래서 BLE를 사용해서 개발하기로 했습니다. 컨셉은 간단했습니다. 내가 선택한 카드를 브로드캐스팅하고, 다른 사람들이 선택한 카드를 내 모바일 기기에 보여주면 되는 것이었습니다. BLE를 사용하면 정보를 브로드캐스팅할 수 있고, 다른 기기에서 브로드캐스팅하는 정보를 읽을 수 있습니다.

BLE에서 데이터를 브로드캐스팅하는 것을 *Advertising*이라고 합니다. 정보를 advertising하는 주체는 *Peripheral*이고, advertising되는 정보를 스캔하여 데이터를 읽어들이는  주체는 *Central*이라고 합니다. Peripheral에서 정보를 advertising할 때에는 특정한 정보를 실어나를 수 있는데요. 이를 *Advertising Data Payload*라고 합니다. 이 정보에 카드 숫자와 이름을 실어서 전송하면 될 것 같습니다.

BLE를 구현하기 위해서, iOS에서는 SDK에 기본적으로 포함돼있는 [`CoreBluetooth`](https://developer.apple.com/library/ios/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html) 프레임워크를 사용하면 손쉽게 개발이 가능합니다. [`CBPeripheralManager`](https://developer.apple.com/library/prerelease/ios/documentation/CoreBluetooth/Reference/CBPeripheralManager_Class/index.html) 클래스와 [`CBCentralManager`](https://developer.apple.com/library/prerelease/ios/documentation/CoreBluetooth/Reference/CBCentralManager_Class/index.html) 클래스를 쓰면 되는데요. BLE를 이용하여 제 이름 석자를 advertising하는 코드는 다음과 같습니다.

**Peripheral**

```swift
import CoreBluetooth

let serviceUUID = CBUUID(string: "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX")
let service = CBMutableService(type: serviceUUID, primary: true)

/// 1. `CBPeripheralManager`를 초기화하고,
self.peripheral = CBPeripheralManager(delegate: self, queue: nil)

/// 2. 사용가능한 상태가 되면 특정 UUID를 가진 서비스를 추가한 뒤에
func peripheralManagerDidUpdateState(peripheral: CBPeripheralManager) {
    if peripheral.state == .PoweredOn {
        self.peripheral.addService(service)
    }
}

/// 3. 원하는 정보를 advertising합니다.
func peripheralManager(peripheral: CBPeripheralManager,
                       didAddService service: CBService,
                       error: NSError?) {
    self.peripheral.startAdvertising([
        CBAdvertisementDataLocalNameKey: "전수열",
        CBAdvertisementDataServiceUUIDsKey: [serviceUUID],
    ])
}
```

> 참고로, UUID는 커맨드라인 명령어를 통해 쉽게 만들 수 있습니다.
>
> ```console
> $ uuidgen
> XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
> ```

마찬가지로, Peripheral에서 advertising하는 정보를 스캔하는 Central 코드는 다음과 같이 작성할 수 있습니다. UUID는 Peripheral에서 advertising에 사용한 UUID와 동일해야합니다.

**Central**

```swift
import CoreBluetooth

let serviceUUID = CBUUID(string: "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX")
let service = CBMutableService(type: serviceUUID, primary: true)

/// 1. `CBCentralManager`를 초기화하고,
self.central = CBCentralManager(delegate: self, queue: nil)

/// 2. 사용가능한 상태가 되면 특정 UUID를 가진 서비스를 스캔합니다.
func centralManagerDidUpdateState(central: CBCentralManager) {
    if central.state == .PoweredOn {
        // 이미 한 번 스캔된 정보라도 계속 스캔합니다.
        let options = [CBCentralManagerScanOptionAllowDuplicatesKey: true]
        self.central.scanForPeripheralsWithServices([serviceUUID], options: options)
    }
}

/// 3. Peripheral이 스캔되면 이 메서드가 호출됩니다.
func centralManager(central: CBCentralManager,
                    didDiscoverPeripheral peripheral: CBPeripheral,
                    advertisementData: [String : AnyObject],
                    RSSI: NSNumber) {
    print(advertisementData[CBAdvertisementDataLocalNameKey]) // "전수열"
}
```

스캔을 시작할 때 `CBCentralManagerScanOptionAllowDuplicatesKey` 옵션을 `true`로 설정해서 한 번 스캔된 정보라도 중복으로 계속 스캔하도록 합니다.

## 4. 원하는 정보를 실어나르기

`CBPeripheralManager`을 사용하여 advertising을 할 때에는 Advertising Data Payload를 포함시킬 수 있는데, 이 정보 중 개발자가 원하는 값을 넣을 수 있는 곳은 `CBAdvertisementDataLocalNameKey`밖에 없습니다. 그마저도 길이가 제한돼있기 때문에, 패킷을 효율적으로 사용하기 위해서는 정보를 저장하는 프로토콜을 직접 정의해야 합니다.

우선, 카드에 대한 정의는 `enum`을 사용해서 작성했습니다. 0부터 0xFF까지의 숫자를 가지도록 정의했습니다.

```swift
public enum Card: Int {
    case Zero = 0
    case Half = 127
    case One = 1
    case Two = 2
    case Three = 3
    case Five = 5
    case Eight = 8
    case Thirteen = 13
    case Twenty = 20
    case Fourty = 40
    case Hundred = 100
    case QuestionMark = 0xFD
    case Coffee = 0xFE
    case None = 0xFF
}
```

그리고 제가 정의한 패킷의 프로토콜은 다음과 같습니다.

| 영역 | 길이 | 예시 | 설명 |
|---|---:|:---:|---|
| Version | 2 | `00` | 프로토콜 버전 (`00`~`FF`) |
| Channel | 2 | `01` | BLE 커버리지 내에서 회의하는 팀이 여럿일 수 있으니, 채널로 구분합니다. (`00`~`FF`) |
| Card | 2 | `FE` | 카드의 16진수 값 (`00`~`FF`) |
| Name | 12 | `전수열` | 사용자 이름 (UTF-8 기준 한글 4글자) |

이렇게 하면 총 18바이트 내에서 필요한 정보를 모두 전송할 수 있습니다. 이제 이 `"00"`, `"01"`, `"FE"`, `"전수열"` 값을 직렬화해서 `CBAdvertisementDataLocalNameKey`로 advertising하면 됩니다.

**Peripheral**

```swift
self.peripheral.startAdvertising([
    CBAdvertisementDataLocalNameKey: "0001FE전수열",
    CBAdvertisementDataServiceUUIDsKey: [serviceUUID],
])
```

그리고, Central에서 정보를 스캔할 때에는 이 값을 각 영역의 길이에 맞게 끊어서 읽을 수 있습니다.

**Central**

```swift
func centralManager(central: CBCentralManager,
                    didDiscoverPeripheral peripheral: CBPeripheral,
                    advertisementData: [String : AnyObject],
                    RSSI: NSNumber) {
    guard let packet = advertisementData[CBAdvertisementDataLocalNameKey] as? String else {
        return
    }
    let version = Int(packet[packet.startIndex..<packet.startIndex.advancedBy(2)])
    let channel = Int(packet[packet.startIndex.advancedBy(2)..<packet.startIndex.advancedBy(4)])
    let cardRawValue = Int(packet[packet.startIndex.advancedBy(4)..<packet.startIndex.advancedBy(6)], radix: 16)
    let card = Card(rawValue: cardRawValue)
    let name = packet[packet.startIndex.advancedBy(6)..<packet.endIndex]
}
```

## 5. 마치며

비록 적은 양의 정보지만, BLE를 사용해서 실시간으로 근거리 통신을 할 수 있게 되었습니다. 이제 남은 것은 카드를 선택할 수 있는 화면과, 다른 사용자가 선택한 카드를 화면에 보여주는 인터페이스입니다. UI 개발은 본 포스트에서 중점적으로 다루고자 하는 주제와는 조금 벗어난 이야기가 될 것 같아, 오픈소스로 공개된 코드로 대신하려고 합니다. 소스코드는 [GitHub](github.com/devxoul/Estimator)에서 볼 수 있으며, Estimator는 [앱스토어]()에서 받아보실 수 있습니다.

## 6. 참고 자료

- [BLE(BLUETOOTH LOW ENERGY) 이해하기 - Hard Copy World](http://www.hardcopyworld.com/gnuboard5/bbs/board.php?bo_table=lecture_tip&wr_id=20)


[^1]: 구성원들의 추정치에 차이가 난다는 것은 해당 일감에 대해 서로가 이해하고 있는 정도가 다르기 때문입니다. 스크럼 마스터는 구성원들이 일감에 대해 모두 비슷한 생각을 가지도록 커뮤니케이션을 유도해야합니다.
[^2]: http://www.bloter.net/archives/226643
