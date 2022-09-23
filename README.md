# Django_world_books
Этот сайт сделан на Pyton с помощью прикладных средств администрироввания Django.
База данных в MySql:
![image](https://user-images.githubusercontent.com/95234863/191929614-513dec4f-28b3-4085-a75e-81991ca7533a.png)


Разработка сайта проходила в программе PyCharm Community Edition 2022.2

Звголовочные файлы будут выглядеть следующим образом:

1) Файл views.py выглядит следующим образом:
 ```
 from django.shortcuts import render
from django.http import HttpResponse
from .models import Book, Author, BookInstance, Genre
from django.views import generic
from django.contrib.auth.mixins import LoginRequiredMixin
from django.http import *
from django.shortcuts import render
from . import AuthorsForm
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Book


class BookCreate(CreateView):
    model = Book
    fields = '__all__'
    success_url = reverse_lazy('books')


class BookUpdate(UpdateView):
    model = Book
    fields = '__all__'
    success_url = reverse_lazy('books')


class BookDelete(DeleteView):
    model = Book
    success_url = reverse_lazy('books')

def authors_add(request):
    author = Author.objects.all()
    authorsform = AuthorsForm()
    return render(request, "catalog/authors_add.html",
                  {"form": authorsform, "author": author})

class LoanedBooksByUserListView(LoginRequiredMixin, generic.ListView):
    """
    Универсальный класс представления списка книг,
    находящихся в заказе у текущего пользователя.
    """
    model = BookInstance
    template_name = 'catalog/bookinstance_list_borrowed_user.html'
    paginate_by = 10

    def get_queryset(self):
        return BookInstance.objects.filter(borrower=self.request.user).filter(status__exact='2').order_by('due_back')




def index(request, num_visits=None):
    # Генерация "количеств" некоторых главных объектов
    num_books = Book.objects.all().count()
    num_instances = BookInstance.objects.all().count()
    # Доступные книги (статус = 'На складе')
    # Здесь метод 'all()' применен по умолчанию.
    num_instances_available = BookInstance.objects.filter(status__exact=2).count()
    # Авторы книг,
    num_authors = Author.objects.count()

    num_visits = request.session.get('num_visits', 0)
    request.session['num_visits'] = num_visits + 1

    # Отрисовка HTML-шаблона index.html с данными
    # внутри переменной context
    return render(request, 'index.html', context={
                                         'num_books': num_books,
                                         'num_instances': num_instances,
                                         'num_instances_available': num_instances_available,
                                         'num_authors': num_authors,
                                         'num_visits': num_visits})
def create(request):
    if request.method == "POST":
        author = Author()
        author.first_name = request.POST.get("first_name")
        author.last_name = request.POST.get("last_name")
        author.date_of_birth = request.POST.get("date_of_birth")
        author.date_of_death = request.POST.get("date_of_death")
        author.save()
        return HttpResponseRedirect("/authors_add/")

def edit1(request, id):
    author = Author.objects.get(id=id)
    if request.method == "POST":
        author.first_name = request.POST.get("first_name")
        author.last_name = request.POST.get("last_name")
        author.date_of_birth = request.POST.get("date_of_birth")
        author.date_of_death = request.POST.get("date_of_death")
        author.save()
        return HttpResponseRedirect("/authors_add/")
    else:
        return render(request, "edit1.html", {"author": author})

class BookListView(generic.ListView):
    model = Book
    paginate_by = 20

class BookDetailView(generic.DetailView):
    model = Book

class AuthorListView(generic.ListView):
        model = Author
        paginate_by = 20
 ```
2) Файл urlys.py выглядит следующим образом:
  ```
  from django.contrib import admin
from django.urls import path
from catalog import views
from django.urls import path, include
from django.conf.urls import url
from django.urls import re_path as url

urlpatterns = [
    path('edit1/<int:id>/', views.edit1, name='edit1'),
    path('create/', views.create, name='create'),
    path('delete/<int:id>/', views.delete, name='delete'),
    path('', views.index, name='index'),
    path('authors_add/', views.authors_add, narne= 'authors_add'),
    path('admin/', admin.site.urls),
    url(r'^books/$', views.BookListView.as_view(), name='books'),
    url(r'^book/(?P<pk>\d+)$', views.BookDetailView.as_view(), name='book-detail'),
    url(r'^authors/$', views.AuthorListView.as_view(), name='authors'),
    url(r'^mybooks/$', views.LoanedBooksByUserListView.as_view(), name='my-borrowed'),

]
from django.urls import path, include

# Добавление URL-адреса для входа в систему
urlpatterns += [
    path('accounts/', include('django.contrib.auth.urls')),
]
urlpatterns += [
    url(r'^book/create/$', views.BookCreate.as_view(), name='book_create'),
    url(r'^book/update/(?P<pk>\d+)$', views.BookUpdate.as_view(), name='book_update'),
    url(r'^book/delete/(?P<pk>\d+)$', views.BookDelete.as_view(), name='book_delete'),
]
   ```
   
   Файл settings.py выглядит следующим образом:
  ```
  """
Django settings for WebBooks project.

Generated by 'django-admin startproject' using Django 4.1.

For more information on this file, see
https://docs.djangoproject.com/en/4.1/topics/settings/

For the full list of settings and their values, see
https://docs.djangoproject.com/en/4.1/ref/settings/
"""
import os
from pathlib import Path

# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent


# Quick-start development settings - unsuitable for production
# See https://docs.djangoproject.com/en/4.1/howto/deployment/checklist/

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY', 'cg#p$g+j9tax!#a3cup@1$8obt2_+&k3q+pmu)5%asj6yjpkag')


# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True
DEBUG = bool(os.environ.get('DJANGO_DEBUG', True))


ALLOWED_HOSTS = []


# Application definition

INSTALLED_APPS = [
    'catalog',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    ]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'WebBooks.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'WebBooks.wsgi.application'


# Database
# https://docs.djangoproject.com/en/4.1/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}


# Password validation
# https://docs.djangoproject.com/en/4.1/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


# Internationalization
# https://docs.djangoproject.com/en/4.1/topics/i18n/

SESSION_SAVE_EVERY_REQUEST = True

LANGUAGE_CODE = 'ru-ru'

TIME_ZONE = 'Europe/Moscow'

USE_I18N = True

USE_Ll0N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/4.1/howto/static-files/

STATIC_URL = 'static/'

# Default primary key field type
# https://docs.djangoproject.com/en/4.1/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Переадресация на главную страницу сайта после входа в систему
LOGIN_REDIRECT_URL = '/'

EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
  
  ```
  
 3) Файл forms.py выглядит следующим образом:
  ```
  
 from django import forms
from datetime import date
from django.forms import ModelForm
from .models import Book


class BookModelForm(ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'genre', 'language', 'author', 'summary', 'isbn']
        fields = [' summary', ]
        labels = {' summary ': ('Аннотация'), }
        help_texts = {' summary ': ('Не более 1000 символов'), }

class AuthorsForm(forms.Form):
    first_name = forms.CharField(label="Имя автора")
    last_name = forms.CharField(label="Фамилия автора")
    date_of_birth = forms.DateField(label="Дата рождения",
                                    initial=format(date.today()),
                                    widget=forms.widgets.DateInput(attrs={'type': 'date'}))
    date_of_death = forms.DateField(label="Дата смерти",
                                    initial=format(date.today()),
                                    widget=forms.widgets.DateInput(attrs={'type': 'date'}))
  ```
  
 4)  Файл admin.py выглядит следующим образом:
  ```
  from django.contrib import admin
from .models import Author, Book, Genre, Language, Status, BookInstance


# admin.site.register(Author)
class AuthorAdmin(admin.ModelAdmin):
    list_display = ('last_name', 'first_name')
    fields = ['first_name', 'last_name',
              ('date_of_birth', 'date_of_death')]


admin.site.register(Author, AuthorAdmin)


# admin.site.register(Book)

class BooksInstanceInline(admin.TabularInline):
    model = BookInstance


@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ('title', 'genre', 'language', 'display_author')
    list_filter = ('genre', 'author')
    display = ('title', 'author', 'display_genre')
    inlines = [BooksInstanceInline]


admin.site.register(Genre)

admin.site.register(Language)

admin.site.register(Status)

# admin.site.register(BookInstance)


@admin.register(BookInstance)
class BookInstanceAdmin(admin.ModelAdmin):
    list_display = ('book', 'status', 'borrower', 'due_back', 'id')
    list_filter = ('status', 'due_back')

    fieldsets = (
        (None, {
            'fields': ('book', 'imprint', 'inv_nom')
        }),
        ('Availability', {
            'fields': ('status', 'due_back', 'borrower')
        }
         ),
    )
   ```
   
   
     Остальные  файлы с расширением .html вы можете найти в архиве в папке catalog.
    
  
  
  
  
