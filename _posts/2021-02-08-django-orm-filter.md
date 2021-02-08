---
layout: post
title: Django ORM Filter
category: Python
tags: [pyhthon, django]
---
SELECT ALL  
SQL  
SELECT * FROM web_article;  
ORM  
Article.objects.all()  
  
WHERE  
SQL  
SELECT * FROM web_article WHERE id=1;  
ORM  
Article.objects.filter(id=1)  
  
LIMIT n  
SQL  
SELECT * FROM web_article LIMIT 10;  
ORM  
Article.objects.all()[:10]  
  
LIMIT n,n  
SQL  
SELECT * FROM web_article LIMIT 5,5;  
ORM  
Article.objects.all()[5:10]  
  
fetchone  
SQL  
SELECT * FROM web_article WHERE id=1;  
ORM  
Article.objects.get(id=1) #Return : Object  
  
fetchall  
SQL  
SELECT * FROM web_article WHERE site_id=1;  
ORM  
Article.objects.filter(site_id=1) #Return : QuerySet  
  
AND  
SQL  
SELECT * FROM web_article WHERE site_id=1 AND hit=0;  
ORM  
Article.objects.filter(site_id=1,hit=0)  
  
OR  
SQL  
SELECT * FROM web_article WHERE site_id=1 OR hit=0;  
ORM  
from django.db.models import Q  
Article.objects.filter(Q(site_id=1)|Q(hit=0))  

LIKE '%s%'
SQL
SELECT * FROM web_article WHERE subject LIKE '%공무원%';
ORM
Article.objects.filter(subject__icontains='공무원')
  
LIKE 's%'  
SQL  
SELECT * FROM web_article WHERE SUBJECT LIKE '유승민%';  
ORM  
Article.objects.filter(subject__startswith='유승민')  
  
LIKE '%s'  
SQL  
SELECT * FROM web_article WHERE SUBJECT LIKE '%의혹';  
ORM  
Article.objects.filter(subject__endswith='의혹')  
  
>=  
SQL  
SELECT * FROM web_article WHERE hit >= 2;  
ORM  
Article.objects.filter(hit__gte=2)  
  
<=  
SQL  
SELECT * FROM web_article WHERE hit <= 2;  
ORM  
Article.objects.filter(hit__lte=2)  
  
>  
SQL  
SELECT * FROM web_article WHERE hit > 1;  
ORM  
Article.objects.filter(hit__gt=1)  
  
<  
SQL  
SELECT * FROM web_article WHERE hit < 1;  
ORM  
Article.objects.filter(hit__lt=1)  
  
LEFT JOIN(ManyToManyField)  
SQL  
SELECT b.id FROM web_article_category AS a LEFT JOIN web_article AS b ON a.article_id = b.id LEFT JOIN web_category AS c ON c.id=a.category_id WHERE c.name='정치';  
ORM  
Category.objects.get(name='정치').article_set.all()  
  
INSERT  
집어넣기  
SQL  
INSERT INTO web_site SET name='뉴스타파';  
ORM  
site = Site(name='뉴스타파')  
site.save() #commit  
  
있으면 가져오고 없으면 집어 넣기  
SQL  
INSERT INTO web_site SET name='한겨레';  
ORM  
site = Site.objects.get_or_create(name='한겨레') #save 메서드 호출 없이도 바로 입력됨  
  
ForeignKey, ManyToManyField  
SQL  
INSERT INTO web_site SET name='PPSS'; #id = 7  
INSERT INTO web_article SET subject='뉴스제목', url='http://news.com/12345', date='2017-02-02 12:34:56', site_id=7; #id = 60  
INSERT INTO web_category SET name='정치'; #id = 1  
INSERT INTO web_category SET name='뉴스'; #id = 7  
INSERT INTO web_article_category SET article_id=60, category_id=1; #category = 정치  
INSERT INTO web_article_category SET article_id=60, category_id=7; #category = 뉴스  
ORM  
site, created = Site.objects.get_or_create(name='PPSS')  
article = Article(subject='뉴스제목', url='http://news.com', date='2017-02-0 2 12:34:56', site=site)  
article.save() #commit  
cate1, created = Category.objects.get_or_create(name='정치')  
cate2, created = Category.objects.get_or_create(name='뉴스')  
article.category.add(cate1)  #relationship 추가  
article.category.add(cate2)  
article.category.remove(cate2) #relationship 취소  
article.category.add(cate2)  
  
DELETE  
web_article 테이블의 기사를 하나 지우려고 할때,   
SQL에서는 relationshop이 있는 web_article_category의 해당 article_id를 삭제해 주어야 한다.  
하지만, ORM에서는 article 객체의 delete 메서드를 이용하면 relationshop에 대한 부분까지 고려해가면서 코딩을 할필요가 없어진다.  
SQL  
DELETE FROM web_article_category WHERE article_id=2;  
DELETE FROM web_article WHERE id=2;  
ORM  
Article.objects.get(id=2).delete()  
  
UPDATE  
SQL  
UPDATE web_article SET subject = '제목변경' WHERE id=4;  
ORM  
article = Article.objects.get(id=4)  
article.subject = '제목변경'  
article.save() #commit  
