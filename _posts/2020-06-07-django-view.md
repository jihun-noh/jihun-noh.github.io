---
layout: post
title: django view 작성
category: Python
tags: [pyhthon, django]
---

1. Function based View   
from rest_framework.decorators import api_view   
@api_view(['GET'])   
def dive_point(request):   
    if request.method == 'GET':   
        queryset = DivePoint.objects.all()   
        serializer = DivePointSerializer(queryset, many=True)   
        return Response(serializer.data)      
path('fbv/', views.dive_point)


2. Class Based View
from rest_framework.views import APIView
class DivePointView(APIView):
    def get(self, request):
        queryset = DivePoint.objects.all()
        serializer = DivePointSerializer(queryset, many=True)
        return Response(serializer.data)
        
path('cbv/', views.DivePointView.as_view()),


3. ViewSets
class DivePointViewSet(viewsets.ModelViewSet):
    serializer_class = DivePointSerializer
    queryset = DivePoint.objects.all()

point_list = DivePointViewSet.as_view({
    'get': 'list',
    'post': 'create',
    })
    
router = routers.DefaultRouter()
router.register(r'divepoints', views.DivePointViewSet)
