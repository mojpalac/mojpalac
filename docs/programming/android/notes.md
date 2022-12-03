### Activity vs Fragment

"You may want to consider using the new Fragment APIs to replace this arbitrary stack of activities with a single activity that more tightly manages its memory. For example if you use the back stack facilities of fragments, when a fragment goes on the back stack and is no longer visible, its onDestroyView() method is called to completely remove its view hierarchy, greatly reducing its footprint.
https://stackoverflow.com/questions/7536988/android-app-out-of-memory-issues-tried-everything-and-still-at-a-loss/7576275#7576275"


Activity itself is never killed by the system, system kills whole process (all activities)

LIFECYCLE
- you can extend from class LifecycleObserver
- pass Lifecycle as paramter, later call: lifecycle.addObserver(this-LifecycleObserver)
- add annotations to methods to be called  @OnLifecycleEvent(Lifecycle.event.ON_XXX)

LIVEDATA
- always val,
- backing field/property poniewz view powinien miec mozliwosc obserwowania livedata, to livedata nie moze byc private, ale jesli jest mutableLiveData wtedy view moze zmienic wartosc, a tak nie powinno byc! z tego wzgledu wprowadza sie backing property, ktora powoduje ze wartosc zmienna zaczyna sie z podkreslnikime i ustawia jako private np. private val _score, natomiast widoczna LiveData dla view bedzie val score: LiveData<Int> get() = _score tym samym view moze obserowowac wartosc, ale nie moze jeje zmieniac
- EASY BUG TO DO - obserowanie state moze powodowac, ze po rotacji otrzymamy kolejny raz state, a powinnismy go dostac tylko raz


### JobScheduler 

create a class that extends JobService

override:
onStartJob -> if returns true then it will continue the the live of the job, false destroys the job. If you want to destroy it on your own call `jobFinished(...)`
onStopJob -> if returns true, the job will be redone

### Room

1. utworzenie entity:
data klasa ktora ma adnotacje @Entity, jest obiektem ktory chcemy zapisac, musi miec @PrimaryKey

2. utworzenie Data Access Object (DAO):
klasa abstrakcyjna lub interface, w ktorej wykonujemy operacje na bazie danych -> tworzymy poleceniea poprzez utworzenie metody z adnotacja, jak:

@Insert(onConflict = OnConflictStrategy.IGNORE)
fun insert(person: Person)
albo:
@Query("SELECT * FROM Person ORDER BY surname ASC")
fun getAll(): LiveData<List<Person>>


3. Utworzenie bazydanych Room
3.1. klasa dziedziczy po RoomDatabase, musi miec adnotacje @Database(entities = [tutajEntites], version = x, exportSchema = true/false)
exportSchema - pozwala zachowac w pamiec jak wygladala tabela w danej wersji
3.2. klasa  abstrakcyjna
3.3. nalezy utworzyc abstrakcyjne funkcje zwracajace kazde dao
3.4. obiekt ma byc singletonem, ktorego nalezy utworzyc przy wykorzystaniu synchronized, jak robi sie poprzez asyncTask nalezy dodac tez callback

4. Utworzenie Repository, ktore zbiera wszystkie zrodla pozyskiwania tej samej iformacji (chmura, bazadanych)
4.1. tutaj inicjalizujemy RoomDatabase
4.2. bierzemy z rooma dao za pomoca ktorego komonikujemy z roomem
4.3. dodajemy obiekty do rooma za pomoca asyntaska

5. Tworzymy viewmodel w ktorym tworzymy repo obiekt i z ktorego bierzemy dane

### Paging

PageLibrary Full version	
1. Stworz klase ktora dziedziczy po DataSource (Generalnie sa 3 popularne klasy po ktorych sie dziedziczy:	
	1. PageKeyedDataSource - dzielimy zrodlo na strony, np. kazda strona ma 10 elementow, strona jest umowna, chodzi o to by wiedziec ze 10 elemnotw tworzy 1 wiekszy element = strona)
	2. ItemKeyDataSource - ladowanie po itemie tzn, jesli jest cos dla niego specyficznego, i sa uszeregowane to mozna wykorzystac to do ladowania
	3. PositionalDataSource - kiedy wiadomo jaka jest wielkosc listy)
	
W przypadku PageKeyedDataSource<Key, Value> mozna uzyc <Int, Item> do tworzenia listy, gdzie Int odpowiada numerowi stronie, a item to dane ktore nas interesuja,	
3 metody do nadpisania: loadBefore - w przypadku requestu bezporesednio do internetu, nie potrzeba implementowac	
loadInitial - pierwsze ladowanie danych, request w repo(dobrze jest przekazac jako parametr params.requestedLoadSize )  ktory nastepnie przekazujemy do callback.onResult()	
loadAfter - nastepne ladowania danych jw.	
istotne jest utworzenie pol	
	val currentPage = 1
val nextPage = currentPage + 1, ktore przekazujemy do callbacku i w loadAfter() przed kazdym requestem nadpisujemy( currentPage = params.key( bo key jest item== current page), nextpage = currentPage + 1)	
	
2. Utworzenie klasy ktora dziedziczy po DataSource.Factory<Key, Item> <parametry takie same jak dla DataSource>	
utworzenie pola np. source ktory jest MutableLiveData<MyDataSource>()	
nadpisujemy onCreate tworzymy tam zmienna lokalna MyDataSource() a jej wartosc postujemy do source liveData, zwracamy zmienna lokalna	
	
3. Utworzenie metody w repo ktora zwraca LiveData<PagedList<Item>> poprzez utworzenie LivePageDataBuilder.build()	
4. w aktywnosci zmieniamy z setowania listy do adaptera na uzywanie adapter.submitList(lista)	
5. w Adapterze zmieniamy z dziedziczenia na PagedListAdapter<Item, ThisAdapter.MyViewHolder>(DIFF_CALLBACK)	
6. (DIFF_CALLBACK) to DiffUtil.ItemCallback<Item> ktory ma 2 metody na sprawdzanie itemow (zazwyczaj po ID) i kontentu - porownanie obiektow do siebie (itemOld == itemNew)	
7. nie nadpisujemy getItemCount	
8. na onBindViewHolder uzywamy getItem() bo nie ma pola dla listy!	

PAGING Library in Room

1. w dao object zwracamy DataSource.Factory<Int, Person> zamiast livedata
2. ta sama zmiana w repo
3. w viewModel tworzymy livedata poprzez wywowalnie na obiekcie z repo .toLiveData(int pageSize)
4. w aktywnosci zmieniamy z setowania listy do adaptera na uzywanie adapter.submitList(lista)
5. w Adapterze zmieniamy z dziedziczenia na PagedListAdapter<Item, ThisAdapter.MyViewHolder>(DIFF_CALLBACK)
6. (DIFF_CALLBACK) to DiffUtil.ItemCallback<Item> ktory ma 2 metody na sprawdzanie itemow (zazwyczaj po ID) i kontentu - porownanie obiektow do siebie (itemOld == itemNew)
7. nie nadpisujemy getItemCount
8. na onBindViewHolder uzywamy getItem() bo nie ma pola dla listy!

### Loaders
Why not use loaders:
https://medium.com/inloopx/its-time-to-ditch-loaders-in-android-6492616775f7

### Dagger

DAGGER2
Dependancy Injection - tworzenie obiektow poprzez dodatnie do konstrukora obiektow zaleznych, a nie tworzenie ich lokalnie
Dagger2 sklada sie z nastepujacych elementow:
- Module - klasa z adnotacja @Module, metody ktore dostarczaja obiekty z adnotacja @Provides i czesto @Singleton - jezeli obiekt jest ciezki w inicjializacji i moze byc uzyty ponownie,
- Components - klej pomiedzy modulami a @Inject, to interfacae, ktory oznaczamy @Component(modules = {zaleznyModul1.class, zaleznyModul2.class}) metody interfacu maja za zadanie dostarczyc obiekty z modulow. Komponenty sa swiadome zakresu obiketow w module i jesli obiekty sa singletom mozna onaczyc kompent aby rowniez byc singletonem
- @Inject - adnotacja ktora umozliwia dostarczenie przez Dagger2 potrzebnych obiektow, dodatkowo jezeli klasa X ktora sama jest tworzona w module i jej parameter Y jest tworzony w module, to ostatecznie nie musi byc tworzona w module :P Poniewaz dagger2 potrafi znalezc zaleznosci. W takim przypadku wystarczy oznaczyc konstrukor jako  class X @Inject constructor(y: Y) , nalezy tez pamietac o tym by klasa wtedy byla @Singleton (jesli w module tak bylo) i nalezy usunac metody z modulu.
na metodach @Inject uzywamy TYLKO aby zastapic this (tak powiedzial Jack Wharton)
By dostac obiekty tworzymy jak najszybsciej Component w kodzie np. w klasie Aplikacji za pomoca DaggerNameOfComponent.builder()... .build() wtedy mamy dostep do metod z componentu

### Picasso
mozna uzywac posredniego serwera (np. thumbor) do stwierdzania jakiej wielkosci ma byc obrazek, wtedy dostajemy gotowej wielkosci obraz -> nie trzeba go zmniejszac na telefonie -> oszczednoscc CPU i baterii

### THREADING
AsyncTask nie moze byc inner class UIControllerze bo bedzie trzymal referencje

StrickMode - pozwala na wykrycie operacji I/O na MainThread
CPUProfiler - displays all
there are ANR trace file that you can get from the device
ANR - moze odbyc sie tez przez inne thready jak jest lockResource (Deadlock) - resource wykorzystywany przez inny watek, ktory jest potrzebny w glowny np. przez synchronized(resource) a ten tam resource jest przekazany do asynctaska

slow rendering jest nazywany jank

Main thread (Ui Thread) zadaniem jest wykonywanie blokow pracy z thread-safe kolejki do moementu zakonczenie pracy.
W tym zawiera sie: callbacki od lifecycle, wydarzenia od uzytkownia, od innych aplikacji i procesow, zarzadzanie viewsow (dlatego tez, nie daje sie zadnych referencji viewsow do innych watkow!- updated tylko poprzez wlascicicela Activity/Fragment), animacji

System probuje sie przerysowac co kazde 16 milisekund - jesli jest za duzo rzeczy do zrobienia na Main threadzie to nie zdazy - jank -> bedzie musial skipowac framey - co oznacze ze ich nie odsiweza.

AsyncTask(90% use caseow) nie moze byc inner(w kotlinie wystarczy usunaca inner) class Activity/Fragment bo bedzie trzymam referencje -> leak + jank
onPreExecution odbywa sie na MainThreadzie
doInBackground - juz na workerThreadzie
onPostExecution - MainThread
onCancel - MainThread

AasynTask ma 1 worker thread

HandlerThread - sklada sie z:
- loopera
kiedy praca do wykonania jest naprawde duza - zwracamy do UI Threada za pomoaca runOnUiThread

ThreadPoolExecutor - tworzy thready na ktorych moze byc wykonana pracy duzej ilosci malej, maks ilosc watkow mozna pobrac z sytemu.

IntentService - w momencie kiedy chcesz odpowiedziec intentowi, ale wisz ze praca troche zajmie a UI aktualnie jest w backgroundzie, Handler zly bo by czekal godzinamy na to az intent przyjdzie (a w miedzyczasie zjada resources)

wszystkei te klasy same ustawiaja thread priority, defaultowo thread priority bedzie takie samo jak maina - 0 (DEFAULT)
Foreground Threads - 95% execution time
background Threads - 5%


Jak aplikacja jest odpalana Android tworzy linux proces dla tej aplikacji z main threadem

### Modularization
androidx-dependency-tracker