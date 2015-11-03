---
layout: post
title: StyleShare 서비스의 구조
author: 김현준
author-email: khj@stylesha.re
description: 스타일쉐어의 엔지니어링 블로그의 첫 글에서는 저희 서비스의 스택을 소개하도록 하겠습니다.
publish: true
---

안녕하세요. 스타일쉐어에서 서버사이드 개발을 하고있는 김현준입니다. 스타일쉐어의 엔지니어링 블로그의 첫 글에서는 저희 서비스의 스택을 소개하도록 하겠습니다. 사실은 [Instagram의 스택](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances-dozens-of)과 유사한 면이 많아 글 또한 많이 유사할 것 같네요.

### 서버

먼저 스타일쉐어는 서버의 운영 체제로 [Ubuntu 12.04 (Precise Pengolin)](http://releases.ubuntu.com/12.04/)를 사용합니다. 모든 서버는 [아마존 웹 서비스(Amazon Web Services)](http://aws.amazon.com/)의 [Elastic Compute Cloud(EC2)](http://aws.amazon.com/ko/ec2) 위에서 돌아가고 있습니다. 스타일쉐어는 EC2 이외에도 [Simple Storage Service(S3)](http://aws.amazon.com/ko/s3/)와 같은 AWS의 다양한 서비스를 사용하고 있는데요, AWS를 사용하는 가장 큰 이유는 유연한 확장성(Scalability)이라 말할 수 있을 것 같습니다. EC2의 서버는 모두 가상 머신이기 때문에 관리 콘솔에서의 쉬운 조작으로 서버를 끄고 켤 수 있을 뿐만 아니라, 장애가 생겼을 때도 간편하게 장애가 생긴 서버를 내리고, 새로운 서버로 대체할 수 있는 이점이 있습니다. 이 모든 기능은 API로도 제공되고 있기 때문에, 자동화도 가능합니다. 실제로 스타일쉐어에서도 웹 요청을 처리하는 웹 서버들과 작업을 처리하는 워커들에 대해서 오토-스케일러를 구현해 사용하고 있습니다.

### 로드 밸런싱

스타일쉐어의 웹 서버들은 AWS의 [Elastic Load Balancing(ELB)](http://aws.amazon.com/elasticloadbalancing/)에 등록되어 있어서 ELB가 수많은 요청들을 여러 서버들에게 차례로 나누어 보냅니다. 보내어진 요청들은 각각의 서버에서 [nginx](http://nginx.org/)를 거치며 또 한번 여러 개의 프로세스로 분배되어 처리됩니다.

### 웹 어플리케이션

스타일쉐어의 웹 어플리케이션은 [Werkzeug](http://werkzeug.pocoo.org/) 기반의 웹 프레임워크 [Flask](http://flask.pocoo.org/)와 ORM 프레임워크인 [SQLAlchemy](http://www.sqlalchemy.org/) 위에서 [Python](http://www.python.org/)으로 구현되어 있습니다.

### 데이터

스타일쉐어의 대부분의 데이터는 [PostgreSQL](http://www.postgresql.org/)에 저장되고 있습니다. 여러 대의 PostgreSQL 인스턴스의 풀링(Pooling)을 하기 위해서 [pgpool](http://www.pgpool.net/)을 사용합니다. 서비스의 성능 향상을 위한 캐싱 도구로는 [Memcached](http://memcached.org/)를 사용합니다.

스타일쉐어에 올라오는 사진들을 비롯한 대부분의 이미지들은 Key 기반의 스토리지인 AWS S3에 저장하고, 관리합니다. S3의 가장 큰 장점은 사용자가 용량 제한과 파티셔닝에 대해 신경쓰지 않아도 된다는 점일 것입니다. 앞으로도 무한히 많은 사진이 올라올 서비스를 만드는 저희로서는 아주 유용하답니다. 이미지 뿐만 아니라, 서비스를 배포할 때마다 만드는 패키지와 매일매일 데이터베이스 백업 모두 S3에 저장되어 있습니다.

### 작업 관리

대부분의 서비스와 마찬가지로, 스타일쉐어도 웹 어플리케이션 서버와 별개로 무거운 작업(Task)을 처리하기 위한 워커(Worker) 서버를 따로 구동하고 있습니다. 여기서 작업이란 계속해서 쏟아지는 웹 요청을 처리하기도 벅찬 웹 어플리케이션에서 처리하기에는 비교적 오래걸리는, 예를 들면 알림(푸시)과 메일을 보내거나, 이미지 프로세싱과 같은 일들을 이야기합니다. 이러한 작업들을 비동기적으로 처리하기 위해 저희는 [Celery](http://celeryproject.org/)와 [RabbitMQ](http://www.rabbitmq.com/)를 사용합니다. Celery는 Python으로 구현된 비동기 작업 워커이고, RabbitMQ는 워커로 넘길 작업을 관리하는 [AMQP 프로토콜](http://www.amqp.org/) 기반의 브로커(Broker) 큐입니다.

### 오픈 소스?

스타일쉐어 서버는 비동기 네트웍(asynchronous I/O)을 구현하기 위해서 [gevent](http://www.gevent.org/)를 사용합니다. 그 외에 배포(deploy)를 위한 [Fabric](http://www.fabfile.org/)과 [boto](https://github.com/boto/boto/)나, 내부 문서화를 위해 사용하는 [Sphinx](http://sphinx.pocoo.org/) 등이 스타일쉐어에서 주로 사용하는 라이브러리/프로젝트 입니다.

### 오픈 소스.

위에 적은 것처럼, 스타일쉐어의 구현의 많은 부분이 오픈 소스 프로젝트에 크게 의존하고 있습니다. 훌륭하고 건강한 오픈 소스 생태계 덕분에 우리는 스타일쉐어를 훨씬 더 수월하게 만들고 지탱할 수 있었습니다. 그래서 저희도 도움을 받은 만큼 기여하고, 구성원으로서 더 나은 생태계를 만드려 합니다. 그 중 하나가 바로 이 스타일쉐어 엔지니어링 블로깅 활동이고, 다른 하나가 저희 팀의 오픈 소스 프로젝트 활동입니다. 스타일쉐어 팀의 오픈 소스 활동들은 [StyleShare’s GitHub](https://github.com/StyleShare)에서 살펴보실 수 있답니다. 여러분들의 관심어린 피드백과 기여도 언제나 감사히 환영합니다.

### 그 외의 도구들

스타일쉐어 실 서비스에서 발생하는 오류와 버그를 추적하기 위해 사용하는 [Exceptional](http://www.exceptional.io/)도 매우 유용합니다. Flask 프레임워크에서 Exceptional 서비스를 쉽게 이용할 수 있도록 도와주는 Flask 확장 모듈인 [Flask-Exceptional](https://github.com/jzempel/flask-exceptional/)이 공개되어 있습니다.

### 함께해요

저희와 비슷한 환경에서 개발하시는, 같은 도구를 사용하시는, 저희에게 도움을 주고 싶으시거나, 저희에게 (저희가 도와드릴 수 있다면) 도움을 받고 싶으신, 또는 그저 많은 이야기를 나누고 싶은 분들까지 많은 분들과의 소통과 교류가 많았으면 좋겠습니다. IRC를 하시는 분들은 오징어 네트워크(irc.ozinger.org)의 #styleshare-tech 채널로 놀러오세요.

