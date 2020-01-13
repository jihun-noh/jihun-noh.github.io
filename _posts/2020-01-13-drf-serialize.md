직렬화(Serializer)  
이제 DRF의 메인 기능인 직렬화를 해보자  

API 통신의 데이터 타입은 주로 JSON 으로 한다. 하지만 Django에서 Model 데이터는 보통 Queryset 이라는 복잡한 타입으로 되어있다. 이런 타입을 JSON으로 바꾸기 위해선, 파이썬 네이티브 타입으로 변환해주는 직렬화(Serializer) 란 작업이 필요하다  

Serializer 생성  
DRF의 Serializer는 Django의 Form과 사용방법이 거의 비슷하다  

models.py  

class Pet(models.Model):
  name = models.CharField(max_length=50)
  age = models.InteagerField()
위와 같은 Pet 모델을 직렬화 할 것이다

먼저 App 폴더 안에 serializers.py라는 파일을 만들어주자

serializers.py

from rest_framework import serializers

class PetSerializer(serializers.Serializer):
    name = serializers.CharField(max_length=50)
    age = serializers.IntegerField()
JSON 변환과정
이제 이 Serializer를 불러와서 JSON으로 반환해주자

views.py

from .serializers import PetSerializer

def getPet(request):
  pet = Pet.objects.get(pk=1)
  serializer = PetSerializer( pet )
_id가 1인 펫을 QuerySet 이라는 복잡한 Django type에서 Python의 Dictionary로 변환하였다

이제 이것을 전송하기 위해 문자열로 변환해주자

from .serializers import PetSerializer
from rest_framework.renderers import JSONRenderer
from django.http import HttpResponse

def getPet(request):
  pet = Pet.objects.get(pk=1)
  serializer = PetSerializer(pet)
  data = JSONRenderer().render(serializer.data)
  return HttpResponse(data)
Array 직렬화
위는 하나의 객체만 직렬화 하는 과정이다. 이번엔 여러개의 객체 인스턴스를 배열에 담아 직렬화 해보자

views.py

from .serializers import PetSerializer

def getPets(request):
  pets = Pet.objects.all()
  serializer = PetSerializer( pets, many=True )
many=True 만 같이 넣어주면 다수의 Pet 모델이 들어온 것을 인식한다

하지만 현재의 serializer.data는 이런식의 데이터이다

[ {'name': 'pet_name'},{ ...} ]
json은 배열이 아니라 object 형태로 작성해야하기 때문에 이 배열을 object 에 담아 문자열로 변환시키자

views.py

from .serializers import PetSerializer

import json

def getPets(request):
  pets = Pet.objects.all()
  serializer = PetSerializer( pets, many=True )
  data = json.dumps({'pets':serializer.data})
  return HttpResponse(data)
이번엔 내장모듈인 json을 활용해 인코딩하였다

이제 아래와 같은 문자열이 반환될 것이다

{ 'pets' : [ {...}, ... ] }
