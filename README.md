# django-snippets



# if data insert or download in json

// change this format

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



// to this
// where searchapp is app name and India is model class name
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


// python manage.py dumpdata ConnectWithMYSQL.S1902000403 --indent 4 > table_data.json
// after dowloading json  replace the model identifier in json file
// python manage.py loaddata table_data.json --app ermapp.S1902000403




