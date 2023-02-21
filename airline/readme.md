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