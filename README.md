# django-snippets



# if data insert or download in json

// change this format
```json
[ 
{ 
"id":1,
"officename":"Chakragaon",
"pincode":744112,
"officetype":"S.O",
"deliverystatus":"Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
},
{ 
"id":2,
"officename":"Chatham ",
"pincode":744102,
"officetype":"S.O",
"deliverystatus":"Non-Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
},
{ 
"id":3,
"officename":"Delanipur ",
"pincode":744102,
"officetype":"S.O",
"deliverystatus":"Non-Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
}]

```


// to this
// where searchapp is app name and India is model class name
```json
[ 
{
    "model": "searchapp.India",
    "pk":1,
    "fields": {
        "officename":"Chakragaon",
"pincode":744112,
"officetype":"S.O",
"deliverystatus":"Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
}
},
{
    "model": "searchapp.India",
    "pk":2,
    "fields": {
        "officename":"Chatham ",
"pincode":744102,
"officetype":"S.O",
"deliverystatus":"Non-Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
}
},
{
    "model": "searchapp.India",
    "pk":3,
    "fields": {
        "officename":"Delanipur ",
"pincode":744102,
"officetype":"S.O",
"deliverystatus":"Non-Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
}
},
{
    "model": "searchapp.India",
    "pk":4,
    "fields": {
        "officename":"Marine Jetty ",
"pincode":744101,
"officetype":"S.O",
"deliverystatus":"Non-Delivery",
"divisionname":"A - N Islands",
"regionname":"Calcutta HQ",
"circlename":"West Bengal",
"taluk":"Portblair",
"districtname":"South Andaman",
"statename":"ANDAMAN & NICOBAR ISLANDS",
"stateshort":"AN"
}
}
]

```
### If you are connected to database via django then run this to download data form database

### here ConnectWithMYSQL is the app name and S1902000403 is the model name

```shell
python manage.py dumpdata ConnectWithMYSQL.S1902000403 --indent 4 > table_data.json
```




## after dowloading json  replace the model identifier in json file
### loading data from json to database using django
### here ermapp is the app name and S1902000403 is the model name

```shell
python manage.py loaddata table_data.json --app ermapp.S1902000403
```


### if you are connected to database you can quicky genereted the models.py of already running database tabels
### just run inspectdb
it create models by introspecting an existing database

```shell
python manage.py inspectdb > models.py
```
it create models by introspecting an existing database




### Get instance of django form
```python
instance = getattr(self, 'instance', None)
        if instance and instance.pk:
            return instance.lead
```
### IF there is form error and want it in ajax
```python
errors = note_form.errors.as_json()
return HttpResponse(errors, status=400, content_type='application/json')
```


### The Django admin documentation generator
https://docs.djangoproject.com/en/3.1/ref/contrib/admin/admindocs/

Add `django.contrib.admindocs` to your INSTALLED_APPS.

before /admin/
```python
path('admin/doc/', include('django.contrib.admindocs.urls')) 
```
```python
pip install docutils
```

Now you have a  ` DOCUMENTATION ` in admin panel
