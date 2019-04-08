# Spis treści
* [Klasy](#klasy)
* [Metody pola i właściwości](#method)
* [Konwencje](#konwencje)
	* [Nazewnictwo](#nazewnictwo) 	
	* [Przestrzenie nazw](#namespace)
	* [Interfejsy](#interfejsy)	
	* [Warunki i pętle](#loops)
* [Globalizacja](#lang) 
* [Modularyzacja](#modularyzacja)
	* [Podział projektów](#project)
	* [Podział folderów](#folder)
	* [RCL](#rcl)
* [Feature Toggle](#feature-toggle)
* [Wersje oprogramowania i konfiguracja](#version)
* [Repozytorium](#repozytorium)
		

# Klasy
Klasy deklarujemy zawsze w pliku o nazwie klasy. Nazwa klasy powinna wskazywać zastosowanie lub sposób użycia klasy. 
Nie stosujemy przedrostków dla klas abstrakcyjnych lub wirtualnych

```
        //Niskopoziomowy Model użytkownika lub Encja zmapowana z bazy danych
        public class User{}  
        
        //Model użytkownika do wykorzystania w warstwie GUI
        public class UserModel {} 
        
        //Serwis do obsługi danych związanych z użytkownikiem dziedziczący po Interfejsie IUserRepository
        public class UserService : IUserRepository {} 
```


# Metody, Pola, Właściwości <a name="method"></a>
Nazwy metod zawsze piszemy z wielkiej litery z wykorzystaniem camelCase. Powinny one mówić o konkretnym użyciu. Parametry przekazane do metod powinny zawsze rozpoczynać się małą literą.
```
    public class String {  
        public int CompareTo(object value);  
        public string[] Split(char[] separator);  
        public string Trim();  
    }  
```
Właściwości klasy tworzymy jako zmienne publiczne i rozpoczynamy je z wielkiej litery, 

```
    public class User
    {       
        public long UserId { get; set; }
        public string Login { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }

        public User(long userId, string login, string firstName, string lastName)
        {
            UserId = userId;
            Login = login;
            FirstName = firstName;
            LastName = lastName;
        }
    }
```

Poniższy listing przedstawia przykładową klasę w której widać że:
- zmienne prywatne stałe ```const``` nazywamy z wielkiej litery
- zmienne prywatne zdefiniowane globalnie w obrębie klasy rozpoczynamy poprzedzając znakiem udenrscore '_'
- zmienne prywatne zdefiniowane w ramach metody rozpoczynamy z małej litery. Kiedy to możliwe kożystamy z typu 'var' zamiast typu nazwanego 
- zmienne przekazywane do konstruktora tak samo jak parametry metod rozpoczynamy z małej litery
- staramy się łączyć zmienne w bloki o tych samych modyfikatorach dostępu oraz typach

```
    public class LoginController : BaseController
    {
        private const string ClassName = nameof(LoginController);
        private const string ConsentName = ".AspNet.Consent";
        private const string DefaultUserWebRole = "User";

        private readonly SessionSettings _sessionSettings;
        private readonly IUserRepository _userRepository;
        
        private string _login;

        public LoginController(AggregateService aggregateService, IUserRepository userRepository, IOptions<SessionSettings> sessionSettings) : base(aggregateService)
        {
            _session = new Session(sessionTimeout, storage);
            _userRepository = userRepository;
        }

        [AllowAnonymous]
        public IActionResult Index()
        {
            _login = GetCurrentUserLogin();
            var portalId = GetPortal().PortalId;
        }
    }
```

# Konwencje 
## Nazewnictwo<a name="naming" />
Nie używać skrótów przy nazwyaniu:

- klas
```
        //źle
        public class Lang { }

        //dobrze
        public Language { }
```
- metod
```
        //źle
        public string GetCultFromLang(Lang lang){ }

        //dobrze
        public string GetCultureFromLanguage(Language language){ }
```
- zmiennych
```
        //źle
        private DbContext ctx;
        private string langName;
        
        //dobrze
        private DbContext context;
        private string languageName;       
```

## Przestrzenie nazw<a name="namespace" /> 
Przestrzeń nazw (namespace) powinna zawsze odpowiadać strukturze katalogów w której znajduje się klasa. Nazewnictwo namespace`ów powinno odpowiadać poniższemu szablonowi ```    <Company>.(<Product>|<Technology>)[.<Feature>][.<Subnamespace>] ``` 
na przykład

```        
        Microsoft.Office.PowerPoint
        DotKit.Core.Security
```
Dodatkowo:
- nie należy używać tej samej nazwy dla namespace oraz klasy znajdującej się wewnątrz tego namespace.
- można używać liczby mnogiej dla namespace tam gdzie jest to wskazane np. ```System.Collections```
- w ramach jednego namespace nie można dwukrotnie utworzyć typu o tej samej nazwie



    
## Interfejsy
Interfejs jest jedynym obiektem dla którego stosujemy przedrostek. Każdy interfejs powinien być poprzedzony literą 'I'. 

## Warunki i pętle <a name="loops" />
Warunki lub pętle posiadające tylko jedną linię kodu nie powinny być zawarte w \{ \}
```
        if(condition == true)
            DoSomething();
        else
            DoSomethingElse();
            
        foreach(var item in itemsCollection)
            result.Add(item.Id);
```


# Globalizacja <a name="lang" />
Wszystkie aplikacje i całość kodu powinny być pisane w języku angielskim. Jeżeli potrzebujemy wyświetlić informację w języku polskim np. nazwy w menu bądź treść strony powinniśmy skorzystać z plików resx w którym będą trzymane tłumaczenia w zależności od wybranego w aplikacji języka (culture). Klucze w resx po których będziemy się odwoływać do tłumaczenia powinny w łatwy sposób określać położenie i wykorzystanie tłumaczenia:


| klucz           | angielski               | polski         |
| ------------- |----------------|-----------|
| Menu.Logout   | Logout           | Wyloguj    |
| Home.Index.Welcome | Welcom to test Application | Witaj w aplikacji testowej |

Przykładowe wykorzystanie klucza

```
<a class="btn" href="/Login/Logout">@SharedLocalizer["Menu.Logout"]</a>
<h4>@SharedLocalizer["Home.Index.Welcome"]</h4>
```

Więcej informacji na temat konfiguracji i wykorzystania SharedLocalizer można znaleźć [Tutaj](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.2).




-------------------
# Modularyzacja

## Podział projektów <a name="project"></a>
Należy dzilić projekty w ramach solucji na niezależne ze względu na funkcjonalności np.

```DotKit.Core``` - projekt zawierający encje i serwisy, niska warstwa kodu mająca połączenie z bazą danych i serwisami zewnętrznymi

```DotKit.Smtp``` - projekt odpowiedzialny za wysyłkę e-maili 

```DotKit.Modules.*``` - projekty RCL. Niezależne moduły reużywalne w ramach projektów webowych (więcej na temat RCL w sekcji RCL)
- ```DotKit.Modules.Login``` - moduł logowania
- ```DotKit.Modules.Event``` - moduł zdarzeń i logów
- ```DotKit.Modules.Security``` - moduł uprawnień i dostępów



## Podział folderów <a name="folder"></a>
Podczas wytwarzania kodu należy zwrócić również uwagę na modularyzację folderów np. Interfejsy, Serwisy, Encje, Kontrolery, Widoki, Zasoby (Resource) itp powinny znajdować się czytelnie zdefiniowanych strukturach i osobnych folderach - co jednocześnie wymusza prawidłowy podział namespaców. Przykładowy podział katalogowy modułu (projektu):

```
  DotKit.Modules.Login
    |-- Controllers
    |-- Modules
    |-- Resources
    |-- Settings
    |-- Views
```


## RCL
Widoki razorowe, strony, kontrolery, modele oraz komponenty moga być kompilowane do RCL (Razor Class Library) i reużywalne w ramach innych projektów.
Aplikacja z referencją do biblioteki razorowej może wykorzystać lub nadpisać zawarte w niej widoki i strony.

Pisząc RCL wytwarzamy projekt musimy zwrócić uwagę na jego konfigurację. Aby powstała nam biblioteka RCL .csproj musi wyglądać jak poniżej:
```
<Project Sdk="Microsoft.NET.Sdk.Razor">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="2.1.2" />
  </ItemGroup>

</Project>
```

Projekty RCL możemy referencjować przy użyciu standardowej referencji projektowej lub Nuget. Więcej informacji na temat RCL można znaleźć [Tutaj](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/ui-class?view=aspnetcore-2.2&tabs=visual-studio).


-------------------

# Feature Toggle

Mechanizm który implementujemy aby móc bez ingerencji w kod włączać/wyłączać części funkcjonalności aplikacji za pomocą prostej modyfikacji pliku appsettings. Aby zaimplementować feature toggle należy:

 - Dodać referencję do biblioteki DotKit.ToogleFeatures.
 - Wewnątrz solucji należy stworzyć klasę dziedziczącą SimpleFeatureToggle np. TestFeature
 - Zmodyfikować lub utworzyć jeżeli nie istnieją 3 klasy:
    - W klasie FeatureToggleWrapper dodać statyczną zmienną nowo utworzonego typu TestFeature
    - W klasie FeatureToggleType Dodać enumerator dla TestFeature
    - W FeatureToggleFilterAttribute w metodzie OnActionExecuting dołożyć case na TestFeature i ustawić mu wartość akcji w przypadku wyłączenia toggla
```   
    switch (_featureToggleType)
    {
        case FeatureToggleType.TestFeature:
            if (!FeatureToggleWrapper.TestFeature.FeatureEnabled)
                context.Result = new RedirectToActionResult(ActionName, ControllerName, null);
        break;
    }
```

 - Dla projektu do którego dodaliśmy referencję należy otworzyć plik startup i zainicjować FeatureToggleWrapper oraz dodać go do services jako singleton. Uwaga. Należy uważać na namespace gdyż może się okazać że FeatureToggleWrapper jest zaimplementowany w kilku projektach. Dodając inicjalizację używajmy pełnych nazw np.
```
    FeatureToggleWrapper.ApplicationViewFeature = new ApplicationViewFeature {ToggleValueProvider = appSettingsProvider};
    DotKit.ToogleFeatures.Filters.FeatureToggleWrapper.LoginFormFeature = new LoginFormFeature {ToggleValueProvider = appSettingsProvider};
```

 - W pliku appsettings dodać do sekcji FeatureToggle zmienną  
```
    "FeatureToggle": {
        TestFeature: "true",
        …
    }
```

Od tej pory możemy wykorzystać nasz FeatureToggle:
1. Do weryfikacji wykonania akcji po stronie kontrolera
```
        [FeatureToggleFilter(FeatureToggleType.TestFeature)]
        public IActionResult Confirm(string id){}
```
2. Po stronie kontrolera przy użyciu DependencyInjection
```
        public HomeController(TestFeature testFeature)
        {
            if (testFeature.FeatureEnabled){...}
        }
```
3. Wyłączenia pojedyńczej funkcjonalności w widoku
```
    @using Manager2.Web.Filters

    @if (FeatureToggleWrapper.TestFeature.FeatureEnabled)
    {
        <a class="dropdown-item" href="#>@SharedLocalizer["test"]</a>
    }
```

-------------------

# Wersje oprogramowania i konfiguracja <a name="version"></a>

Wszystkie wersje oprogramowania powinny być wytwarzane w oparciu o framework .NET Core posiadających suport LTS (Long Term Support). Jeżeli do realizacji projektu jest wymagana wersja inna nie posiadająca LTS możliwe jest użycie innej po wcześniejszym uzasadnieniu i ustaleniu tego z zespołem OPL.

Wytworzone przy użyciu frameworka aplikacje muszą być publikowane jako aplikacje self-contained. Publikacja taka sprawia, że aplikacji możemy użyć na dowolnym serwerze i zawiera w sobie wszystkie niezbędne do działania biblioteki.

Serwery będą przygotowywane i utrzymywane przez zespół utrzymaniowy po stronie OPL na podstawie instrukcji przygotowywanych przez dostawcę.

# Repozytorium
Podstawowym repozytorium za pomocą którego OPL ma wgląd w kod źródłowy jest GIT. Na potrzeby wewnętrznego wytwarzania kodu można korzystać z dowolnego systemu kontroli wersji.
