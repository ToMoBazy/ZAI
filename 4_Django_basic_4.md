# Zaawansowane aplikacje internetowe

<br>

## Modele

<br>

Informacje o utworzonych modelach przechowywane są w pliku *filmy/models.py*. Każdy model to osobna tabela w bazie danych. Poniżej przedstawiona została zawartość pliku *filmy/models.py*. Składają się na nią cztery modele, nie połączone ze sobą (na razie) żadnymi relacjami.

<br>

<details>

<summary>Plik <b>filmy/models.py</b>:</summary>

<font size="2">

```python
from django.db import models


class Film(models.Model):
    tytul = models.CharField(max_length=64, blank=False, unique=True)
    rok = models.PositiveSmallIntegerField(blank=False)
    opis = models.TextField(default="")
    premiera = models.DateField(null=True, blank=True)
    imdb_pkts = models.DecimalField(max_digits=4, decimal_places=2, null=True, blank=True)
    

    def __str__(self):
        return "{} ({})".format(self.tytul, str(self.rok))


class ExtraInfo(models.Model):
    GATUNEK = {
        (0, 'Inne'),
        (1, 'Horror'),
        (2, 'Komedia'),
        (3, 'Sci-fi'),
        (4, 'Dramat')
    }
    czas_trwania = models.PositiveSmallIntegerField(null=True, blank=True)
    gatunek = models.PositiveSmallIntegerField(choices=GATUNEK,null=True, blank=True)
    rezyser = models.CharField(max_length=50, blank=True, null=True)
    
    
class Ocena(models.Model):
    recenzja = models.TextField(default="", blank=True)
    gwiazdki = models.PositiveSmallIntegerField(default=5)
    

class Aktor(models.Model):
    imie = models.CharField(max_length=32)
    nazwisko = models.CharField(max_length=32)
```

</font>

</details>

<br>

## Relacje

<br>

*Django* obsługuje relacje *1-to-1*, *many-to-1* oraz *many-to-many*. Poniżej zaprezentowano przykłady relacji między poszczególnymi modelami.

<br>

### Relacja *1-to-1*

<br>

Fragment pliku *filmy/models.py*:

<font size="2">

```python
( ... )

class ExtraInfo(models.Model):
    GATUNEK = {
        (0, 'Inne'),
        (1, 'Horror'),
        (2, 'Komedia'),
        (3, 'Sci-fi'),
        (4, 'Dramat')
    }
    czas_trwania = models.PositiveSmallIntegerField(null=True, blank=True)
    gatunek = models.PositiveSmallIntegerField(choices=GATUNEK,null=True, blank=True)
    punkty_widzow = models.PositiveSmallIntegerField(default=0)
    film = models.OneToOneField(Film, on_delete=models.CASCADE, null=True, blank=True)  
    
    # <-- relacja 1-to-1 z modelem 'Film'
    # Po ustaleniu relacji z modelem Film możliwa jest np. następująca 
    # zmiana reprezentacji tekstowej modelu ExtraInfo

    def __str__(self):
        for g in list(self.GATUNEK):
            if g[0] == self.gatunek:
                gok = g[1]
        return "Film: {}, gatunek: {}, czas trwania: {}, punkty od widzów: {}".format(self.film.tytul, 
                gok, self.czas_trwania, self.punkty_widzow)
( ... )
```

</font>

<br>

Dodatkowe informacje związane z filmem można przywołać korzystając z atrybutu *film.extrainfo*. 

<br>

###  Relacja *Many-to-1*

<br>

Fragment pliku *filmy/models.py*:

<font size="2">

```python
( ... )
class Ocena(models.Model):
    recenzja = models.TextField(default="", blank=True)
    gwiazdki = models.PositiveSmallIntegerField(default=5)
    film = models.ForeignKey(Film, on_delete=models.CASCADE)        
    
    # <-- relacja many-to-1 z modelem 'Film'
    # Po ustaleniu relacji z modelem Film możliwa jest np. następująca 
    # zmiana reprezentacji tekstowej modelu Ocena

    def __str__(self):
        rec = self.recenzja[:20] + ' ...'
        return "Film: {}, gwiazdki: {}, recenzja: {}".format(self.film.tytul, str(self.gwiazdki), rec)
( ... )
```

</font>

<br>

Oceny związane z filmem można przywołać korzystając z atrybutu *film.ocena_set.all()*. 

<br>

### Relacja *Many-to-many*

<br>

Fragment pliku *filmy/models.py*:

<font size="2">

```python
( ... )
class Aktor(models.Model):
    imie = models.CharField(max_length=32)
    nazwisko = models.CharField(max_length=32)
    filmy = models.ManyToManyField(Film)                        
    
    # <-- relacja many-to-many z modelem 'Film'
    # Po ustaleniu relacji z modelem Film możliwa jest np. 
    # następująca zmiana reprezentacji tekstowej modelu Aktor

    def __str__(self):
        return "{} {}, gra w {} filmach z bazy danych".format(self.imie, self.nazwisko, str(self.filmy.count()))
( ... )
```

</font>

<br>

Aktorów związanych z filmem można przywołać korzystając z atrybutu *film.aktor_set.all()*;<br> 
Filmy przypisane do konkretnego aktora można przywołać korzystając z atrybutu *aktor.filmy.all()*. 

<br>

### Modele - dokonywanie zmian i konieczność aktualizacji bazy danych

<br>

Po dokonaniu jakichkolwiek zmian w modelach należy uruchomić migracje, w celu aktualizacji bazy danych:

<br>

<font size="2">

```python
(env) D:\ZAI\PROJEKTY\NSTC111111>python manage.py makemigrations
Migrations for 'filmy':
  filmy\migrations\0013_alter_extrainfo_gatunek.py
    - Alter field gatunek on extrainfo

(env) D:\ZAI\PROJEKTY\NSTC111111>python manage.py migrate       
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, filmy, sessions
Running migrations:
  Applying filmy.0013_alter_extrainfo_gatunek... OK

(env) D:\ZAI\PROJEKTY\NSTC111111>

```

</font>

<br>

## Admin

<br>

*Admin* jest wbudowaną plikacją, dostępną zawsze po utworzeniu projektu i aplikacji we frameworku *Django*. Powinien być wykorzystywany wyłącznie przez grupę upoważnionych użytkowników do wykonywania czynności administracyjnych w tworzonej aplikacji. Dostęp do aplikacji *Admin* jest zapewniony poprzez zawartość pliku *filmweb/urls.py*:

<br>

<font size="2">

```python
from django.contrib import admin

urlpatterns = [
    path('admin/', admin.site.urls),
]
```

</font>

<br>

### Dodawanie modelu do aplikacji *Admin*

<br>

Najprostszym sposobem jest rejestracja modelu, dokonana w pliku *filmy/admin.py*:

<br>

<font size="2">

```python
from django.contrib import admin
from .models import Film, ExtraInfo, Ocena, Aktor


admin.site.register(Film)
admin.site.register(ExtraInfo)
admin.site.register(Ocena)
admin.site.register(Aktor)
```

</font>

<br>

Uzyskujemy wówczas w aplikacji *Admin* możliwość oglądania i edycji poszczególnych modeli: modele *ExtraInfo*, *Ocena* i *Aktor* zawierają pola, w których widać film, z którym są związane, natomiast model *Film* nie zawiera żadnych pól z pozostałych modeli, mimo, że wiążą go z nimi odpowiednie relacje. <br>
Poniżej zaprezentowano aktualną zawartość pliku *filmy/admin.py*, pozwalającą na wyświetlenie i edycję modelu *Film* wraz z powiązanymi z nim pozostałymi modelami:

<br>

<details>

<summary>Zawartość pliku <b>filmy/admin.py</b>:</summary>

<font size="2">

```python
from django.contrib import admin
from .models import Film, ExtraInfo, Ocena, Aktor


admin.site.register(Aktor)

class ExtraInfoInline(admin.TabularInline):
    model = ExtraInfo

class OcenaInline(admin.TabularInline):
    model = Ocena
    extra = 0

class AktorInline(admin.TabularInline):
    model = Aktor.filmy.through
    extra = 0

@admin.register(Film)
class FilmAdmin(admin.ModelAdmin):
    inlines = [ExtraInfoInline, OcenaInline, AktorInline]
```

</font>

</details>

<br>

