# Zaawansowane aplikacje internetowe

## Read - Wyświetlanie filmów z bazy danych

<br>

Zawartość pliku *filmy/views.py*

<font size="2">

```python

from django.http import HttpResponse
from django.template import loader
from filmy.models import Film


def wszystkie(request):
    template = loader.get_template("filmy/wszystkie.html")
    wszystkie_filmy = Film.objects.all()
    context = {'wszystkie_filmy':wszystkie_filmy,}
    return HttpResponse(template.render(context, request))

def szczegoly(request,film_id):
    template = loader.get_template("filmy/szczegoly.html")
    film = Film.objects.get(id=film_id)
    context = {'film': film}
    return HttpResponse(template.render(context,request))

```

</font>

<br>

Zawartość pliku *wszystkie.html*

<font size="2">

```python

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Lista filmów w bazie danych</title>
</head>
<body>
<ul>
    {% for film in wszystkie_filmy %}
        <li><a href="/filmy/szczegoly/{{ film.id }}/">{{film.tytul}}</a></li>
    {% endfor %}
    </ul>
</body>
</html>

```

</font>

<br>

<br>

Zawartość pliku *szczegoly.html*

<font size="2">

```python

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Szczegoly filmu {{film.tytul}}</title>
</head>
<body>
<h3>
    Tytuł filmu: {{film.tytul}} </br>
    Rok produkcji: {{film.rok}} </br>
    Data premiery: {{film.premiera}}, </br>
    Opis: {{film.opis}}, </br>
    Punkty od widzów: {{film.imdb_pkts}} 
</h3>
</body>
</html>

```

</font>

<br>


## Create - Dodawanie nowego filmu do bazy danych

<br>

Dodawanie filmu do bazy danych wymaga przesłania informacji przez przeglądarkę. Niezbędny jest do tego formularz. Do aplikacji *filmy* należy dodać plik *forms.py*, w którym będą definiowane wykorzystywane w aplikacji formularze. Poniżej zaprezentowano zawartość pliku *filmy/forms.py*. Dokładniejsze informacje dotyczące formularzy w Django dostępne są <a href="https://docs.djangoproject.com/en/4.2/topics/forms/" target="_blank">tutaj</a>.

<br>

<summary>Zawartość pliku <b>filmy/forms.py</b>:</summary>

<br>

<font size="2">

```python
from django.forms import ModelForm
from filmy.models import Film


class FilmForm(ModelForm):
    class Meta:
        model = Film
        fields = ['tytul', 'rok', 'opis', 'premiera', 'imdb_pkts']
```

</font>

<br>

Fragment zawartości pliku *filmy/views.py* z funkcją *nowy* 

<font size="2">

```python
from django.shortcuts import render, redirect
from filmy.forms import FilmForm


def nowy(request):
    nowyform = FilmForm(request.POST or None)
    if nowyform.is_valid():
        nowyform.save()
        return redirect(wszystkie)
    return render(request, 'filmy/c.html', {'nowyform': nowyform})
```

</font>

<br>

i wskazujący na tę funkcję nowy URL w pliku *filmy/urls.py*:

<br>

<font size="2">

```python
from django.urls import path
from filmy.views import wszystkie, szczegoly, nowy


urlpatterns = [
    path('wszystkie/',wszystkie),
    path('szczegoly/<int:film_id>/',szczegoly),
    path('nowy/', nowy),
```

</font>

<br>

<summary>Zawartość plików <b>c.html, u.html</b>:</summary>

<br>

<font size="2">

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Tworzenie lub edycja filmu</title>
</head>
<body>
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ nowyform }}
    <button type="submit">Wyślij dane</button>
</form>
</body>
</html>
```

</font>

<br>

## Update - edycja filmu w bazie danych

<br>

Fragment zawartości pliku *filmy/views.py* z funkcją *edycja*

<br>

<font size="2">

```python

from django.shortcuts import get_object_or_404

def edycja(request, film_id):
    film = get_object_or_404(Film, pk=film_id)
    form = FilmForm(request.POST or None, instance=film)
    if form.is_valid():
        form.save()
        return redirect(wszystkie)
    return render(request, 'filmy/u.html', {'form':form})
```

</font>

<br>

i nowy URL w pliku *filmy/urls.py*:

<br>

<font size="2">

```python
from django.urls import path
from .views import wszystkie, szczegoly, nowy, edycja


urlpatterns = [
    path('wszystkie/',wszystkie),
    path('wszystkie/<int:film_id>/',szczegoly),
    path('nowy/', nowy'),
    path('edycja/<int:film_id>/', edycja),
]
```

</font>

<br>

## Delete

Fragment zawartości pliku *filmy/views.py* z funkcją *usun*

<br>

<font size="2">

```python
def usun(request, film_id):
    film = get_object_or_404(Film, pk=film_id)
    if request.method=="POST":
        film.delete()
        return redirect(wszystkie)
    return render(request, 'filmy/usun.html', {'film': film})
```

</font>

<br>

i nowy URL w pliku *filmy/urls.py*:

<br>

<font size="2">

```python
from django.urls import path
from filmy.views import wszystkie, szczegoly, nowy, edycja, usun


urlpatterns = [
    path('wszystkie/',wszystkie),
    path('wszystkie/<int:film_id>/',szczegoly),
    path('nowy/', nowy),
    path('edycja/<int:film_id>/', edycja),
    path('usun/<int:film_id>/', usun)
]
```

</font>

<br>

<details>

<summary>Zawartość pliku <b>usun.html</b>:</summary>

<font size="2">

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Usuwanie filmu</title>
</head>
<body>
<h1>Czy na pewno chcesz usunąć z bazy film {{ film.tytul }} ?</h1>
<form method="post">
    {% csrf_token %}
    <button type="submit">Usuń film</button>
</form>

</body>
</html>
```

</font>

</details>

<br>
