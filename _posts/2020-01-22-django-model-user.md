---
layout: post
title: django user model
category: Python
tags: [pyhthon, django]
---

User model의 기본적으로 있는 여러 필드들이 있다.

username

필수사항이다. 150자 이하, 영숫자를 포함하도록 기본적인 vaildate가 있다.

first_name

선택사항이다. 30자 이하

last_name

선택사항. 150자 이하

email

선택사항

password

필수 사항. 설정한 암호의 해시값과 메타데이터값이다. 장고는 원래 암호를 그대로 저장하지 않고 특정 알고리즘으로 암호화후 저장한다.

groups

그룹에 대한 필드이다.

user_permissions

유저의 권한을 설정하는 필드

is_staff

참이면 admin 사이트에 접속할 수 있다.

is_active

이 계정이 활성인지를 결정.

is_superuser 모든 권한을 갖게 한다.

Boolean. Designates that this user has all permissions without explicitly assigning them.

last_login

마지막으로 로그인한 시간을 기록

date_joined

계정이 생성된 날짜를 기록

유저 확장
기본 필드들에 만족하고 사용하면 좋겠지만 기존 User model에서 추가, 수정, 삭제하고 싶은 경우도 있다. 그러기 위한 3가지 방법이 있다.

Proxy Model
User model 과 One-To-One 연결
AbstractBaseUser
AbstractUser
Proxy Model
User 모델을 직접 상속하고 class Meta 에서 proxy=True 속성으로 Proxy model임을 설정할 수 있다.

이 방법은 기존의 User 스키마는 그대로 두고 기존 동작을 변경하거나 추가하는데 사용된다.

from django.contrib.auth.models import User
from .managers import PersonManager

class Person(User):
    objects = PersonManager()

    class Meta:
        proxy = True
        ordering = ('first_name', )

    def do_something(self):
        ...
이 예시에서 정렬을 first_name으로 했다. do_something() 처럼 메소드를 추가해 새로운 동작을 추가할 수도 있다.

하지만 필드를 바꾸지 못하니 자주 사용하진 않는 방법이다.

User model 과 One-To-One 연결
User 객체가 생성될 때 같이 연결된 사용자 모델이 같이 생성되게 하는 방법이다. 기존 User 모델에 손상주지 않으면서 새 필드들을 추가할 수 있다.

from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField(max_length=500, blank=True)
    location = models.CharField(max_length=30, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
receiver를 사용하면 이벤트가 발생할 때를 찾을 수 있다. Save 이벤트가 발생할때마다 create_user_profile와 save_user_profile 를 호출해 User가 생성될때 Profile 모델도 생성되도록한다.

템플릿에서 다음과 같이 사용할 수 있다.

<h2></h2>
<ul>
  <li>Username: </li>
  <li>Location: </li>
  <li>Birth Date: </li>
</ul>
view.py에서는 다음과 같이 사용할 수 있다.

def update_profile(request, user_id):
    user = User.objects.get(pk=user_id)
    user.profile.bio = 'Lorem ipsum dolor sit amet, consectetur adipisicing elit...'
    user.save()
form 만들기
forms.py
class UserForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ('first_name', 'last_name', 'email')

class ProfileForm(forms.ModelForm):
    class Meta:
        model = Profile
        fields = ('url', 'location', 'company')
view.py
@login_required
@transaction.atomic
def update_profile(request):
    if request.method == 'POST':
        user_form = UserForm(request.POST, instance=request.user)
        profile_form = ProfileForm(request.POST, instance=request.user.profile)
        if user_form.is_valid() and profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, _('Your profile was successfully updated!'))
            return redirect('settings:profile')
        else:
            messages.error(request, _('Please correct the error below.'))
    else:
        user_form = UserForm(instance=request.user)
        profile_form = ProfileForm(instance=request.user.profile)
    return render(request, 'profiles/profile.html', {
        'user_form': user_form,
        'profile_form': profile_form
    })
profile.html

<form method="post">
  {% csrf_token %}
  {{ user_form.as_p }}
  {{ profile_form.as_p }}
  <button type="submit">Save changes</button>
</form>

AbstractBaseUser
완전히 새로운 User 모델을 상속받아 새로 정의한다. 데이터 스키마에 영향을 많이 주기 때문에 주의하며 사용해야한다. 기존 필드뿐만 아니라 동작까지 다 구현한다.

from __future__ import unicode_literals

from django.db import models
from django.core.mail import send_mail
from django.contrib.auth.models import PermissionsMixin
from django.contrib.auth.base_user import AbstractBaseUser
from django.utils.translation import ugettext_lazy as _

from .managers import UserManager


class User(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(_('email address'), unique=True)
    first_name = models.CharField(_('first name'), max_length=30, blank=True)
    last_name = models.CharField(_('last name'), max_length=30, blank=True)
    date_joined = models.DateTimeField(_('date joined'), auto_now_add=True)
    is_active = models.BooleanField(_('active'), default=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)

    objects = UserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    class Meta:
        verbose_name = _('user')
        verbose_name_plural = _('users')

    def get_full_name(self):
        '''
        Returns the first_name plus the last_name, with a space in between.
        '''
        full_name = '%s %s' % (self.first_name, self.last_name)
        return full_name.strip()

    def get_short_name(self):
        '''
        Returns the short name for the user.
        '''
        return self.first_name

    def email_user(self, subject, message, from_email=None, **kwargs):
        '''
        Sends an email to this User.
        '''
        send_mail(subject, message, from_email, [self.email], **kwargs)
User 모델을 관리하는 Manager도 정의해야한다.

from django.contrib.auth.base_user import BaseUserManager

class UserManager(BaseUserManager):
    use_in_migrations = True

    def _create_user(self, email, password, **extra_fields):
        """
        Creates and saves a User with the given email and password.
        """
        if not email:
            raise ValueError('The given email must be set')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_user(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_superuser', False)
        return self._create_user(email, password, **extra_fields)

    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault('is_superuser', True)

        if extra_fields.get('is_superuser') is not True:
            raise ValueError('Superuser must have is_superuser=True.')

        return self._create_user(email, password, **extra_fields)
그리고 이 User를 사용하기 위해 setting.py 에서 필드를 추가한다.

AUTH_USER_MODEL = 'core.User'
참조하는 법
이 모델을 다른 모델에서 참조하는 방법은 2가지가 있다.

from django.db import models
from testapp.core.models import User

class Course(models.Model):
    slug = models.SlugField(max_length=100)
    name = models.CharField(max_length=100)
    tutor = models.ForeignKey(User, on_delete=models.CASCADE)
재사용하고 싶거나 공개로 하고 싶을때는 다음과 같이 한다.

from django.db import models
from testapp.core.models import User

class Course(models.Model):
    slug = models.SlugField(max_length=100)
    name = models.CharField(max_length=100)
    tutor = models.ForeignKey(User, on_delete=models.CASCADE)
AbstractUser
User에서 동작은 그대로 하고 필드만 재정의할 때 사용한다.

from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    bio = models.TextField(max_length=500, blank=True)
    location = models.CharField(max_length=30, blank=True)
    birth_date = models.DateField(null=True, blank=True)
이렇게 하면 기존 필드에서 정의한 필드가 추가된다. 똑같이 setting.py에서 이 User를 사용한다고 명시한다.

AUTH_USER_MODEL = 'core.User'

결론

proxy model : Django User가 제공하는 모든 것에 만족하며 추가 정보를 저장할 필요가 없습니다.
One to One : Django가 인증을 처리하는 방식에 만족하며 인증되지 않은 일부 관련 속성을 사용자에게 추가해야합니다.
AbstractBaseUser : Django가 인증을 처리하는 방식이 프로젝트에 맞지 않습니다.
AbstractUser : Django가 인증을 처리하는 방법은 프로젝트에 완벽하게 맞지만 별도의 모델을 만들지 않고도 추가 속성을 추가하려고합니다.
