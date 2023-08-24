
# Django-Biblioteca online
<p>O biblioteca online ce permite achizitia de carti online </p>

<img src="https://github.com/ClaudiuT03/Book_Store.git">

### Aplicatie live
* Puteti accesa site-ul direct din link urmator: <a href="https://dj-bookstore.onrender.com/" target="_blank" >Deployed App</a>

(Nota: pentru ca este un site gazduit pe o platforma gratuita, incarcarea siteului poate dura pana la 30 de secunde, insa aplicatia poate fi testat si pe plan local
).

# Cuprins
- [About_this_App](#About_this_App)
- [Get_Started](#Get_Started)
- [Books_App](#Books_App)
  * [models](#models)
  * [migrations](#migrations)
  * [admin](#admin)
  * [server](#server)
  * [views](#views)
  * [urls](#urls)
  * [templates](#templates)
  * [logins](#logins)
- [Accounts_App](#Accounts_App)
  * [signup](#signup)
  * [signup_view](#signup_view)
  * [static_files](#static_files) 
  
<hr>

## Despre aplicatie
Aceasta librarie online contine mai multe carti care pot fi achizitionate prin intermediul a 2 modalitati de plata: paypal si cu cardul de debit. De asemenea carti pot fi cumparate doar daca userul este autentificat.

De asemenea, inainte de a finaliza o comanda, veti fi directionati catre pagina de autentificare. Aplicatia va oferi informatii si despre disponibilitatea cartiilor pe stoc.

## Get_Started

 Pentru ca aplicatia sa functioneze, este necesar sa parcugem cativa pasi:


* Pregatirea spatiului virtual

$`pipenv shell`

$`pipenv install django==4.2.4`

## Books_App

Pentru a porni aplicatia, tastati in terminal urmatoarele comenzi:

(django_project)$`django-admin startproject ecom_project .` 

(django_project)$`python manage.py startapp books`

IDE-ul folosit pentru rularea aplicatiei este PyCharm.



### models

Aici se va regasit codul creata cu atributele pentru fiecare carte in parte:


	from django.db import models
	from django.urls import reverse

	class Book(models.Model):
	    title  = models.CharField(max_length = 200)
	    author = models.CharField(max_length = 200)
	    description = models.CharField(max_length = 500, default=None)
	    price = models.FloatField(null=True, blank=True)
	    image_url = models.CharField(max_length = 2083, default=False)
	    follow_author = models.CharField(max_length=2083, blank=True)  
	    book_available = models.BooleanField(default=False)

	    def __str__(self):
		return self.title


	class Order(models.Model):
		product = models.ForeignKey(Book, max_length=200, null=True, blank=True, on_delete = models.SET_NULL)
		created =  models.DateTimeField(auto_now_add=True) 

		def __str__(self):
			return self.product.title



 
Modelul "Book" contine 7 atribute, deorece atunci cand cumparam o carte, acestea pot fi identificate dupa mai multe criterii, fie dupa titlul cartii "title", autor "author", pret "price". Totodata se regaseste si o scurta descriere a cartii "description", coperta acesteia "image_url" pentru a fi identificata cu usurinta de catre cumparator, o referinta despre autor "follow_author" si daca acea carte este sau nu in stoc "book_available". Acest lucru permite administratorului sa actualizeze permanent biblioteca online. Daca este setata la True "cartea este disponibila", iar daca este setata la False este "stoc epuizat".

Modelul "Order" are 2 atribute "product" si "created" care inregistreaza momentul in care se face comanda.

Ambele modele folosesc def__str__(self) pentru titlu, iar acest string inseamna ca pe pagina de administrare pentru ambele modele, va fi afisat titlul cartii in loc de un id.

## migrations 

Pentru a putea face migrarile este nevoie sa rulam urmatoarele comenzi:

(django_project)$`python manage.py makemigrations`

(django_project)$`python manage.py migrate`

 Aceasta comanda ne spune ce modificari vor fi facute in baza noastra de date. Putem vedea mai multe informatii desre exectuariea migrarilor in urmatorul link <a href="https://docs.djangoproject.com/en/3.0/topics/migrations/">django documentation</a>


### admin

now we need Am inregistrat modelele in fisierul admin.py pentru a putea fi folosite

	from django.contrib import admin
	from .models import Book, Order

	admin.site.register(Book)
	admin.site.register(Order)


 Libraria .models importam modelele "Book" si "Order", din fisierul Models.py 
si pentru ca fiecare model sa se inregistreze folosim comanda --> admin.site.register(model_name)

### server

Pentru a ne asigura ca serverul functioneaza corect folosim urmatoarea comanda:

(django_project)$`python manage.py runserver`

* accesam acest ip dintr-un browser http://127.0.0.1:8000/

Va fi afisat mesajul 'The install worked successfully! Congratulations!' care ne va confirma ca serverul este functional.


* accesam acest link http://127.0.0.1:8000/admin/

Chiar sub User vor fi afisate cartie pe care le are biblioteca online si de asemenea si modelele "Book" si "Orders".
Apasand click pe modelul "Books", vom fi directionati pe pagina unde vom primi mesajul 'Select a Book to change' iar in coltul din dreapta se regaseste butonul "ADD BOOK +", apasati click si veti fi directionati catre pagina care contine toate atributele modelului cartii selectate.
 

### views

In "views" am creat mai mai multe clase de obiecte:

	from django.shortcuts import render 
	from django.views.generic import ListView, DetailView
	from django.views.generic.edit import CreateView
	from .models import Book, Order
	from django.urls import reverse_lazy
	from django.db.models import Q # for search method
	from django.http import JsonResponse
	import json



	class BooksListView(ListView):
	    model = Book
	    template_name = 'list.html'


	class BooksDetailView(DetailView):
	    model = Book
	    template_name = 'detail.html'


	class SearchResultsListView(ListView):
		model = Book
		template_name = 'search_results.html'

		def get_queryset(self): # new
			query = self.request.GET.get('q')
			return Book.objects.filter(
			Q(title__icontains=query) | Q(author__icontains=query)
			)

	class BookCheckoutView(DetailView):
	    model = Book
	    template_name = 'checkout.html'


	def paymentComplete(request):
		body = json.loads(request.body)
		print('BODY:', body)
		product = Book.objects.get(id=body['productId'])
		Order.objects.create(
			product=product
		)
		return JsonResponse('Payment completed!', safe=False)


* Clasa 'BookListView'  este o clasa care utilizeaza module django ListView pentru a afisa continutul cartii si este legata de fisierul list.html.
  
* Clasa 'BookDetailView'  utilizeaza DelailView  pentru a afisa detailiile carti, acestas fiind legata de fisierul detail.html. 

* Clasa 'SearchResultsView' utilizeaza ListView ofera rezultatele cautarii in baza de date si este legata de fisierul
 search_results.html. SearchResultsView va ajuta la asocierea cuvintelor introduse cu titlul cartii si numele autorului (cartea va putea fi cautat doar dupa titlu sau numele autorului).

* Clasa 'BookCheckoutView' utilizeaza DetailView  si este legat de fisierul checkout.html. in acest fel utilizatorul confirma ca plateste pentru cartea selectata.

* In cele din urma, avem Clasa paymentComplete care pastreaza inregistrarea cartii achizitionatede utilizator iar aceasta inregistrare va fi pastrata in modelul nostru. Plata se poate face prin paypal sau car de debit


### urls

 In fisierul urls.py  regasim locatia sau URL-ul pe care pagina web va functiona.


Importam 'path' din libraria django.url pe care il vom folosi pentru rutarea adreselor URL. 



### templates

In templates am creat 5 fisiere html

 
* base.html contine bara de navigare pentru site.
* list.html ofera lista tuturor cartilor.
* detail.html ofera detalii precum titlurile cartilor, numele autorilor, descrierile si detaliile tuturor campurilor prezente.
* checkout.html  ofera detalii despre carea pe care ati selectat-o pentru a fi cumparata si ofera cele 2 modalitati de plata--> paypal card de debit.
* search_results ofera rezultatele cautarilor prin potrivirea cuvintelor introduse de catre utilizatori (furnizate in baza de date) cu titlul cartii si numele autorului.

### logins


	from django.contrib.auth.mixins import LoginRequiredMixin 

	class BookCheckoutView(LoginRequiredMixin, DetailView):
	    model = Book
	    template_name = 'checkout.html'
	    login_url     = 'login'

In clasa BookCheckoutView se asigura faptul ca inainte de finalizarea comenzii, utilizatorul trebuie sa se autentifice ( va trebuie sa aiba cont pe site) iar in cazul in care acesta nu are cont creat, va fi directionat catre pagina ce contine doua optiuni: Log In si Sing Up.



## Accounts_App


### signup

In sectiunea de inregistrare, utilizatorul in poate crea cont pentru a putea face comanda de carti. Dupa crearea contului, utilizatorul va fi redirectionat catre pagina de autentificare.

