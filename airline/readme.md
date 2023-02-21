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

## Create new Passenger model
* Add Passenger class in models.py
```
  class Passenger(models.Model):
  first_name = models.CharField(max_length=64)
  last_name = models.CharField(max_length=64)
  flights = models.ManyToManyField(Flight, blank = True, related_name="passengers")


  def __str__(self):
    return f"{self.first_name} {self.last_name}"
```

* Perform migrations since new models were created **python manage.py makemigrations** and **python manage.py migrate**
* Register new Passenger model on **admin.py**
* Open /admin and create new passengers

## Add Passenger list on Flight view
* On **views.py** modify flight function to send passengers list to **flight.html**
```
  def flight(request, flight_id):
  flight = Flight.objects.get(pk=flight_id)
  return render(request, "flights/flight.html", {
    "flight": flight,
    "passengers": flight.passengers.all()
    })
```

```
<h2>Passengers</h2>
<ul>
    {% for passenger in passengers %}
    <li>{{ passenger}}</li>
    {% empty %}
    <li>No passengers</li>
    {% endfor %}
</ul>
```

* Add link to go between flights list and details, adding <a href> on html files
```
<a href="{% url 'index' %}">Back to Flight List</a>
```

```
    {% for flight in flights %}
      <li>
        <a href="{% url 'flight' flight.id %}">
        Flight {{ flight.id }} : {{ flight.origin }} to {{ flight.destination }}
          </a>
      </li>
    {% endfor %}
```

## Add feature to allow booking
* Add new path on **urls.py**
```
path("<int:flight_id>/book", views.book, name="book"),
```
* Add new book function on **views.py**, using reverse method which uses the name defined on **urls.py**
```
def book(request, flight_id):
    if request.method == "POST":
        flight = Flight.objects.get(pk=flight_id)
        passenger = Passenger.objects.get(pk=int(request.POST["passenger"]))
        passenger.flights.add(flight)
        return HttpResponseRedirect(reverse("flight", args=(flight.id,)))

```
* Add a form on flight.html to allow booking not already booked passengers into a flight
```
<h2>Add Passenger</h2>
<form action=" {% url 'book' flight.id %}" method="POST" >
    {% csrf_token %}
    <select name="passenger">
        {% for passenger in non_passengers %}
            <option value="{{ passenger.id }}">{{ passenger }}</option>
        {% endfor %}
    </select>
    <input type="submit">
</form>

```
* Modify flight function in **views.py** to include a list of not registered passengers in current flight using exclude method (opposite of filter)
```
def flight(request, flight_id):
    flight = Flight.objects.get(pk=flight_id)
    return render(request, "flights/flight.html", {
        "flight": flight,
        "passengers": flight.passengers.all(),
        "non_passengers": Passenger.objects.exclude(flights=flight).all()
    })
```
