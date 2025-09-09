# Django Cheat Sheet

A comprehensive reference for Django - a high-level Python web framework that encourages rapid development and clean, pragmatic design.

---

## Table of Contents
- [Installation and Setup](#installation-and-setup)
- [Project Structure](#project-structure)
- [Models](#models)
- [Views](#views)
- [Templates](#templates)
- [URL Routing](#url-routing)
- [Forms](#forms)
- [Admin Interface](#admin-interface)
- [Authentication](#authentication)
- [Middleware](#middleware)
- [Static Files](#static-files)
- [Database Operations](#database-operations)
- [REST Framework](#rest-framework)
- [Testing](#testing)
- [Deployment](#deployment)
- [Best Practices](#best-practices)

---

## Installation and Setup

### Installation
```bash
# Install Django
pip install django

# Create new project
django-admin startproject myproject
cd myproject

# Create new app
python manage.py startapp myapp

# Run development server
python manage.py runserver

# Create superuser
python manage.py createsuperuser

# Make migrations
python manage.py makemigrations
python manage.py migrate
```

### Virtual Environment Setup
```bash
# Create virtual environment
python -m venv django_env

# Activate (Windows)
django_env\Scripts\activate

# Activate (Mac/Linux)
source django_env/bin/activate

# Install requirements
pip install -r requirements.txt

# Generate requirements file
pip freeze > requirements.txt
```

## Project Structure

### Django Project Layout
```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
└── myapp/
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    ├── views.py
    ├── urls.py
    ├── migrations/
    ├── templates/
    └── static/
```

### Settings Configuration
```python
# settings.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# Security
SECRET_KEY = 'your-secret-key-here'
DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# Applications
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',
    'rest_framework',  # If using DRF
]

# Middleware
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myproject.urls'

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}

# Static files
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_L10N = True
USE_TZ = True
```

## Models

### Basic Model Definition
```python
# models.py
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse

class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name_plural = "Categories"
        ordering = ['name']
    
    def __str__(self):
        return self.name
    
    def get_absolute_url(self):
        return reverse('category-detail', kwargs={'pk': self.pk})

class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    tags = models.ManyToManyField('Tag', blank=True)
    content = models.TextField()
    excerpt = models.CharField(max_length=300, blank=True)
    featured_image = models.ImageField(upload_to='posts/', blank=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    published_date = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', 'published_date']),
        ]
    
    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        return reverse('post-detail', kwargs={'slug': self.slug})
    
    @property
    def is_published(self):
        return self.status == 'published'

class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    color = models.CharField(max_length=7, default='#007bff')  # Hex color
    
    def __str__(self):
        return self.name

class Comment(models.Model):
    post = models.ForeignKey(Post, related_name='comments', on_delete=models.CASCADE)
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    parent = models.ForeignKey('self', null=True, blank=True, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    is_approved = models.BooleanField(default=True)
    
    class Meta:
        ordering = ['created_at']
    
    def __str__(self):
        return f'Comment by {self.author.username} on {self.post.title}'
```

### Advanced Model Features
```python
from django.db import models
from django.core.validators import MinValueValidator, MaxValueValidator
from django.contrib.auth.models import AbstractUser

# Custom User Model
class CustomUser(AbstractUser):
    email = models.EmailField(unique=True)
    bio = models.TextField(max_length=500, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', default='avatars/default.png')
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

# Model with custom methods
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    discount = models.IntegerField(
        validators=[MinValueValidator(0), MaxValueValidator(100)],
        default=0
    )
    
    def get_discounted_price(self):
        return self.price * (1 - self.discount / 100)
    
    @classmethod
    def get_featured_products(cls):
        return cls.objects.filter(featured=True)
    
    def save(self, *args, **kwargs):
        # Custom save logic
        if not self.slug:
            self.slug = slugify(self.name)
        super().save(*args, **kwargs)

# Abstract Base Class
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True

class Article(TimeStampedModel):
    title = models.CharField(max_length=200)
    content = models.TextField()
```

## Views

### Function-Based Views
```python
# views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpResponse, JsonResponse
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from django.core.paginator import Paginator
from .models import Post, Category
from .forms import PostForm

def home(request):
    posts = Post.objects.filter(status='published').select_related('author', 'category')
    return render(request, 'blog/home.html', {'posts': posts})

def post_list(request):
    posts = Post.objects.filter(status='published')
    
    # Search functionality
    search_query = request.GET.get('search')
    if search_query:
        posts = posts.filter(title__icontains=search_query)
    
    # Pagination
    paginator = Paginator(posts, 10)
    page_number = request.GET.get('page')
    page_obj = paginator.get_page(page_number)
    
    context = {
        'page_obj': page_obj,
        'search_query': search_query,
    }
    return render(request, 'blog/post_list.html', context)

def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, status='published')
    comments = post.comments.filter(is_approved=True)
    
    if request.method == 'POST':
        # Handle comment submission
        content = request.POST.get('content')
        if content and request.user.is_authenticated:
            Comment.objects.create(
                post=post,
                author=request.user,
                content=content
            )
            messages.success(request, 'Comment added successfully!')
            return redirect('post-detail', slug=slug)
    
    context = {
        'post': post,
        'comments': comments,
    }
    return render(request, 'blog/post_detail.html', context)

@login_required
def post_create(request):
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            form.save_m2m()  # For ManyToMany fields
            messages.success(request, 'Post created successfully!')
            return redirect('post-detail', slug=post.slug)
    else:
        form = PostForm()
    
    return render(request, 'blog/post_form.html', {'form': form})

def api_posts(request):
    posts = Post.objects.filter(status='published').values(
        'id', 'title', 'slug', 'created_at'
    )
    return JsonResponse(list(posts), safe=False)
```

### Class-Based Views
```python
# views.py
from django.views.generic import ListView, DetailView, CreateView, UpdateView, DeleteView
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.urls import reverse_lazy
from django.contrib import messages

class PostListView(ListView):
    model = Post
    template_name = 'blog/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10
    
    def get_queryset(self):
        queryset = Post.objects.filter(status='published').select_related('author')
        
        # Filter by category
        category = self.request.GET.get('category')
        if category:
            queryset = queryset.filter(category__name=category)
        
        return queryset
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Category.objects.all()
        return context

class PostDetailView(DetailView):
    model = Post
    template_name = 'blog/post_detail.html'
    context_object_name = 'post'
    
    def get_queryset(self):
        return Post.objects.filter(status='published')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['comments'] = self.object.comments.filter(is_approved=True)
        return context

class PostCreateView(LoginRequiredMixin, CreateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    
    def form_valid(self, form):
        form.instance.author = self.request.user
        messages.success(self.request, 'Post created successfully!')
        return super().form_valid(form)

class PostUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Post
    form_class = PostForm
    template_name = 'blog/post_form.html'
    
    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
    
    def form_valid(self, form):
        messages.success(self.request, 'Post updated successfully!')
        return super().form_valid(form)

class PostDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Post
    template_name = 'blog/post_confirm_delete.html'
    success_url = reverse_lazy('post-list')
    
    def test_func(self):
        post = self.get_object()
        return self.request.user == post.author
    
    def delete(self, request, *args, **kwargs):
        messages.success(request, 'Post deleted successfully!')
        return super().delete(request, *args, **kwargs)
```

## Templates

### Base Template
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My Django App{% endblock %}</title>
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{% url 'home' %}">My Blog</a>
            <div class="navbar-nav ms-auto">
                {% if user.is_authenticated %}
                    <a class="nav-link" href="{% url 'post-create' %}">Create Post</a>
                    <a class="nav-link" href="{% url 'logout' %}">Logout</a>
                {% else %}
                    <a class="nav-link" href="{% url 'login' %}">Login</a>
                    <a class="nav-link" href="{% url 'register' %}">Register</a>
                {% endif %}
            </div>
        </div>
    </nav>
    
    <main class="container mt-4">
        {% if messages %}
            {% for message in messages %}
                <div class="alert alert-{{ message.tags }} alert-dismissible fade show" role="alert">
                    {{ message }}
                    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                </div>
            {% endfor %}
        {% endif %}
        
        {% block content %}
        {% endblock %}
    </main>
    
    <footer class="bg-dark text-light text-center py-3 mt-5">
        <p>&copy; 2023 My Django App. All rights reserved.</p>
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    {% block scripts %}
    {% endblock %}
</body>
</html>
```

### List Template
```html
<!-- templates/blog/post_list.html -->
{% extends 'base.html' %}
{% load static %}

{% block title %}Blog Posts - {{ block.super }}{% endblock %}

{% block content %}
<div class="row">
    <div class="col-md-8">
        <h1>Latest Posts</h1>
        
        <!-- Search Form -->
        <form method="GET" class="mb-4">
            <div class="input-group">
                <input type="text" name="search" class="form-control" 
                       placeholder="Search posts..." value="{{ search_query }}">
                <button class="btn btn-outline-secondary" type="submit">Search</button>
            </div>
        </form>
        
        <!-- Posts -->
        {% for post in page_obj %}
            <article class="card mb-4">
                {% if post.featured_image %}
                    <img src="{{ post.featured_image.url }}" class="card-img-top" alt="{{ post.title }}">
                {% endif %}
                <div class="card-body">
                    <h2 class="card-title">
                        <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
                    </h2>
                    <p class="card-text">{{ post.excerpt|default:post.content|truncatewords:30 }}</p>
                    <div class="d-flex justify-content-between align-items-center">
                        <small class="text-muted">
                            By {{ post.author.get_full_name|default:post.author.username }} on 
                            {{ post.created_at|date:"M d, Y" }}
                        </small>
                        {% if post.category %}
                            <span class="badge bg-secondary">{{ post.category.name }}</span>
                        {% endif %}
                    </div>
                </div>
            </article>
        {% empty %}
            <p>No posts found.</p>
        {% endfor %}
        
        <!-- Pagination -->
        {% if page_obj.has_other_pages %}
            <nav aria-label="Posts pagination">
                <ul class="pagination justify-content-center">
                    {% if page_obj.has_previous %}
                        <li class="page-item">
                            <a class="page-link" href="?page=1">First</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.previous_page_number }}">Previous</a>
                        </li>
                    {% endif %}
                    
                    <li class="page-item active">
                        <span class="page-link">{{ page_obj.number }} of {{ page_obj.paginator.num_pages }}</span>
                    </li>
                    
                    {% if page_obj.has_next %}
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.next_page_number }}">Next</a>
                        </li>
                        <li class="page-item">
                            <a class="page-link" href="?page={{ page_obj.paginator.num_pages }}">Last</a>
                        </li>
                    {% endif %}
                </ul>
            </nav>
        {% endif %}
    </div>
    
    <div class="col-md-4">
        <div class="card">
            <div class="card-header">
                <h5>Categories</h5>
            </div>
            <div class="card-body">
                {% for category in categories %}
                    <a href="?category={{ category.name }}" class="btn btn-sm btn-outline-primary me-1 mb-1">
                        {{ category.name }}
                    </a>
                {% endfor %}
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### Template Filters and Tags
```html
<!-- Custom template tags -->
{% load static %}
{% load blog_extras %}  <!-- Custom template tags -->

<!-- Built-in filters -->
{{ post.title|title }}
{{ post.content|truncatewords:50 }}
{{ post.created_at|date:"F d, Y" }}
{{ post.created_at|timesince }} ago
{{ post.content|linebreaks }}
{{ post.content|striptags }}
{{ value|default:"No value" }}

<!-- Custom filters (in templatetags/blog_extras.py) -->
{{ post.content|markdown }}
{{ post|reading_time }}

<!-- Template tags -->
{% for post in posts %}
    {% cycle 'odd' 'even' as row_class %}
    <div class="{{ row_class }}">
        {{ post.title }}
    </div>
{% endfor %}

{% now "Y" %}
{% lorem 3 p %}

<!-- Conditional tags -->
{% if user.is_authenticated %}
    Welcome, {{ user.username }}!
{% elif user.is_anonymous %}
    Please log in.
{% else %}
    Hello, guest!
{% endif %}

<!-- Loop tags -->
{% for post in posts %}
    {{ forloop.counter }}. {{ post.title }}
    {% if forloop.first %}
        <p>This is the first post.</p>
    {% endif %}
    {% if forloop.last %}
        <p>This is the last post.</p>
    {% endif %}
{% empty %}
    <p>No posts available.</p>
{% endfor %}
```

## URL Routing

### Project URLs
```python
# myproject/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
    path('accounts/', include('django.contrib.auth.urls')),
    path('api/v1/', include('api.urls')),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### App URLs
```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    # Function-based views
    path('', views.home, name='home'),
    path('posts/', views.post_list, name='post-list'),
    path('post/<slug:slug>/', views.post_detail, name='post-detail'),
    path('create/', views.post_create, name='post-create'),
    
    # Class-based views
    path('posts/list/', views.PostListView.as_view(), name='post-list-cbv'),
    path('post/<slug:slug>/detail/', views.PostDetailView.as_view(), name='post-detail-cbv'),
    path('post/create/', views.PostCreateView.as_view(), name='post-create-cbv'),
    path('post/<slug:slug>/update/', views.PostUpdateView.as_view(), name='post-update'),
    path('post/<slug:slug>/delete/', views.PostDeleteView.as_view(), name='post-delete'),
    
    # API endpoints
    path('api/posts/', views.api_posts, name='api-posts'),
    
    # Category views
    path('category/<str:name>/', views.category_posts, name='category-posts'),
    
    # Advanced patterns
    path('archive/<int:year>/', views.year_archive, name='year-archive'),
    path('archive/<int:year>/<int:month>/', views.month_archive, name='month-archive'),
    path('search/', views.search_posts, name='search-posts'),
    
    # Include other app URLs
    path('comments/', include('comments.urls')),
]
```

### URL Patterns and Converters
```python
# URL pattern examples
from django.urls import path, re_path
from . import views

urlpatterns = [
    # Basic patterns
    path('', views.index, name='index'),
    path('about/', views.about, name='about'),
    
    # Path converters
    path('user/<int:user_id>/', views.user_detail, name='user-detail'),
    path('post/<slug:slug>/', views.post_detail, name='post-detail'),
    path('category/<str:name>/', views.category_detail, name='category-detail'),
    path('archive/<int:year>/<int:month>/<int:day>/', views.day_archive, name='day-archive'),
    
    # Regular expressions
    re_path(r'^post/(?P<year>[0-9]{4})/$', views.year_posts, name='year-posts'),
    re_path(r'^user/(?P<username>[\w.@+-]+)/$', views.user_profile, name='user-profile'),
    
    # Optional parameters
    path('posts/', views.post_list, name='post-list'),
    path('posts/page/<int:page>/', views.post_list, name='post-list-page'),
    
    # Nested includes
    path('blog/', include([
        path('', views.blog_index, name='blog-index'),
        path('post/<int:pk>/', views.post_detail, name='post-detail'),
    ])),
]
```

## Forms

### Model Forms
```python
# forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
from .models import Post, Comment, Category

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'category', 'tags', 'content', 'excerpt', 'featured_image', 'status']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 10}),
            'excerpt': forms.TextInput(attrs={'class': 'form-control'}),
            'category': forms.Select(attrs={'class': 'form-control'}),
            'tags': forms.CheckboxSelectMultiple(),
            'status': forms.Select(attrs={'class': 'form-control'}),
        }
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['category'].queryset = Category.objects.all()
        self.fields['title'].help_text = 'Enter a descriptive title for your post'
    
    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError('Title must be at least 5 characters long.')
        return title

class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 4,
                'placeholder': 'Write your comment here...'
            })
        }

class CustomUserCreationForm(UserCreationForm):
    email = forms.EmailField(required=True)
    first_name = forms.CharField(max_length=30, required=True)
    last_name = forms.CharField(max_length=30, required=True)
    
    class Meta:
        model = User
        fields = ('username', 'first_name', 'last_name', 'email', 'password1', 'password2')
    
    def save(self, commit=True):
        user = super().save(commit=False)
        user.email = self.cleaned_data['email']
        user.first_name = self.cleaned_data['first_name']
        user.last_name = self.cleaned_data['last_name']
        if commit:
            user.save()
        return user
```

### Regular Forms
```python
class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    subject = forms.CharField(
        max_length=200,
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={'class': 'form-control', 'rows': 5})
    )
    
    def clean_email(self):
        email = self.cleaned_data['email']
        if not email.endswith('@example.com'):
            raise forms.ValidationError('Email must be from example.com domain')
        return email
    
    def send_email(self):
        # Custom method to send email
        from django.core.mail import send_mail
        send_mail(
            subject=self.cleaned_data['subject'],
            message=self.cleaned_data['message'],
            from_email=self.cleaned_data['email'],
            recipient_list=['admin@example.com'],
        )

class SearchForm(forms.Form):
    query = forms.CharField(
        max_length=200,
        widget=forms.TextInput(attrs={
            'class': 'form-control',
            'placeholder': 'Search posts...'
        })
    )
    category = forms.ModelChoiceField(
        queryset=Category.objects.all(),
        empty_label='All Categories',
        required=False,
        widget=forms.Select(attrs={'class': 'form-control'})
    )
    date_from = forms.DateField(
        required=False,
        widget=forms.DateInput(attrs={'class': 'form-control', 'type': 'date'})
    )
    date_to = forms.DateField(
        required=False,
        widget=forms.DateInput(attrs={'class': 'form-control', 'type': 'date'})
    )
```

## Admin Interface

### Basic Admin Configuration
```python
# admin.py
from django.contrib import admin
from django.utils.html import format_html
from .models import Post, Category, Tag, Comment

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'post_count', 'created_at']
    list_filter = ['created_at']
    search_fields = ['name']
    prepopulated_fields = {'slug': ('name',)}
    
    def post_count(self, obj):
        return obj.post_set.count()
    post_count.short_description = 'Number of Posts'

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ['name', 'colored_name']
    
    def colored_name(self, obj):
        return format_html(
            '<span style="color: {}">{}</span>',
            obj.color,
            obj.name
        )
    colored_name.short_description = 'Colored Name'

class CommentInline(admin.TabularInline):
    model = Comment
    extra = 0
    readonly_fields = ['created_at']
    fields = ['author', 'content', 'is_approved', 'created_at']

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category', 'status', 'created_at', 'post_image']
    list_filter = ['status', 'category', 'created_at', 'author']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    list_per_page = 20
    
    fieldsets = (
        ('Basic Information', {
            'fields': ('title', 'slug', 'author', 'category')
        }),
        ('Content', {
            'fields': ('content', 'excerpt', 'featured_image')
        }),
        ('Metadata', {
            'fields': ('tags', 'status', 'published_date'),
            'classes': ('collapse',)
        }),
    )
    
    filter_horizontal = ['tags']
    inlines = [CommentInline]
    
    def post_image(self, obj):
        if obj.featured_image:
            return format_html(
                '<img src="{}" style="max-height: 50px;"/>',
                obj.featured_image.url
            )
        return 'No Image'
    post_image.short_description = 'Image'
    
    def save_model(self, request, obj, form, change):
        if not change:  # If creating new object
            obj.author = request.user
        super().save_model(request, obj, form, change)
    
    actions = ['make_published', 'make_draft']
    
    def make_published(self, request, queryset):
        updated = queryset.update(status='published')
        self.message_user(request, f'{updated} posts were successfully published.')
    make_published.short_description = 'Mark selected posts as published'
    
    def make_draft(self, request, queryset):
        updated = queryset.update(status='draft')
        self.message_user(request, f'{updated} posts were successfully marked as draft.')
    make_draft.short_description = 'Mark selected posts as draft'

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['post', 'author', 'created_at', 'is_approved']
    list_filter = ['is_approved', 'created_at']
    search_fields = ['content', 'author__username']
    actions = ['approve_comments', 'disapprove_comments']
    
    def approve_comments(self, request, queryset):
        queryset.update(is_approved=True)
    approve_comments.short_description = 'Approve selected comments'
    
    def disapprove_comments(self, request, queryset):
        queryset.update(is_approved=False)
    disapprove_comments.short_description = 'Disapprove selected comments'
```

## Authentication

### User Authentication Views
```python
# views.py
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required
from django.contrib.auth.forms import AuthenticationForm
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import CustomUserCreationForm

def register_view(request):
    if request.method == 'POST':
        form = CustomUserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            username = form.cleaned_data.get('username')
            messages.success(request, f'Account created for {username}!')
            login(request, user)
            return redirect('home')
    else:
        form = CustomUserCreationForm()
    
    return render(request, 'registration/register.html', {'form': form})

def login_view(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            username = form.cleaned_data.get('username')
            password = form.cleaned_data.get('password')
            user = authenticate(username=username, password=password)
            if user is not None:
                login(request, user)
                messages.info(request, f'Welcome back, {username}!')
                return redirect('home')
            else:
                messages.error(request, 'Invalid username or password.')
        else:
            messages.error(request, 'Invalid username or password.')
    else:
        form = AuthenticationForm()
    
    return render(request, 'registration/login.html', {'form': form})

def logout_view(request):
    logout(request)
    messages.info(request, 'You have successfully logged out.')
    return redirect('home')

@login_required
def profile_view(request):
    return render(request, 'registration/profile.html', {'user': request.user})
```

### Custom User Model
```python
# models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    email = models.EmailField(unique=True)
    bio = models.TextField(max_length=500, blank=True)
    birth_date = models.DateField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', default='avatars/default.png')
    is_verified = models.BooleanField(default=False)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
    
    def __str__(self):
        return self.email
    
    def get_full_name(self):
        return f"{self.first_name} {self.last_name}".strip()

# In settings.py
AUTH_USER_MODEL = 'myapp.CustomUser'
```

## Middleware

### Custom Middleware
```python
# middleware.py
from django.utils.deprecation import MiddlewareMixin
from django.http import HttpResponseForbidden
import logging

logger = logging.getLogger(__name__)

class RequestLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Code to be executed for each request before
        # the view (and later middleware) are called.
        
        logger.info(f"Request: {request.method} {request.path}")
        
        response = self.get_response(request)
        
        # Code to be executed for each request/response after
        # the view is called.
        
        logger.info(f"Response: {response.status_code}")
        
        return response

class IPBlockingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
        self.blocked_ips = ['192.168.1.100', '10.0.0.1']  # Example blocked IPs
    
    def __call__(self, request):
        ip = self.get_client_ip(request)
        
        if ip in self.blocked_ips:
            return HttpResponseForbidden('Your IP is blocked')
        
        response = self.get_response(request)
        return response
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip

class SecurityHeadersMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        # Add security headers
        response['X-Content-Type-Options'] = 'nosniff'
        response['X-Frame-Options'] = 'DENY'
        response['X-XSS-Protection'] = '1; mode=block'
        response['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
        
        return response
```

## Static Files

### Static Files Configuration
```python
# settings.py
import os

# Static files (CSS, JavaScript, Images)
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]

# Media files (User uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# Static file finders
STATICFILES_FINDERS = [
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
]

# Production settings
if not DEBUG:
    # Use whitenoise for static files in production
    MIDDLEWARE.insert(1, 'whitenoise.middleware.WhiteNoiseMiddleware')
    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

### Using Static Files in Templates
```html
{% load static %}

<!-- CSS -->
<link rel="stylesheet" type="text/css" href="{% static 'css/style.css' %}">

<!-- JavaScript -->
<script src="{% static 'js/script.js' %}"></script>

<!-- Images -->
<img src="{% static 'images/logo.png' %}" alt="Logo">

<!-- Dynamic static files -->
<img src="{% static 'images/'|add:post.image_name %}" alt="{{ post.title }}">

<!-- Get static files URL -->
{% get_static_prefix as STATIC_PREFIX %}
<img src="{{ STATIC_PREFIX }}images/logo.png" alt="Logo">
```

## Database Operations

### QuerySet Operations
```python
# Basic queries
from .models import Post, Category, Tag

# Get all objects
all_posts = Post.objects.all()

# Filter objects
published_posts = Post.objects.filter(status='published')
recent_posts = Post.objects.filter(created_at__gte=datetime.now() - timedelta(days=7))

# Get single object
try:
    post = Post.objects.get(slug='my-post')
except Post.DoesNotExist:
    post = None

# Get or create
post, created = Post.objects.get_or_create(
    title='My Post',
    defaults={'content': 'Default content', 'author': user}
)

# Exclude objects
non_draft_posts = Post.objects.exclude(status='draft')

# Order objects
latest_posts = Post.objects.order_by('-created_at')
alphabetical_posts = Post.objects.order_by('title')

# Limit results
first_10_posts = Post.objects.all()[:10]
posts_5_to_15 = Post.objects.all()[5:15]

# Complex lookups
matching_posts = Post.objects.filter(
    Q(title__icontains='django') | Q(content__icontains='python')
)

# Related object queries
posts_with_category = Post.objects.select_related('category')
posts_with_tags = Post.objects.prefetch_related('tags')

# Annotations and aggregations
from django.db.models import Count, Avg

categories_with_counts = Category.objects.annotate(
    post_count=Count('post')
)

average_posts_per_category = Category.objects.aggregate(
    avg_posts=Avg('post__id')
)

# Raw SQL
raw_posts = Post.objects.raw(
    'SELECT * FROM blog_post WHERE status = %s',
    ['published']
)

# Bulk operations
Post.objects.filter(status='draft').update(status='published')
Post.objects.filter(created_at__lt=old_date).delete()

bulk_posts = [
    Post(title=f'Post {i}', content=f'Content {i}', author=user)
    for i in range(100)
]
Post.objects.bulk_create(bulk_posts)
```

### Database Transactions
```python
from django.db import transaction
from django.db.models import F

# Atomic transactions
@transaction.atomic
def create_post_with_comments(post_data, comments_data):
    post = Post.objects.create(**post_data)
    
    for comment_data in comments_data:
        Comment.objects.create(post=post, **comment_data)
    
    return post

# Manual transaction management
def transfer_posts(from_user, to_user):
    with transaction.atomic():
        posts = Post.objects.filter(author=from_user)
        posts.update(author=to_user)
        
        # Update user stats
        from_user.post_count = F('post_count') - posts.count()
        to_user.post_count = F('post_count') + posts.count()
        
        from_user.save(update_fields=['post_count'])
        to_user.save(update_fields=['post_count'])

# Savepoints
def complex_operation():
    with transaction.atomic():
        # Create a savepoint
        sid = transaction.savepoint()
        
        try:
            # Some operation that might fail
            risky_operation()
        except Exception:
            # Rollback to savepoint
            transaction.savepoint_rollback(sid)
        else:
            # Commit savepoint
            transaction.savepoint_commit(sid)
```

## REST Framework

### DRF Serializers and ViewSets
```python
# serializers.py
from rest_framework import serializers
from .models import Post, Category, Tag

class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ['id', 'name', 'description']

class TagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tag
        fields = ['id', 'name', 'color']

class PostSerializer(serializers.ModelSerializer):
    author = serializers.StringRelatedField(read_only=True)
    category = CategorySerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'author', 'category', 'tags', 
                 'content', 'excerpt', 'status', 'created_at']
        read_only_fields = ['slug', 'created_at']
    
    def create(self, validated_data):
        validated_data['author'] = self.context['request'].user
        return super().create(validated_data)

# views.py
from rest_framework import viewsets, permissions, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['category', 'status']
    search_fields = ['title', 'content']
    ordering_fields = ['created_at', 'title']
    ordering = ['-created_at']
    
    def get_queryset(self):
        queryset = Post.objects.all()
        if self.action == 'list':
            queryset = queryset.filter(status='published')
        return queryset
    
    @action(detail=True, methods=['post'])
    def set_featured(self, request, pk=None):
        post = self.get_object()
        post.is_featured = not post.is_featured
        post.save()
        return Response({'status': 'featured status updated'})
    
    @action(detail=False)
    def recent(self, request):
        recent_posts = Post.objects.filter(status='published').order_by('-created_at')[:10]
        serializer = self.get_serializer(recent_posts, many=True)
        return Response(serializer.data)

# urls.py
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'posts', views.PostViewSet)
router.register(r'categories', views.CategoryViewSet)

urlpatterns = router.urls
```

## Testing

### Unit Tests
```python
# tests.py
from django.test import TestCase, Client
from django.contrib.auth.models import User
from django.urls import reverse
from .models import Post, Category
from .forms import PostForm

class PostModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.category = Category.objects.create(
            name='Test Category',
            description='A test category'
        )
    
    def test_post_creation(self):
        post = Post.objects.create(
            title='Test Post',
            slug='test-post',
            author=self.user,
            category=self.category,
            content='This is a test post content.',
            status='published'
        )
        
        self.assertEqual(post.title, 'Test Post')
        self.assertEqual(post.author, self.user)
        self.assertEqual(str(post), 'Test Post')
        self.assertTrue(post.is_published)
    
    def test_post_absolute_url(self):
        post = Post.objects.create(
            title='Test Post',
            slug='test-post',
            author=self.user,
            content='Test content'
        )
        self.assertEqual(post.get_absolute_url(), '/post/test-post/')

class PostViewTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(
            username='testuser',
            password='testpass123'
        )
        self.category = Category.objects.create(name='Test Category')
        self.post = Post.objects.create(
            title='Test Post',
            slug='test-post',
            author=self.user,
            category=self.category,
            content='Test content',
            status='published'
        )
    
    def test_post_list_view(self):
        response = self.client.get(reverse('post-list'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Test Post')
    
    def test_post_detail_view(self):
        response = self.client.get(reverse('post-detail', kwargs={'slug': 'test-post'}))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, 'Test Post')
    
    def test_post_create_requires_login(self):
        response = self.client.get(reverse('post-create'))
        self.assertEqual(response.status_code, 302)  # Redirect to login
    
    def test_post_create_with_login(self):
        self.client.login(username='testuser', password='testpass123')
        response = self.client.get(reverse('post-create'))
        self.assertEqual(response.status_code, 200)
        
        # Test form submission
        post_data = {
            'title': 'New Test Post',
            'content': 'New test content',
            'category': self.category.id,
            'status': 'published'
        }
        response = self.client.post(reverse('post-create'), post_data)
        self.assertEqual(response.status_code, 302)  # Redirect after successful creation
        self.assertTrue(Post.objects.filter(title='New Test Post').exists())

class PostFormTest(TestCase):
    def setUp(self):
        self.category = Category.objects.create(name='Test Category')
    
    def test_post_form_valid_data(self):
        form_data = {
            'title': 'Test Post',
            'content': 'This is a test post content.',
            'category': self.category.id,
            'status': 'published'
        }
        form = PostForm(data=form_data)
        self.assertTrue(form.is_valid())
    
    def test_post_form_no_title(self):
        form_data = {
            'content': 'This is a test post content.',
            'category': self.category.id,
            'status': 'published'
        }
        form = PostForm(data=form_data)
        self.assertFalse(form.is_valid())
        self.assertIn('title', form.errors)
```

## Deployment

### Production Settings
```python
# settings/production.py
from .base import *
import os

DEBUG = False
ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# Security settings
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_SECONDS = 31536000
SECURE_REDIRECT_EXEMPT = []
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
    }
}

# Static files
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'django.log',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}

# Cache
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
    }
}
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM python:3.9

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /code

COPY requirements.txt /code/
RUN pip install -r requirements.txt

COPY . /code/

RUN python manage.py collectstatic --noinput

CMD ["gunicorn", "myproject.wsgi:application", "--bind", "0.0.0.0:8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/code
    environment:
      - DEBUG=1
      - SECRET_KEY=your-secret-key
    depends_on:
      - db
      - redis

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres

  redis:
    image: redis:6
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

## Best Practices

### Project Organization
```python
# settings/base.py - Base settings
# settings/development.py - Development settings
# settings/production.py - Production settings
# settings/testing.py - Testing settings

# apps/
# ├── blog/
# ├── users/
# ├── core/
# └── api/

# Use environment variables
import os
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
DATABASE_URL = config('DATABASE_URL')

# Custom managers
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(status='published')

class Post(models.Model):
    # ... fields ...
    
    objects = models.Manager()  # Default manager
    published = PublishedManager()  # Custom manager

# Use model properties for computed fields
@property
def full_name(self):
    return f"{self.first_name} {self.last_name}"

# Use model methods for business logic
def can_edit(self, user):
    return self.author == user or user.is_staff

# Use signals for decoupled operations
from django.db.models.signals import post_save
from django.dispatch import receiver

@receiver(post_save, sender=Post)
def create_post_slug(sender, instance, created, **kwargs):
    if created and not instance.slug:
        instance.slug = slugify(instance.title)
        instance.save()
```

### Security Best Practices
```python
# Use Django's built-in security features
CSRF_COOKIE_SECURE = True
SESSION_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True

# Validate and sanitize user input
from django.core.validators import validate_email
from django.core.exceptions import ValidationError

def clean_email(self):
    email = self.cleaned_data['email']
    try:
        validate_email(email)
    except ValidationError:
        raise forms.ValidationError('Invalid email address')
    return email

# Use permissions and decorators
from django.contrib.auth.decorators import user_passes_test

def is_author(user):
    return user.is_authenticated and hasattr(user, 'author')

@user_passes_test(is_author)
def author_dashboard(request):
    # View for authors only
    pass

# Sanitize HTML content
from django.utils.html import strip_tags, escape

def clean_content(content):
    # Remove dangerous HTML tags
    return strip_tags(content)
```

---

## Common Commands

| Command | Purpose | Example |
|---------|---------|----------|
| `startproject` | Create new project | `django-admin startproject mysite` |
| `startapp` | Create new app | `python manage.py startapp blog` |
| `runserver` | Start development server | `python manage.py runserver` |
| `makemigrations` | Create migrations | `python manage.py makemigrations` |
| `migrate` | Apply migrations | `python manage.py migrate` |
| `collectstatic` | Collect static files | `python manage.py collectstatic` |
| `createsuperuser` | Create admin user | `python manage.py createsuperuser` |
| `shell` | Django shell | `python manage.py shell` |
| `test` | Run tests | `python manage.py test` |

---

## Resources
- [Official Django Documentation](https://docs.djangoproject.com)
- [Django REST Framework](https://www.django-rest-framework.org)
- [Django Packages](https://djangopackages.org)
- [Two Scoops of Django](https://www.twoscoopspress.com)
- [Django Best Practices](https://django-best-practices.readthedocs.io)

---
*Originally compiled from various sources. Contributions welcome!*