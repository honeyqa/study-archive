# RabbitMQ

표준 [AMQP](http://www.amqp.org/) 메시지 브로커 소프트웨어

## 설치

* 환경
	* Mac OS X 10.10.4 (Yosemite)
* 다운로드
	* [Mac Standalone @ 공식 홈페이지](https://www.rabbitmq.com/install-standalone-mac.html)
	* 설치 후
		* Web Console 활성화
			* `sbin/rabbitmq-plugins enable rabbitmq_management`
		* RabbitMQ 실행
			* `sbin/rabbitmq-server` 
		* Web Console 접속
			* [http://localhost:15672](http://localhost:15672/)
			* username : **guest**
			* password : **guest**

## 튜토리얼

* [Rabbit MQ 공식 홈페이지 Tutorial - Javascript](https://www.rabbitmq.com/tutorials/tutorial-one-javascript.html)
* [honeyqa/rabbitmq-study](https://github.com/honeyqa/rabbitmq-study/tree/master/tutorial_nodejs)

## 샘플 프로젝트

REST API 서버를 이용해 간단하게 요청을 받으면 RabbitMQ에 데이터를 넣은 후, Worker를 이용해 MySQL 서버에 데이터를 넣어주는 샘플 프로젝트를 진행했다.

### Server

---

[Source](https://github.com/honeyqa/rabbitmq-study/tree/master/nodejs-rest)

* Express를 이용해 간단한 REST API Server를 구축했다.
* 서버는 다음과 같이 동작한다.
	* 먼저 RabbitMQ에 연결한 뒤,
		* `amqp.connect('amqp://guest:guest@localhost:5672', function(err, conn) {`
	* 정상적으로 연결되면 Channel을 생성한다.
		* `conn.createChannel(function(err, ch) {`
	* 정상적으로 Channel이 생성되면 서버가 실행되고, `localhost:8080/queue/{message}`에 POST Request를 보내면 `expressTest`라는 Queue에 메시지를 전송한다.

### Worker

---

[Source](https://github.com/honeyqa/rabbitmq-study/tree/master/nodejs-worker)

* Worker는 다음과 같이 동작한다.
	* 서버와 마찬가지로, 먼저 RabbitMQ에 연결한 뒤,
		* `amqp.connect('amqp://guest:guest@localhost:5672', function(err, conn) {`
	* Channel을 생성한 뒤, Database에 연결하고, 정상적으로 연결되면  Queue에 연결해서 메시지를 받아온다.
	* 메시지를 받게되면, Database에 받아온 메시지를 Insert하고, Queue에 메시지가 올때까지 기다린다.

## Problem