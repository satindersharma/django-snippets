

`models.py`

```python
from django.db import models
from django.contrib.auth.models import AbstractUser, Permission
from django.utils.translation import gettext, gettext_lazy as _
from django.db.models.signals import post_save

# Create your models here.

'''
delete migrations folder
python manage.py makemigrations profiles
python manage.py migrate --fake profiles zero
python manage.py migrate profiles
'''


class CustomUser(AbstractUser):

    # email_password = .CharField(db_column='department_name', max_length=45)
    user_id = models.SmallAutoField(primary_key=True)
    username = models.CharField(
        db_column='username', max_length=45, unique=True)
    name = models.CharField(
        db_column='name', max_length=45, blank=True, null=True)
    ph_1 = models.PositiveIntegerField(
        db_column='ph_1', blank=True, null=True)
    ph_2 = models.PositiveIntegerField(
        db_column='ph_2', blank=True, null=True)
    user_info_id = models.PositiveSmallIntegerField(
        db_column='user_info_id', blank=True, null=True)
    # department = models.CharField(
    #     db_column='department', max_length=45, blank=True, null=True)
    # role = models.CharField(
    #     db_column='role', max_length=45, blank=True, null=True)
    # department = models.OneToOneField('Department',
    #     db_column='department', max_length=45, on_delete=models.CASCADE)
    # role = models.ManyToManyField('Role',db_column='role')
    # # department = models.OneToOneField(Department, on_delete=models.CASCADE, blank=True, null=True)
    # # role = models.ManyToManyField(Role)

    date_joined = None
    first_name = None
    last_name = None

    REQUIRED_FIELDS = ['email', ]  # already set in abstract model

    def __str__(self):
        if self.name is not None:
            return self.name
        return self.username

    class Meta:
        verbose_name = "User"
        verbose_name_plural = "Users"
        db_table = 'user'
        unique_together = ('email',)


class CustomPermission(Permission):
    pass


class Department(models.Model):
    # id = models.PositiveSmallIntegerField(primary_key=True,editable=False)
    id = models.SmallAutoField(primary_key=True)
    department_name = models.CharField(
        db_column='department_name', max_length=45, unique=True)
    # user = models.OneToOneField(
    #     'CustomUser', on_delete=models.CASCADE, default=1)

    def __str__(self):
        return self.department_name

    class Meta:
        db_table = "department"
        verbose_name = _("Department")
        verbose_name_plural = _("Departments")


class Role(models.Model):
    id = models.SmallAutoField(primary_key=True)
    department = models.ForeignKey('Department', on_delete=models.CASCADE)
    user = models.ManyToManyField('CustomUser', through='UserRole')
    # user = models.ManyToManyField('CustomUser')
    role_name = models.CharField(db_column='role_name',
                                 max_length=45, unique=True)

    def __str__(self):
        return self.role_name

    class Meta:
        db_table = "role"
        verbose_name = _("Role")
        verbose_name_plural = _("Roles")


class UserRole(models.Model):
    id = models.SmallAutoField(primary_key=True)
    role = models.ForeignKey('Role',
                             on_delete=models.CASCADE)
    user = models.ForeignKey('CustomUser',
                             on_delete=models.CASCADE)

    def __str__(self):
        return self.role.role_name

    class Meta:
        db_table = "user_role"


class UserProfile(models.Model):
    id = models.SmallAutoField(primary_key=True)
    user = models.OneToOneField('CustomUser', on_delete=models.CASCADE)
    image = models.ImageField(upload_to="userprofile/%Y/%m/%d/",default='profile.png')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f'{self.user}'

    class Meta:
        verbose_name = _("User Profile")
        verbose_name_plural = _("User Profiles")
        db_table = 'user_profile'


# create the user profile is a new user created
def create_user_profile(sender, instance, created, **kwargs):

    if created:

        UserProfile.objects.create(user=instance)


# attached a post save signal to the user model
post_save.connect(create_user_profile, sender=CustomUser)
```

`forms.py`

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm
# from django.contrib.auth.models import User
from django.contrib.auth import get_user_model
from django import forms
from django.contrib.auth.forms import UserCreationForm, UserChangeForm, UsernameField, AuthenticationForm, PasswordResetForm
from django.core.exceptions import ValidationError
from django.contrib.admin.options import ModelAdmin
from django.contrib.auth.models import Permission
# from .models import CustomUser, GENDER_CHOICES
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth import authenticate
# from django.utils.translation import ugettext_lazy as _
from django.utils.translation import gettext_lazy as _
from users.models import UserProfile
# from django.conf import settings
# User = settings.AUTH_USER_MODEL
'''
class SignUpForm(UserCreationForm):
	username = forms.CharField(
		label='',
		max_length=30,
		min_length=5,
		required=True,
		widget=forms.TextInput(
			attrs={
				"placeholder": "Username",
				"class": "form-control"
			}
		)
	)

	password1 = forms.CharField(
		label='',
		max_length=30,
		min_length=8,
		required=True,
		widget=forms.PasswordInput(
			attrs={
				"placeholder": "Password",
				"class": "form-control"
			}
		)
	)

	password2 = forms.CharField(
		label='',
		max_length=30,
		min_length=8,
		required=True,
		widget=forms.PasswordInput(
			attrs={
				"placeholder": "Confirm Password",
				"class": "form-control"
			}
		)
	)

	class Meta:
		model = get_user_model()
		fields = ('username', 'name', 'email', 'password1', 'password2',)

'''


class CheckReq(forms.ModelForm):
    content_type = forms.ModelMultipleChoiceField(queryset=Permission.objects.all(),
                                                  required=False,
                                                  widget=forms.SelectMultiple(
        attrs={"class": "select2 form-control", "autocomplete": "off"}))
    aleady = forms.ModelMultipleChoiceField(queryset=Permission.objects.all(),
                                            required=False,
                                            widget=forms.SelectMultiple(
        attrs={"class": "select2 form-control", "autocomplete": "off"}))

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # print("intial sevalue", self.user)
        # print("intial arvalue", self.instance)
        # print("intial arvalue", kwargs)
        cuser = kwargs.get("instance")
        if kwargs.get("instance") is not None:

            self.fields['aleady'].queryset = cuser.user_permissions.all()

    class Meta:
        model = Permission
        # fields = ['__all__']
        fields = ["content_type", "aleady"]


class CustomUserCreationForm(UserCreationForm):
    # def __init__(self, *args, **kwargs):
    #     super(UserCreationForm, self).__init__(*args, **kwargs)
    #     self.fields['first_name'] = forms.CharField(required=True)
    #     self.fields['gender'] = forms.ChoiceField(choices=GENDER_CHOICES,
    #                                               widget=forms.RadioSelect,
    #
    #
    username = forms.CharField(
        label='',
        # max_length=30,
        # min_length=2,
        required=True,
        widget=forms.TextInput(
            attrs={
                "placeholder": "Username",
                "class": "form-control"
            }
        )
    )

    email = forms.EmailField(
        label='',
        # max_length=30,
        # min_length=5,
        required=True,
        widget=forms.EmailInput(
            attrs={
                "placeholder": "Email",
                "class": "form-control"
            }
        )
    )

    password1 = forms.CharField(
        label='',
        # max_length=30,
        # min_length=8,
        required=True,
        widget=forms.PasswordInput(
            attrs={
                "placeholder": "Password",
                "class": "form-control"
            }
        )
    )

    password2 = forms.CharField(
        label='',
        # max_length=30,
        # min_length=8,
        required=True,
        widget=forms.PasswordInput(
            attrs={
                "placeholder": "Confirm Password",
                "class": "form-control"
            }
        )
    )

    class Meta:
        model = get_user_model()
        fields = ('username', 'email', 'password1', 'password2',)


class CustomUserChangeForm(UserChangeForm):

    class Meta:
        model = get_user_model()
        fields = UserChangeForm.Meta.fields


class CustomAuthenticationForm(AuthenticationForm):
    def __init__(self, *args, **kwargs):
        super(CustomAuthenticationForm, self).__init__(*args, **kwargs)
        self.fields['username'].widget.attrs.update(
            {'placeholder': 'username/email', 'class': 'form-control', })
        self.fields['password'].widget.attrs.update(
            {'placeholder': 'passoword', 'class': 'form-control', })
        # for name in self.fields.keys():
        #     self.fields[name].widget.attrs.update({
        #         'class': 'form-control',
        #     })

    def clean(self):
        username = self.cleaned_data.get('username')
        password = self.cleaned_data.get('password')
        rem = self.request.POST.get('remember-me')
        # print(self.cleaned_data)
        # print(self.request.session.get_expiry_age())
        print("--------------rem------------", rem)

        if username is not None and password:
            self.user_cache = authenticate(
                self.request, username=username, password=password)

            if not self.request.POST.get('remember-me', None):
                print("session will expire")
                self.request.session.set_expiry(0)

            if self.user_cache is None:
                raise self.get_invalid_login_error()
            else:
                self.confirm_login_allowed(self.user_cache)

        return self.cleaned_data


'''
another example

class FriendForm(forms.ModelForm):
    # change the widget of the date field.
    dob = forms.DateField(
        label='What is your birth date?',
        # change the range of the years from 1980 to currentYear - 5
        widget=forms.SelectDateWidget(
            years=range(1980, datetime.date.today().year-5))
    )

    def __init__(self, *args, **kwargs):
        super(FriendForm, self).__init__(*args, **kwargs)
        # add a "form-control" class to each form input
        # for enabling bootstrap
        for name in self.fields.keys():
            self.fields[name].widget.attrs.update({
                'class': 'form-control',
            })

    class Meta:
        model = Friend
        fields = ("__all__"

'''


class CustomPasswordResetForm(PasswordResetForm):
    # def __init__(self, *args, **kwargs):
    #     super(CustomPasswordResetForm, self).__init__(*args, **kwargs)
    #     self.fields['email'].widget.attrs.update({'class': 'form-control'})

    def clean_email(self):
        email = self.cleaned_data.get('email')
        if not get_user_model().objects.filter(email__iexact=email, is_active=True).exists():
            msg = _(f"No user registered with the {email} E-Mail address.")
            self.add_error('email', msg)
        return email


class UserProfileForm(forms.ModelForm):
    prefix = 'profile'

    class Meta:
        model = UserProfile
        fields = '__all__'
        exclude = ('user',)
```




`views.py`


```python
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import Permission
from django.views.generic.edit import CreateView, View
from django.views.generic import DetailView, TemplateView, UpdateView
# from django.views.generic import DetailView
from django.contrib.auth import get_user_model
from django.contrib import messages
from ERPControlProject.mixins import NextUrlMixin
from .forms import CustomUserCreationForm, CustomAuthenticationForm, CustomPasswordResetForm, CheckReq
from django.contrib.messages.views import SuccessMessageMixin
from django.contrib.auth.views import LoginView, LogoutView, PasswordResetView
from django.http import HttpResponseRedirect, JsonResponse, HttpResponse
from django.urls import reverse_lazy
from django.contrib import messages
# from profiles.models import Setting
from django.core.exceptions import ObjectDoesNotExist
from django.contrib.auth.mixins import LoginRequiredMixin
from users.models import UserProfile
from users.forms import UserProfileForm
from erpapp.models import Note
from ERPControlProject.mixins import BaseContextMixin
'''

192.168.0.221
celec
12345
name of db crm



def signup(request):
	if request.method == "POST":
		form = UserCreationForm(request.POST)
		if form.is_valid():
			form.save()
			username = form.cleaned_data.get('username')
			password = form.cleaned_data.get('password1')
			user = authenticate(username=username, password=password)
			login(request, user)
			return redirect('home')
		else:
			form = UserCreationForm()

		return render(request, 'signup.html', {'form': form})

'''


# class DashboardView(LoginRequiredMixin,TemplateView):
#     # model = Setting
#     template_name = "dashboard.html"

#     def get_context_data(self, **kwargs):
#         # Call the base implementation first to get a context
#         context = super().get_context_data(**kwargs)
#         # Add in a QuerySet of all the books
#         try:
#             get_set_user = Setting.objects.get(user=self.request.user)
#             if get_set_user:
#                 context['object'] = get_set_user
#         except ObjectDoesNotExist:
#             pass
#         return context

class AddPerm(CreateView):
    form_class = CheckReq
    template_name = "addperm.html"
    # model = Permission
    # Sending user object to the form, to verify which fields to display/remove (depending on group)

    def get_form_kwargs(self):
        kwargs = super(AddPerm, self).get_form_kwargs()
        if self.request.user.is_authenticated:
            kwargs.update({'instance':  self.request.user})
    #     # print("keagr self",self.request.user)
    #     # print("keagrs",kwargs)
        return kwargs


class Permm(DetailView):
    model = get_user_model()
    template_name = "perm.html"
    # fields = ['username']
    # form_class = CheckReq
    fields = ["id", "name", "content_type", "codename", ]

    def get_object(self):
        obj = super().get_object()
        # Record the last accessed date
        obj = obj.user_permissions.all()
        # obj = obj.get_all_permissions()
        return obj

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["all_perms"] = Permission.objects.all()
        return context


class ComparisionDashboardView(TemplateView):
    template_name = "comparision-dashboard.html"


class SignUpView(NextUrlMixin, CreateView):
    form_class = CustomUserCreationForm
    success_url = '/'
    template_name = 'registration/signup.html'
    success_message = 'You have signed up successfully'
    default_next = '/'

    def form_valid(self, form):
        # save the new user first
        form.save()
        # get the username and password
        # authenticate user then login
        user = authenticate(
            username=form.cleaned_data['username'], password=form.cleaned_data['password1'], )
        login(self.request, user)
        messages.success(self.request, 'You have signed up successfully')
        next_path = self.get_next_url()
        return HttpResponseRedirect(next_path)


class UserLoginView(NextUrlMixin, SuccessMessageMixin, LoginView):
    form_class = CustomAuthenticationForm
    redirect_authenticated_user = True
    # template_name = 'registration/login3.html'


class UserLogoutView(NextUrlMixin, SuccessMessageMixin, LogoutView):
    pass


class CustomPasswordResetView(PasswordResetView):
    form_class = CustomPasswordResetForm


class UserProfileView(LoginRequiredMixin, BaseContextMixin, UpdateView):
    template_name = 'registration/profile_page.html'
    model = UserProfile
    form_class = UserProfileForm
    success_url = reverse_lazy('user-profile')


    def get_object(self, queryset=None):
        ''' overwrite the method so we not need to config url '''
        try:
            obj = self.model.objects.get(user=self.request.user)
            return obj
        except Exception as ex:
            print(ex)
            return

    def get_initial(self, *args, **kwargs):
        ''' initialize your's form values here '''
        base_initial = self.initial.copy()
        base_initial.update({'user': self.request.user})
        return base_initial

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['form'] = None
        context['profile_form'] = self.get_form()
        return context

    def form_valid(self, form, *args, **kwargs):
        if not form.instance.user:
            form.instance.user = self.request.user
        form.save()
        if self.request.is_ajax():
            return JsonResponse({'result': 'Success', 'content': 'Profile Updated'})
        return super().form_valid(form)

    def form_invalid(self, form):
        if self.request.is_ajax() and form.errors:
            errors = form.errors.as_json()
            return HttpResponse(errors, status=400, content_type='application/json')
        return super().form_invalid(form)

```


`urls.py`

```python
from django.contrib import admin
from django.urls import path, include
from .views import (SignUpView, UserLoginView, UserLogoutView, CustomPasswordResetView,
                    ComparisionDashboardView, Permm, AddPerm, UserProfileView)
urlpatterns = [
    path('addperm/', AddPerm.as_view(), name='addperm'),
    path('perm/<int:pk>', Permm.as_view(), name='perm'),
    path('signup/', SignUpView.as_view(), name='signup'),
    path('login/', UserLoginView.as_view(), name='login'),
    path('profile/', UserProfileView.as_view(), name='user-profile'),
    # path('dashboard/', DashboardView.as_view(), name='dashboard'),
    path('comparision/', ComparisionDashboardView.as_view(),
         name='comparision-dashboard'),
    path('logout/', UserLogoutView.as_view(), name='logout'),
    path('password_reset/', CustomPasswordResetView.as_view(), name='password_reset'),
    path('', include('django.contrib.auth.urls')),
]
```

`mixins.py`

```python
class NextUrlMixin(object):
    default_next = "/"

    def get_next_url(self):
        request = self.request
        next_ = request.GET.get('next')
        next_post = request.POST.get('next')
        redirect_path = next_ or next_post or None
        rdata = ''.join(redirect_path.split())
        redirect_path = rdata.strip()
        if is_safe_url(redirect_path, request.get_host()):
            print("save url", redirect_path)
            return redirect_path
        return self.default_next
        
```



`backends.py `

```python
from django.contrib.auth import get_user_model
from django.contrib.auth.backends import ModelBackend
from django.db.models import Q


class EmailOrUsernameModelBackend(ModelBackend):

    def authenticate(self, request, username=None, password=None, **kwargs):
        if username is None:
            username = kwargs.get(get_user_model().USERNAME_FIELD)
        if username is None or password is None:
            return
        try:
            # # user = get_user_model()._default_manager.get_by_natural_key(username)
            # user = get_user_model()._default_manager.get(
            #     Q(username__iexact=username) | Q(email__iexact=username))
            
            users = get_user_model()._default_manager.filter(
            Q(username__iexact=username) | Q(email__iexact=username))
            if users:
                for user in users:
                    if user.check_password(password):
                        return user
            else:
                return None
        except get_user_model().DoesNotExist:
            # Run the default password hasher once to reduce the timing
            # difference between an existing and a nonexistent user (#20760).
            get_user_model().set_password(password)
        else:
            if user.check_password(password) and self.user_can_authenticate(user):
                return user
```



`admin.py`

```python
from django.contrib import admin
from django.contrib.auth import get_user_model
# from django.contrib.auth.models import Permission, Group, PermissionsMixin
from .models import CustomUser, Department, Role, UserRole, UserProfile
from django.contrib.auth.admin import UserAdmin
from django.utils.translation import gettext, gettext_lazy as _


class RoleAdmin(admin.ModelAdmin):
    model = Role
    # list_display = ('role_name', 'department',)
    # list_display = ['__all__']


class UserRoleInline(admin.TabularInline):
    model = UserRole
    extra = 0


class CustomUserAdmin(UserAdmin):
    model = CustomUser
    list_display = ('username', 'email',  'is_staff',)
    # exclude = ('first_name', 'date_joined', 'last_name')
    inlines = [
        UserRoleInline
    ]
    fieldsets = (
        (None, {'fields': ('username', 'password')}),
        (_('Personal info'), {'fields': ('name', 'email',)}),
        (_('Permissions'), {
            'fields': ('is_active', 'is_staff', 'is_superuser', 'groups', 'user_permissions'),
        }),
        (_('Important dates'), {'fields': ('last_login',)}),
    )
    # the add_fieldsets allowed extra fields to be dispaly on user create form
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('username', 'email', 'password1', 'password2'),
        }),
    )


admin.site.register(Department)
admin.site.register(UserRole)
admin.site.register(Role, RoleAdmin)

admin.site.register(CustomUser, CustomUserAdmin)
admin.site.register(UserProfile)
```



#### in  your project settings.py
`settings.py`
```python

AUTH_USER_MODEL = 'users.CustomUser'  # new
AUTHENTICATION_BACKENDS = [
    'users.backends.EmailOrUsernameModelBackend', 'django.contrib.auth.backends.ModelBackend']
LOGIN_URL = 'login'
# LOGIN_URL_REDIRECT = '/lead/'

LOGIN_REDIRECT_URL = '/lead/'
# LOGOUT_URL = '/users/login'
LOGOUT_REDIRECT_URL = '/'
ACCOUNT_EMAIL_REQUIRED = True
```
