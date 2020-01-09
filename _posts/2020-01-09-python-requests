---
layout: post
title: requests
category: Python
tags: [pyhthon]
---
설치
pip install requests

사용법
import requests

get, post, put, delete, head, options
>>> r = requests.get('https://api.github.com/events')
>>> r = requests.post('http://httpbin.org/post', data = {'key':'value'})
>>> r = requests.put('http://httpbin.org/put', data = {'key':'value'})
>>> r = requests.delete('http://httpbin.org/delete')
>>> r = requests.head('http://httpbin.org/get')
>>> r = requests.options('http://httpbin.org/get')

URL에 파라미터 넘기기
예1)
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get('http://httpbin.org/get', params=payload)
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
예2)
>>> payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
>>> r = requests.get('http://httpbin.org/get', params=payload)
>>> print(r.url)
http://httpbin.org/get?key1=value1&key2=value2&key2=value3

Response Content
>>> r = requests.get('https://api.github.com/events')
>>> r.text # 텍스트 형식으로 내용 출력
>>> r.encoding # 인코딩 확인
'utf-8'
>>> r.encoding = 'ISO-8859-1' # 인코딩

Binary Resopnse Content
r.content # 바이너리 형식으로 접근 (gzip과 deflate는 자동 디코딩 됨)
>>> from PIL import Image
>>> from StringIO import StringIO
>>> i = Image.open(StringIO(r.content)) # 받은 바이너리 데이터를 이미지로 만들기

JSON Response Content
>>> r.json()

Raw Response content
>>> r = requests.get('https://api.github.com/events', stream=True)
>>> r.raw

>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
with open(filename, 'wb') as fd:
for chunk in r.iter_content(chunk_size):
fd.write(chunk)

Custom Headers
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}
>>> r = requests.get(url, headers=headers)

더 복잡한 POST
예1) 딕셔너리 형식
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.post("http://httpbin.org/post", data=payload)
>>> print(r.text)
예2) json 형식
>>> import json
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> r = requests.post(url, data=json.dumps(payload))
예3) json 파라미터
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> r = requests.post(url, json=payload)

POST a Multipart-Encoded File
예1) 일반
>>> url = 'http://httpbin.org/post'
>>> files = {'file': open('report.xls', 'rb')}
>>> r = requests.post(url, files=files)
>>> r.text
예2) 타입 지정
>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}
>>> r = requests.post(url, files=files)
>>> r.text
예3) 문자열을 파일로 보냄
>>> url = 'http://httpbin.org/post'
>>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}
>>> r = requests.post(url, files=files)
>>> r.text

multipart/form-data 로 매우 큰 파일을 보는 것은 requests에서 지원하지 않는다.
그럴 때는 requests-toolbelt 별도 패키지를 이용하라.링크

경고:
파일을 열 때는 바이너리 모드로 여는 것을 적극 권장한다.
왜냐하면 Requests는 Content-Length 헤더를 붙이기 때문이다.
만약 붙게 된다면 파일에 바이트 수가 설정되고 만약 텍스트 모드로 파일을 연다면 오류가 발생할 수 있다.
바이너리를 utf-8로 바꾸는 방법 : decode('utf-8')

Response Status Codes
>>> r = requests.get('http://httpbin.org/get')
>>> r.status_code
200
>>> r.status_code == requests.codes.ok # 쉬운 참조를 위한 방법
True
4XX client error 또는 5XX server error response 일 때는 Response.raise_for_status() 예외를 일으킬 수 있다.
>>> bad_r = requests.get('http://httpbin.org/status/404')
>>> bad_r.status_code
404
>>> bad_r.raise_for_status()
>>> r.raise_for_status() # status_code가 200이라면 None을 받게 된다.
None

Response 헤더
>>> r.headers
{
'content-encoding': 'gzip',
'transfer-encoding': 'chunked',
'connection': 'close',
'server': 'nginx/1.0.4',
'x-runtime': '148ms',
'etag': '"e1ca502697e5c9317743dc078f67693f"',
'content-type': 'application/json'
}
RFC 7230 에 따르면 HTTP 헤더는 대소문자를 구분한다.
>>> r.headers['Content-Type'] # 하지만 어느걸 써도 접근이 가능하다.
'application/json'
>>> r.headers.get('content-type')
'application/json'

Cookies
>>> url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)
>>> r.cookies['example_cookie_name']
'example_cookie_value'
자신의 쿠키를 보내기
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')
>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'

Redirection and History
>>> r = requests.get('http://github.com')
>>> r.url
'https://github.com/'
>>> r.status_code
200
>>> r.history
[]

GET, OPTIONS, POST, PUT, PATCH, DELETE 를 사용한다면 allow_redirects 파라미터로 redirection 해제 가능
>>> r = requests.get('http://github.com', allow_redirects=False)
>>> r.status_code
301
>>> r.history
[]

HEAD를 사용한다면 사용 가능
>>> r = requests.head('http://github.com', allow_redirects=True)
>>> r.url
'https://github.com/'
>>> r.history
[]

Timeouts
지정된 시간 이후에 기다리는 걸 멈추라고 Requests에 전달 가능 (이 때 예외 발생)
>>> requests.get('http://github.com', timeout=0.001)

에러와 예외
1. DNS 실패, 거절된 연결 등의 네트워크 문제시 ConnectionError 예외 발생
2. 흔치 않지만 잘못된 HTTP response 문제시 HTTPError 예외 발생
3. request가 타임아웃 되면 Timeout 예외 발생
4. request가 최대 리다이렉션을 초과한다면 TooManyredirects 예외 발생
5. 모든 예외는 requests.exceptions.RequestException 에서 상속됨.

세션 유지
s = requests.Session()
headers = {"User-Agent":"Mozilla/5.0", "Referer":None} # 빼려면 None
headers.update(self.headers)
d = self.s.post(url, data)
d.status_code
c = d.content

정리
d.status_code
d.request.headers # request header
d.headers # response header
d.content # byte
d.text # text
headers.update({}) # add header
