---
layout: post
title: 소규모팀에 적합한 QA 프로세스 구축기(스타일쉐어팀의 QA방식)
author: 박성환
author-email: walter@styleshare.kr
description: 스타일쉐어 앱의 안정성을 위해 저희팀에서 진행중인 QA과정에 대해서 공유해보고자 합니다.
publish: true
---
안녕하세요. [스타일쉐어](https://www.stylesha.re/)에서 PM을 맡고있는 [박성환](https://twitter.com/UnsteadyFlow) 입니다. 스타일쉐어팀이 **QA프로세스**를 도입한 것은 약 4개월 정도 되었습니다. 기존에는 QA 프로세스 없이 진행했었지만 주요 기능에 대한 오류감소 및 릴리즈 안정성 확보를 위해 도입을 고민하게 되었습니다.

QA프로세스를 처음 도입할때 많은 고민이 있었습니다. 대규모 서비스에 적용하는 QA프로세스를 그대로 도입하기에는 인력 + 시간이 모두 부족했기에 시간과 인력이 많이 투여되는(다만, 안정성이 높음) 명세기반 테스트는 최소화하고, 도입 가능한 서비스(구글플레이의 단계적 배포, Crashlytics)를 활용해 부족한 부분을 커버하는 형식으로 저희 식의 간략화된 QA프로세스를 만들었습니다.(인력 + 시간이 상대적으로 제한적인 스타트업에 좀 더 효율적인 방식.)

- **스타일쉐어팀의 QA 기간 : 앱 업데이트 당 3일(테스트/수정/릴리즈까지의 모든 기간)**
- **테스트 인원 : 2명 (1차QA 1명, 최종확인 1명)**
- **마이너 버그 수정 버전에서는 QA진행하지 않음**

스타일쉐어팀의 QA프로세스는 **“주요 사용 케이스의 동작 확인"** + **"수많은 사용 패턴에 대한 대응”**으로 정리할 수 있습니다. 저희 팀이 진행하고 있는 방식을 조금 더 자세히 설명해 드리자면 아래와 같습니다.(API 테스트, 자동화 테스트를 제외한 앱 릴리즈 전 진행하는 사용성 테스트에 대한 내용만을 담았습니다.)

##1. QA일정

![](/images/2015-04-16-qa-process/qa_schedule.png)

스타일쉐어 앱의 업데이트 주기는 4주에 1회로 진행하고 있습니다. 그 중 1주 단위의 스프린트가 3주 동안 진행되고 4주차 스프린트는 QA 및 릴리즈 스프린트로 진행됩니다. 매 스프린트에서 담당 엔지니어가 수정 혹은 추가된 단위기능에 대해 간단한 테스트가 끝나면 4주차에 알파 빌드 및 전 구성원이 설치/사용해보고 동시에 1차 QA(통합 테스트)를 진행하게 됩니다. 1차 QA의 버그들을 수정하면 베타버전 빌드 및 최종 확인을 진행한뒤 문제없으면 바로 릴리즈가 되어 사용자에게 신규 버전을 제공합니다.

##2. 주요 사용 케이스의 동작 확인

###1) 1차 QA(명세기반 테스팅)

4주차에 신규 알파버전이 생성되면 1차 QA를 진행하게 됩니다. 스타일쉐어는 **전담 QA담당자가 없습니다.** 1차 QA는 다른 파트 엔지니어 1명이 테스트를 진행하고 2차는 PM이 최종확인 후 릴리즈 됩니다. 이 단계에서는 Test case를 바탕으로한 명세기반 테스트로 진행됩니다.

![](/images/2015-04-16-qa-process/QA_sheet.png)

테스트 케이스(TC)를 통한 테스팅은 핵심적인 기능 및 주 사용케이스에 대한 검수작업이라고 보시면 됩니다. 게임 혹은 복잡도가 높은 서비스의 경우에는 매 업데이트마다 모든 케이스에 대한 테스트가 어렵고 비효율적이기 때문에 리스크 분석기법, 탐색적 테스팅, 경계값 테스팅 등과 같은 방식을 사용하지만 스타일쉐어 서비스의 경우 상대적으로 복잡도가 낮아 매 업데이트 마다 대부분의 기능에 대한 테스팅을 진행합니다(TC로 100% 커버리지를 목표로 하지 않습니다. 불가능하다는 것을 인정하고 진행하는 것이 효율적). 테스트케이스 작성시에 유의했던 부분은 **쉽고 명확하게** 케이스를 명시해서 오류에 대한 판단이 명확하도록 하고 스타일쉐어 앱을 처음 본 사람도 바로 테스트가 가능하도록 작성하고 있습니다.
(스트레스 테스트는 특이 사항이 있을 경우에만 진행합니다.)

###2) 교차 테스팅
스타일쉐어의 경우에는 1차QA 과정을 담당 엔지니어가 아닌 **다른 파트의 엔지니어**(iOS버전 테스트의 경우 web, backend, Android 개발자 중 1명이 진행)가 1차 테스트를 진행합니다. 이 방식의 장점은 매번 같은 사람이 테스트하는 것보다 다른 백그라운드를 가진 엔지니어가 테스트 함으로써 다양한 시각으로 테스트를 하게 되 오류발견이라던지 서비스 개선 아이디어를 찾는데 더 효과적이었습니다. 그리고 신규 입사자의 경우 가장 먼저 테스트 담당자로 참여할 수 있도록 합니다(가장 빠르게 서비스 플로우를 이해할 수 있는 방법).

###3) 최종확인
1차 QA 및 전사 베타버전 사용의 피드백을 통해 나온 버그/주요 기능에 대해 마지막 점검하는 절차입니다. 이 부분은 제품책임자(PM)가 담당을 하며, 이 부분을 통과하면 릴리즈 단계로 진행되어 사용자에게 업데이트 된 앱이 전달됩니다.

##3. 수많은 사용 패턴에 대한 대응

###단계적 출시(안드로이드)
1차 QA과정인 테스트케이스를 통한 테스팅은 명시되어 있는 패턴과 제한적인 환경(Device, 해상도, 인터넷 환경 등등)에서의 주요 케이스에 대한 테스팅만 가능합니다. 하지만 사용자는 수많은 환경 및 사용패턴으로 서비스를 사용하기 때문에 이 부분을 TC의 스크립트로 모두 추가하고 살펴보기란 불가능에 가깝습니다. 그래서 저희 팀은 [단계적 출시](https://support.google.com/googleplay/android-developer/answer/3131213?hl=ko)를 도입해서 대응하고 있습니다. 

![](/images/2015-04-16-qa-process/release_by_stages.png)

모든 테스트 과정을 완료한 뒤 구글플레이 개발자 콘솔에서 앱 업데이트시 ‘지금 출시’가 아닌 **‘단계적 출시’**로 선택합니다. 그리고 비율을 선택할 수 있는데 이 비율은 업데이트가 적용되는 사용자 비율을 설정하는 기능입니다. 즉, 전체 사용자가 아닌 미리 지정한 비율의 사용자에게만 업데이트 버전을 제공함으로써 우선적으로 우리가 예상하지 못한 버그나 불편한 부분이 있는지 확인해볼 수 있습니다. 스타일쉐어팀의 경우 5%의 사용자 비율로 단계적 출시를 1~2일 동안 진행한뒤 버그 리포팅 및 CS내용 확인 후 100% 대상으로 업데이트를 진행합니다.(5% 단계적 출시 이후 패치된 버전을 배포하면 해당그룹(5%)에게만 업데이트 됩니다.) 

이 부분은 오류에 대한 대응 및 새로운 기능에 대한 부분적인 반응을 볼 수 있는 용도로도 사용할 수 있어 매우 활용도가 높습니다.(신규 앱에 대해서는 해당 기능 사용이 불가능합니다. 업데이트시에만 사용가능합니다.)

##4. 도입효과


###1) Crash Free Sessions(Crashlytics)

![](/images/2015-04-16-qa-process/crash_report_ios.png)

4월 13일 기준으로 Crash Free Sessions는 전체 사용자 중 **99.8%**의 안정성을 가져가고 있으며(이전에는 95~96%), 기존에는 주말과 같이 사용자가 많은 경우 그만큼 크래시 발생빈도도 높았지만 최근 버전에서는 주말/평일 관계없는 그래프를 보이고 있습니다.

###2) Crash Report(Flurry)

![](/images/2015-04-16-qa-process/crash_report_android.png)

위 지표는 1월~3월 까지의 Flurry의 안드로이드 버전 Crash Report를 캡처한 화면입니다. 1월 초만 해도 일 40회 정도의 크래시가 발생했다면 최근은 **일 3~5회** 정도로 개선된 모습을 확인할 수 있습니다.

##5. 마무리

다만, 이러한 노력에도 버그는 여전히 존재합니다. 그래서 저희 QA프로세스도 개선할 방향을 모색하고 있는데, 현재의 개선 목표는 **‘퀄리티는 유지하되 속도는 빠르게’** 라는 방향으로 진행 중입니다. 그물을 더 촘촘히 짜듯이 명세기반 테스트의 규모를 늘리는 것에는 시간적/효율적인 한계가 분명히 존재하므로 자동화 테스팅(UI)의 강화를 통해서 부족한 부분을 채워보기 위한 시도를 준비하고 있습니다.

하루라도 빠른 서비스의 개선도 매우 중요하지만 그만큼 **우리가 전달하고자 하는 것을 문제없이 사용자에게 제공하는 것**도 속도만큼 중요하다 생각 합니다. 문제없이 전달하기 위해 계속해서 고민하고 시도해볼 수 있도록 하겠습니다.
