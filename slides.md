---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: 코드로 서버리스 API 인프라 구축하기
info: |
  Python 함수는 만들 수 있는데, 이것을 어떻게 배포하고 사용할 수 있을까요?
  이 발표에서는 AWS Lambda와 API Gateway를 사용하여 서버리스 API 인프라를 구축하는 방법을 알아봅니다.
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: fade-out
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

## 코드로 서버리스 API 인프라 구축하기

with Serverless Framework


---
layout: image-left
image: /me.jpg
backgroundSize: contain
---

<br>
<br>
<br>
<br>
<br>
<br>

# 안녕하세요! 천명욱입니다.


---

# 목차

<Toc minDepth="1" maxDepth="1" columns="2">
</Toc>


---

# 멋진 함수를 만들었는데...

<br>
<div v-click>
```python
# prime.py
def is_prime(number):
    if number <= 1:
        return False

    if number <= 3:
        return True

    if number % 2 == 0 or number % 3 == 0:
        return False

    i = 5
    while i * i <= number:
        if number % i == 0 or number % (i + 2) == 0:
            return False
        i += 6

    return True
```
</div>


---
layout: center
---

# 다른 사람이 사용했으면 좋겠다!

다른 사람이 사용할 수 있도록 배포하고 싶다.
web API로 만들어서 공유하고 싶다.


---

# API 는 어떻게 만들지?
<br>
<h3 v-click> Django? </h3>
<br>
<h3 v-click> Flask? </h3>
<br>
<h3 v-click> FastAPI? </h3>
<br>
<h3 v-click>새로운 프레임워크를 배워야하네... </h3>


---

# 배포도 해야 한다고? 서버를 임대해야 해?
<br>
<h3 v-click>AWS EC2 인스턴스 만들고</h3>
<br>
<h3 v-click>SSH 로 접속해서 소스 코드 받고</h3>
<br>
<h3 v-click>nginx 설치 하고 WSGI 연결해주고</h3>
<br>
<h3 v-click>계속 켜두면 비용 청구되니 다 쓰고 EC2 꺼줘야하고..</h3>


---
layout: center
---

# 나는 그냥 내가 만든 함수 하나 알리고 싶은건데... 


---
layout: center
---

# 좋은 방법 없을까?


---
layout: image-right
image: /Amazon_Lambda_architecture_logo.png
---

# AWS Lambda 를 써보자
<br>

### 장점
<v-clicks>

- 간단한 함수에 적합하다.
- 프레임워크를 배우지 않아도 된다.
- 호출 한 만큼만 비용이 청구된다.

</v-clicks>


---
layout: image-right
image: /create-lambda.png
backgroundSize: contain
---

# 아직도 뭔가 번거롭다...
<br>

<v-clicks>

- 초기 설정도 많고 콘솔에서 하나하나 입력하기 너무 귀찮다.
- 매번 코드 업로드 해야 하는 것도 번거롭다.
- api gateway 설정도 해야하고...

</v-clicks>


---

# 만약
<br>

<v-clicks>

- 설정 파일 하나로 필요한 모든 인프라를 생성할 수 있다면?
- 코드를 업로드 하지 않아도 자동으로 배포된다면?
- 모든 인프라를 다시 명령어 한번으로 삭제할 수 있다면?

</v-clicks>


---
layout: image
image: /social-card-serverless-framework.png
backgroundSize: contain
---


---

# Serverless Framework 설치
<br>

```bash
npm install serverless -g
```

---
layout: image-right
image: /create-project-4.png
backgroundSize: contain
transition: slide-up
---

# 새로운 프로젝트 만들기
<br>

```bash
serverless
```


---

> 주의: 적절한 권한을 가진 access key 설정이 필요.

![Local Image](/create-project-3.png)


---

# 디렉터리 구조
<br>

````md magic-move
```bash
math $ tree
.
├── README.md
├── handler.py
└── serverless.yml
```
```bash
math $ tree
.
├── README.md
├── handler.py
├── prime.py
└── serverless.yml
```
````


---
transition: slide-up
---

# handler.py
<br>

````md magic-move
```python {*|4|*}
import json


def hello(event, context):
    body = {
        "message": "Go Serverless v3.0! Your function executed successfully!",
        "input": event,
    }

    response = {"statusCode": 200, "body": json.dumps(body)}

    return response
```
```python
import json


def handle(event, context):
    body = {
        "message": "Go Serverless v3.0! Your function executed successfully!",
        "input": event,
    }

    response = {"statusCode": 200, "body": json.dumps(body)}

    return response
```
```python
import json

from prime import is_prime


def handle(event, context):
    number = int(event["queryStringParameters"]["number"])
    
    if is_prime(number):
        return f"{number} is prime!"

    return f"{number} is not prime!"
```
````


---

# serverless.yml
<br>

````md magic-move
```yaml {*|9|10|11|12-14|*}
service: math
frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9

functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /
          method: get
```
```yaml
service: math
frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9
  region: ap-northeast-2

functions:
  hello:
    handler: handler.hello
    events:
      - httpApi:
          path: /
          method: get
```
```yaml
service: math
frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9
  region: ap-northeast-2

functions:
  prime:
    handler: handler.handle
    events:
      - httpApi:
          path: /
          method: get
```
```yaml
service: math
frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9
  region: ap-northeast-2

functions:
  prime:
    handler: handler.handle
    events:
      - httpApi:
          path: /
          method: get
```
````

---

# 배포하기
<br>

```bash
serverless deploy
```
<v-clicks>
    <img src="/deploy.png" alt="deploy" />
</v-clicks>


---

# 배포된 API 확인하기
<br>

<v-clicks>
    <img src="/test.png" alt="test" />
</v-clicks>


---
layout: center
---

# 맛보기 후에는 더 깊이 알아보기!


---
layout: center
---

# Q & A

<v-clicks>

- CloudFormation 으로 하면 되는 거 아닌가요?
- 현업에서도 사용하나요?
- 비용은 어떻게 되나요?
- apig 3.5$ per million requests
- lambda 0.2$ per million requests

</v-clicks>




---
layout: center
class: text-center
---

# 감사합니다

