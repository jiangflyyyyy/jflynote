# Web开发

## Django-rest-framework

### restful规范(10条)

 1. 数据的安全保障:url链接一般采用https协议进行传输,注:采用https协议,可以提高数据交互过程中的安全性
 2. 接口特征表现
    - 用api关键标识接口url
        - https://api.baidu.com
        - https://www.baidu.com/api
    看到api字眼就代表该请求url链接是完成前后台数据交互的
 3. 多数据版本共存
    - 在url链接中标识数据版本
        - https://api.baidu.com/v1
        - https://api.baidu.com/v2
    url链接中的v1、v2就是不同数据版本的体现(只有在一种数据资源有多版本情况下)
 4. 数据即资源,均使用名词(可使用复数)
 5. 资源操作由请求方式决定(method)
 6. 过滤,通过在url传参的形式传递搜索条件
 7. 响应状态码
 8. 错误处理,应返回错误信息,erro当做key
 9. 返回结果,针对不同操作,服务器向用户返回的结果应该符合以下规范
    - GET /collection:返回对象资源的列表(数组)
    - GET /collection/resource:返回单个资源对象
    - POST /collection:返回新生成的资源对象
    - PUT /collection/resource:返回完整的资源对象
    - PATCH /collection/resource:返回完整的资源对象
    - DELETE /collection/resource:返回一个空文档
 10. 需要url请求的资源需要访问资源的请求链接

----------
### 安装与使用 ##

 1. 安装: 
    pip install djangorestframework
 2. 使用:
    需要在settings.py中的INSTALLED_APPS中注册
```python
INSTALLED_APPS = [
        'rest_framework',
    ]
```
#### models.py
```python
from django.db import models

class Book(models.Model):
    name = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    publish = models.CharField(max_length=32)
```
#### ser.py
```python
from rest_framework import serializers
from app01.models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = '__all__'
```
###基于APIView
#### views.py
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from app01.models import Book
from app01.ser import BookSerializer

class BooksView(APIView):
    def get(self, request):
        book = Book.objects.all()
        book_ser = BookSerializer(book, many=True)
        return Response(book_ser.data)

    def post(self, request):
        book_ser = BookSerializer(data=request.data)
        if book_ser.is_valid():
            book_ser.save()
        return Response(book_ser.data)
        
class BookView(APIView):
    def get(self, request, pk):
        book = Book.objects.filter(pk=pk).first()
        book_ser = BookSerializer(book)
        return Response(book_ser.data)

    def put(self, request, pk):
        book = Book.objects.filter(pk=pk).first()
        book_ser = BookSerializer(book, request.data)
        if book_ser.is_valid():
            book_ser.save()
        return Response(book_ser.data)

    def delete(self, request, pk):
        Book.objects.filter(pk=pk).delete()
        return Response({'msg': '删除成功'})
```
#### urls.py
```python
urlpatterns = [
    # 基于APIView
    path('books/', BooksView.as_view()),
    re_path('book/(?P<pk>\d+)', BookView.as_view()),
]
```
### 基于GenericAPIView
#### views.py
```python
from rest_framework.generics import GenericAPIView
from rest_framework.response import Response
from app01.models import Book
from app01.ser import BookSerializer

class Books2View(GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request):
        book = self.get_queryset()
        book_ser = self.get_serializer(book, many=True)
        return Response(book_ser.data)

    def post(self, request):
        book_ser = self.get_serializer(data=request.data)
        if book_ser.is_valid():
            book_ser.save()
        return Response(book_ser.data)


class Book2View(GenericAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request, pk):
        book = self.get_object()
        book_ser = self.get_serializer(book)
        return Response(book_ser.data)

    def put(self, request, pk):
        book = self.get_object()
        book_ser = self.get_serializer(book, request.data)
        if book_ser.is_valid():
            book_ser.save()
        return Response(book_ser.data)

    def delete(self, request, pk):
        self.get_object().delete()
        return Response({'msg': '删除成功'})
```
#### urls.py
```python
urlpatterns = [
    # 基于GenericAPIView
    path('books2/', Books2View.as_view()),
    re_path('book2/(?P<pk>\d+)', Book2View.as_view()),
]
```
### 基于GenericAPIView的5个视图扩展类
#### views.py
```python
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin
from app01.models import Book
from app01.ser import BookSerializer

class Books3View(GenericAPIView, ListModelMixin, CreateModelMixin):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request):
        return self.list(request)

    def post(self, request):
        return self.create(request)


class Book3View(GenericAPIView, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get(self, request, pk):
        return self.retrieve(request, pk)

    def put(self, request, pk):
        return self.update(request, pk)

    def delete(self, request, pk):
        return self.destroy(request, pk)
```
#### urls.py
```python
urlpatterns = [
    # 基于GenericAPIView的5个视图扩展类
    path('books3/', Books3View.as_view()),
    re_path('book3/(?P<pk>\d+)', Book3View.as_view()),
]
```
### 基于GenericAPIView的9个视图子类
#### views.py
```python
from rest_framework.generics import ListAPIView, CreateAPIView, RetrieveAPIView, UpdateAPIView, \
    DestroyAPIView, ListCreateAPIView, RetrieveDestroyAPIView, RetrieveUpdateAPIView, \
    RetrieveUpdateDestroyAPIView
from app01.models import Book
from app01.ser import BookSerializer

class Books4View(ListCreateAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer


class Book4View(RetrieveUpdateDestroyAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```
#### urls.py
```python
urlpatterns = [
    # 基于GenericAPIView的9个视图子类
    path('books4/', Books4View.as_view()),
    re_path('book4/(?P<pk>\d+)', Book4View.as_view()),
]
```
### 基于ModelViewSet
#### views.py
```python
from rest_framework.viewsets import ModelViewSet
from app01.models import Book
from app01.ser import BookSerializer

class Book5View(ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```
#### urls.py
```python
urlpatterns = [
    # 基于ModelViewSet
    path('books5/', Book5View.as_view(actions={'get':'list', 'post':'create'})),
    re_path('book5/(?P<pk>\d+)', Book5View.as_view(actions={'get': 'retrieve',
                                                            'put': 'update',
                                                            'delete': 'destroy'})),
]
```