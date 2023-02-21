# CS50 - SQL, Models, and Migrations Lecture

## Initial Procedure
* django-admin startproject airline
* python manage.py startapp flights
* add 'flights' on INSTALLED_APPS
* add "flights/" on urlpatterns
* create **urls.py** inside flights an add urlpatterns array

## Create Models
* Add classes on **models.py**
* Create database according models by using migration
  * First create the migration (the instructions)
  * Then migrate (use the instructions)
  ```
  python manage.py makemigrations
  #check migrations/0001_initial.py file
  python manage.py migrate
  #check db.sqlite3
  ```
* Enter django shell **python manage.py shell** 
  ```
  from flights.models import Flight
  f = Flight(origin="New York", destination="London", duration ' 415)
  f.save()
  Flight.objects.all()
  ```
  
* Implement __str__ funciton on model
  ```
  def __str__(self):
    return f"{self.id}: {self.origin} to {self.desitnation}"
  ```
  
* Reenter django shell and check flights
  ```
  from flight.models import Flight
  flights = Flight.objects.all()
  flights
  flight = flights.first()
  fligth.id
  flight.destination
  flight.duration
  flight.delete()
  
  ```
  
### Create a new model to relate to existing model
  ```
  class Airport(models.Model):
    code = models.CharField(max_length=3)
    city = models.CharField(max_length=64)

    def __str__(self):
        return f"{self.city} ({self.code})"


class Flight(models.Model):
    origin = models.ForeignKey(Airport, on_delete=models.CASCADE(), related_name="departures")
    destination = models.ForeignKey(Airport, on_delete=models.CASCADE(), related_name="arrivals")
    duration = models.IntegerField()
  ```
* Update models using **makemigrations** and **migrate** commands
* Enter python manage.py shell
```
  from flights.models import *
  jfk = Airport(code="JFK", city ="New York")
  jfk.save()
  lhr = Airport(code="LHR", city="London")
  lhr.save()
  cdg = Airport(code="CDG", city="Paris")
  cdg.save()
  nrt = Airport(code="NRT", city="Tokyo")
  nrt.save()
  
  f = Flight(origin=jfk, destination = lhr, duration = 415)
  f.save()
  f
  f.origin
  f.origin.city
  
  lhr.arrivals.all()
  jfk.departures.all()
```

## Create views to display list of flights
* In **ulrs.py** add
```
  urlpatterns =[
    path("", views.index, name="index"),
]
```
* In **views.py** add index function 
```
  def index(request):
    return render(request, "flights/index.html", {
        "flights": Flight.objects.all()
    })
```
* Create folders **templates/flights** and add **layout.html**
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flights</title>
</head>
<body>
    {% block body %}
    {% endblock %}
</body>
</html>
```
* Add **index.html**
```
{% extends "flights/layout.html" %}

{% block body %}
  <h1>Flights</h1>
  <ul>
    {% for flight in flights %}
      <li>Flight {{ flight.id }} : {{ flight.origin }} to {{ flight.destination }} </li>
    {% endfor %}
  </ul>
{% endblock %}
```

* Enter Shell **python manage.py shell** to create new airports and flights
```
  from flights.models import *
  Airport.objects.filter(city="New York")
  Airport.objects.filter(city="New York").first()
  Airport.objects.get(city="New York") #only works when only one meeting criteria exists 
  
  jfk = Airport.objects.get(city="New York")
  cdg = Airport.objects.get(city="Paris")
  f = Flight(origin = jfk, destination = cdg, duration = 435)
  f.save() 
```
## Use Django Admin App
* Run **python manage.py createsuperuser**
* Add models on **admin.py**
```
from .models import Flight, Airport

admin.site.register(Airport)
admin.site.register(Flight)
```

* Start server, open /admin site and create new Airports and Flights

* Create new url to view flights details
  * add new path **urls.py**, function **views.py** and html template **flight.html**

```
urlpatterns =[
    path("", views.index, name="index"),
    path("<int:flight_id>", views.flight, name="flight"),
]
```

```
def flight(request, flight_id):
    # flight = Flight.objects.get(id=flight_id)
    flight = Flight.objects.get(pk=flight_id)
    return render(request, "flights/flight.html", {
        "flight": flight
    })
```

```
{% extends "flights/layout.html" %}

{% block body %}
  <h1>Flight {{flight.id}}</h1>
  <ul>
    <li>Origin: {{ flight.origin }}</li>
    <li>Destination: {{ flight.destination }}</li>
    <li>Duration: {{ flight.duration }}</li>
  </ul>
{% endblock %}
```