---
layout: post
title: "Circle CI에서 rbenv를 이용해서 Ruby 2.2와 CocoaPods 0.39 버전 사용하기"
author: 전수열
author-email: jeon@stylesha.re
description: 최근 Circle CI에서 Ruby 버전을 2.3으로, CocoaPods 버전을 1.0으로 업그레이드함에 따라 발생하는 빌드 문제를 rbenv를 이용해서 해결한 경험을 공유합니다.
publish: true
---

최근 Circle CI에서 Ruby 버전을 2.3으로, CocoaPods 버전을 1.0으로 업그레이드함에 따라 발생하는 빌드 문제를 rbenv를 이용해서 해결한 경험을 공유합니다. 최종적으로 완성된 **`Gemfile`**과 **`circle.yml`** 파일은 [마지막 섹션](#conclusion)에서 확인하실 수 있습니다.

## 1. CocoaPods 1.0

지난 2015년 12월에 CocoaPods 1.0.0 베타 버전이 처음 공개되었습니다. CocoaPods이 1.0 버전으로 업그레이드되면서 [굉장히 많은 변화](http://blog.cocoapods.org/CocoaPods-1.0/)가 있었는데요. 가장 큰 변화는 DSL입니다. 추상 타겟<sup>Abstract Target</sup>과 타겟 상속<sup>Target Inheritance</sup>이 새롭게 소개되면서, 0.39 버전까지 자주 사용되던 `link_with` 및 `:exclusive => true`와 같은 구문이 제거되었습니다.

이에 따라 기존에 사용하던 **`Podfile`**이 CocoaPods 1.0 버전과는 호환되지 않는 문제가 발생했습니다. 이를 해결하기 위한 가장 좋은 방법은 새로운 DSL을 사용하여 **`Podfile`**을 다시 작성하는 것이지만, 꽤 많은 서드파티 라이브러리를 사용하는 StyleShare의 경우 새로운 DSL을 적용하여 빌드하면 각종 문제로 인해 빌드가 정상적으로 이루어지지 않았습니다. 4년동안 유지되고 있는 프로젝트이다보니, 레거시 Objective-C 코드와 라이브러리, 그리고 새로운 Swift 코드와 라이브러리가 혼용되어 사용되는 것도 원인 중 하나일 것입니다.

따라서 StyleShare에서는 CocoaPods 0.39 버전을 사용하기로 결정을 했습니다. 하지만 최근 Circle CI에서 CocoaPods 버전을 공식적으로 1.0 버전으로 업그레이드하면서 빌드가 깨지기 시작했습니다. Circle CI 환경에서 CocoaPods 0.39 버전을 사용하려면 어떻게 해야 할까요?

<figure>
  <img src="/images/2016-08-02-circleci-ios-rbenv/circleci-build-fail.png">
  <figcaption>
  &#9650; ㅠㅠ
  </figcaption>
</figure>

## 2. Bundler를 이용해서 Gem 관리하기

[Bundler](http://bundler.io)는 Ruby로 작성된 라이브러리들의 버전을 관리해주는 강력한 도구입니다. CocoaPods에서 **`Podfile`**에 의존성을 기재하듯, Bundler에서는 **`Gemfile`**에 의존성을 기재합니다.

```ruby
source 'https://rubygems.org'

gem 'cocoapods', '~> 0.39'
```

`$ gem install bundler` 명령어를 사용하면 **`Gemfile`**에 기재된 의존성 라이브러리들을 설치해줍니다. 이렇게 설치된 CocoaPods을 사용할 때에는 `$ pod COMMAND` 대신 `$ bundle exec pod COMMAND` 명령어를 사용해야 합니다.

```console
$ gem install bundler
$ bundle install --path vendor/bundle
$ bundle exec pod --version
0.39.0
```

## 3. Ruby 2.3과 CocoaPods 0.39

Bundler를 사용해서 CocoaPods 0.39 버전을 사용하기만 하면 모든 문제가 해결될 줄 알았습니다. 하지만 더 큰 삽질이 남아있었는데요. 바로 Ruby 2.3 버전이 CocoaPods 0.39 버전과 호환되지 않는 것이었습니다.

```console
$ bundle exec pod install
Updating local specs repositories
Analyzing dependencies
```

신나게 `$ bundle exec pod install` 명령어를 실행하니, 의존성을 분석하는 듯 싶다가 갑자기 에러를 주르륵 뱉습니다. 에러 로그의 `#### Error` 항목을 보면 에러 메시지가 나와있습니다.

> NoMethodError - undefined method `to_ary' for #\<Pod::Specification name="...">

이 에러 메시지로 CocoaPods GitHub 저장소의 [이슈를 검색](https://github.com/CocoaPods/CocoaPods/issues?utf8=%E2%9C%93&q=undefined%20method%20to_ary)해보면 꽤나 많은 이슈가 올라와 있습니다. 이 이슈들을 보면, 모두 Ruby 버전이 2.3이라는 공통점이 있습니다. Ruby 버전을 2.2로 내렸더니 문제가 해결됐다는 댓글들도 굉장히 많고요. Circle CI의 Ruby 버전을 2.2로 낮추면 문제가 해결될 것 같습니다.

[Circle CI 문서](https://circleci.com/docs/configuration/#ruby-version) 내용에 따라 **`circle.yml`**에 Ruby 버전을 기재해봅시다.

```yaml
machine:
  ruby:
    version: 2.2.5
```

그러나 Circle CI의 OS X 컨테이너에서는 Ruby 버전 변경을 지원하지 않는다고 합니다.

<figure>
  <img src="/images/2016-08-02-circleci-ios-rbenv/circle-ci-ruby-version-not-support.png">
  <figcaption>
  &#9650; ㅠㅠ (2)
  </figcaption>
</figure>

## 4. rbenv를 이용해서 Ruby 2.2 사용하기

그러다가 알게된 것이 바로 [rbenv](https://github.com/rbenv/rbenv)입니다. rbenv를 사용하면 여러개의 Ruby 버전을 깔끔하게 관리할 수 있게 됩니다. rbenv는 [Homebrew](http://brew.sh)를 사용해서 쉽게 설치할 수 있습니다.

```console
$ brew install rbenv
```

rbenv는 `~/.rbenv` 디렉토리에 안에 여러 Ruby 버전을 설치하고 관리합니다. rbenv를 설치한 뒤 가장 먼저 할 일은 환경변수 `$PATH`를 설정해주는 것입니다. `$PATH`에는 `$HOME/.rbenv/shims`와 `$HOME/.rbenv/bin` 경로가 포함되어있어야 합니다.

#### 4.1 환경변수 설정하기

Circle CI에서는 환경변수를 설정하는 [편리한 인터페이스](https://circleci.com/docs/configuration/#environment)를 제공합니다. 하지만, Circle CI에서 실행되는 각 명령어는 별도의 쉘에서 실행됩니다. 그말인 즉슨, 각 명령어가 실행되기 직전에 새로운 쉘이 실행되고, `$PATH` 환경변수를 덮어쓰는 **`.bash_profile`**이 실행된 후 명령어가 실행된다는 뜻인데요. 이렇게 될 경우 `$PATH` 환경변수의 가장 우선순위는 항상 `/usr/local/bin`이 가지게 됩니다. 그리고 같은 이유로 `$ export FOO=bar`와 같은 명령어도 사용할 수 없게 됩니다.[^1]

고민을 하다가 생각해낸 방법은 바로 **`.bash_profile`**의 내용을 변경(!)하는 것입니다. 그렇게 되면 우리가 원하는 `$PATH`를 항상 우선순위로 둘 수 있게 됩니다. 아래와 같이 환경변수를 설정하는 명령어를 **`.bash_profile`**의 가장 아랫줄에 삽입하도록 설정했습니다.

```yaml
machine:
  pre:
    - echo "export PATH=\$HOME/.rbenv/shims:\$HOME/.rbenv/bin:\$PATH" >> .bash_profile
    - echo "export RBENV_SHELL=bash" >> .bash_profile
```

#### 4.2 rbenv에 Ruby 2.2 설치하기

그 다음으로 할 일은 원하는 Ruby 2.2 버전을 설치하는 것입니다. `$ rbenv install -l`을 사용해서 설치 가능한 모든 Ruby 버전을 조회할 수 있고, `$ rbenv install 2.2.5` 명령어를 사용해서 2.2.5 버전을 설치할 수 있습니다.

```console
$ rbenv install -l
Available versions:
  1.8.5-p113
  1.8.5-p114
  1.8.5-p115
  1.8.5-p231
  ...
$ rbenv install 2.2.5
```

이렇게 설치된 버전은 두 가지 방법으로 사용될 수 있습니다. 한 가지 방법은 시스템 전체에서 사용하는 것이고, 다른 한 가지 방법은 프로젝트 단위로 사용하는 방법입니다. 시스템 전체에서 사용하려면 `$ rbenv global 2.2.5` 명령어를, 프로젝트 단위로 사용하려면 `$ rbenv local 2.2.5` 명령어를 사용합니다.

`global` 명령어를 사용해서 Ruby 버전을 선택하면 `~/.rbenv/version` 파일에 선택된 버전이 기록됩니다.

```console
$ rbenv global 2.2.5
$ cat ~/.rbenv/version
2.2.5
```

`local` 명령어를 사용하면 현재 디렉토리의 `.ruby-version` 파일에 선택된 버전이 기록됩니다.

```console
$ rbenv local 2.2.5
$ cat .ruby-version
2.2.5
```

`local` 명령어로 선택된 Ruby 버전은 `global` 명령어로 선택된 Ruby 버전보다 우선순위가 높습니다. `$ rbenv version` 명령어를 사용하면 현재 선택된 버전을 확인할 수 있습니다.

```console
$ rbenv version
2.2.5 (set by /project/path/.ruby-version)
```

Circle CI에서는 편의를 위해 `global` 명령어를 사용해서 Ruby 버전을 선택하도록 했습니다.

```yaml
dependencies:
  pre:
    - brew update
    - brew install rbenv
    - rbenv install 2.2.5
    - rbenv global 2.2.5
```

#### 4.3 Bundler 다시 설치하기

rbenv를 사용해서 새로운 Ruby 버전을 설치했기 때문에, Circle CI 시스템에서 제공하는 Gem도 다시 설치해야 합니다. 우리는 Bundler로 Gem 의존성을 관리하기로 했으므로, Bundler만 재설치합니다.

```console
$ gem install bundler --no-ri --no-rdoc
$ rbenv rehash
```

`$ gem install` 명령어를 실행한 후에는 `$ rbenv rehash` 명령어를 실행해서 executable 경로들을 재설정해주어야 합니다.

#### 4.4 **`~/.rbenv`** 경로 캐싱하기

rbenv를 사용해서 Ruby를 설치하는 과정이 굉장히 오래 걸립니다. 이 경우, Circle CI에서 제공하는 캐싱 기능을 사용해서 이 과정을 한 번만 하고 건너뛸수 있게 됩니다.

```yaml
dependencies:
  cache_directories:
    - ~/.rbenv
```

위와 같이 **`circle.yml`**를 설정해주면 컨테이너 실행시 **`~/.rbenv`** 디렉토리가 캐시로부터 설정됩니다. 캐싱된 디렉토리를 사용하는 경우 Ruby 버전이 미리 설치되어있기 때문에 `$ rbenv install`시에 `--skip-existing` 옵션을 추가해주어서 캐싱된 버전을 재설치하지 않도록 합니다.

<h2 id="conclusion">5. 마치며</h2>

최종적으로 완성된 **`Gemfile`**과 **`circle.yml`** 파일은 다음과 같습니다.

**Gemfile**

```ruby
source 'https://rubygems.org'

gem 'cocoapods', '~> 0.39'
```

**circle.yml**

```yaml
machine:
  pre:
    - echo "export PATH=\$HOME/.rbenv/shims:\$HOME/.rbenv/bin:\$PATH" >> .bash_profile
    - echo "export RBENV_SHELL=bash" >> .bash_profile
  xcode:
    version: 7.3

dependencies:
  cache_directories:
    - ~/.rbenv
  pre:
    - brew update
    - brew install rbenv
    - rbenv install 2.2.5 --skip-existing
    - rbenv global 2.2.5
    - gem install bundler --no-ri --no-rdoc
    - rbenv rehash
    - bundle install --path vendor/bundle
  override:
    - bundle exec pod --version
    - bundle exec pod install
```

[^1]: [https://circleci.com/docs/environment-variables/#custom](https://circleci.com/docs/environment-variables/#custom)
