# Aide mémoire : MAUI

## Sommaire

- [Fichier Ressources.](#one)
- [Les cycles de vie.](#two)
- [Gestion des événements.](#three)
- [Injection de dépendance.](#four)
- [Gestion de l'écran de démarrage.](#five)
- [Navigation.](#six)
- [InitializeComponent à quoi ça sert ?](#seven)
- [Les espaces de noms XAML.](#eight)
- [Créer une extension de balisage.](#nine)
- [Hiérarchie des balises + liste de celle les plus utiliser dans un fichier .xaml](#ten)
- [Exemple d’affichage dynamique de données](#eleven)
- [Utilisation du package 'CommunityToolkit.Mvvm'](#twelve)

---

<br>

## <a name="one">Fichier Ressources : </a>

Ce fichier se trouve à la racine du projet **App.xaml**.  
Tout ce qui est commun dans l'application ce trouve dans le fichier `Ressources`  
Et la référence à ceci se trouve dans `App.xaml` dans la balise `<ResourceDictionary>`  

```xml
<?xml version = "1.0" encoding = "UTF-8" ?>
<Application xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:MyMauiApp"
             x:Class="MyMauiApp.App">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <ResourceDictionary Source="Resources/Colors.xaml" /> <!-- Référence au fichier Ressources -->
                <ResourceDictionary Source="Resources/Styles.xaml" /> <!-- Référence au fichier Ressources  -->
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

---

<br>

## <a name="two">Les cycles de vies : </a>

La gestion des cycles de vie se fait côté 'behind' c'est à dire dans la classe .cs et n'ont pas directement dans le fichier .xaml

```c#
namespace MyMauiApp;

public partial class MainPage : Application
{
    public MainPage()
    {
        InitializeComponent();
    }

    protected override void OnStart()
    {
        base.OnStart();
    }

    protected override void OnResume()
    {
        base.OnResume();
    }

    protected override void OnSleep()
    {
        base.OnSleep();
    }
}
```

---

<br>

## <a name="three">Gestion des événements : </a>

Si on reprend l'exemple du compteur qui est dans l'app par défaut.

```xml
<Button
    x:Name="CounterBtn"
    SemanticProperties.Hint="Counts the number of times you click"
    Clicked="OnCounterClicked"
    HorizontalOptions="Fill" />
```

On donne un nom pour pouvoir le cibler côté .cs , celui-ci seras présent du côté .cs comme une variable donc elle doit avoir un nom unique.  
`x:Name="CounterBtn"`

Et côté code _behind_ donc fichier .cs il y a deux étape.

1. Accéder au bonne élément par son nom et lui assigner une valeur :  
   => `CounterBtn.Text = $"Clicked {count} time";`

2. Utilisation d'une méthode pour faire réagir le template côté .xaml d'une façon spécifique, ici on veux juste afficher le contenu de notre variable dans l'élément cibler et mettre en place l'accessibiliter dans le cas ou la personne utilise un lecteur d'écran :  
   => `SemanticScreenReader.Announce(CounterBtn.Text);`

IMPORTANT, règle de base pour une méthode de gestion d'évent :

- Doit toujours retourner **void**.
- Deux params obligatoire :
  - un **object** indiquant l’objet qui a déclenché l’événement (appelé expéditeur, _sender_)
  - un **EventArgs** contenant tous les arguments passés au gestionnaire d’événements par l’expéditeur.
- La méthode de gestion de l’évent doit être **private**.
- La méthode de gestion peut être **async** s’il a besoin d’exécuter des opérations asynchrones.
- Le nom de la méthode suit une convention standard, **On** suivi du nom du contrôle **Counter**, et le nom de l’événement **Clicked**.

```c#
private void OnCounterClicked(object sender, EventArgs e)
{
    //..logique
}
```

---

<br>

## <a name="four">Comprendre la Class MauiProgram spécifique pour chaque plateforme (android, IOS, ect) + l'injection de dépendance. </a>

Chaque plateforme native a un point de départ différent, qui crée et initialise l’application.  
Vous pouvez trouver le code ci-dessous à la racine du projet il se nomme **MauiProogram.cs**.  
On remarque que l'on lui donne le type de la classe de démarage de notre programme : `UserMauiApp<App>()` **App**, ensuite ont peut chainer des méthodes de configuration, par exemple dans le code ci dessous, une injection de dépendance pour des fichiers de type font.

```c#
namespace MyMauiApp;

public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();
        builder
            .UseMauiApp<App>()
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            });

        return builder.Build();
    }
}
```

---

<br>

## <a name="five">Comprendre et personaliser l'écran de démarage de l'application </a>

Pour accéder à ce fichier fait un clique ou un double clique sur le nom de ta solution.  
La section **ItemGroup** située sous le groupe de propriétés initial vous permet de spécifier une image et une couleur pour l’écran de démarrage qui s’affiche durant le chargement de l’application, avant l’apparition de la première fenêtre. Vous pouvez également définir les emplacements par défaut des polices, des images et des ressources utilisées par l’application.

```xml
<Project Sdk="Microsoft.NET.Sdk">
   <ItemGroup>
        <!-- App Icon -->
        <MauiIcon Include="Resources\appicon.svg"
                  ForegroundFile="Resources\appiconfg.svg"
                  Color="#512BD4" />

        <!-- Splash Screen -->
        <MauiSplashScreen Include="Resources\appiconfg.svg"
                          Color="#512BD4"
                          BaseSize="128,128" />

        <!-- Images -->
        <MauiImage Include="Resources\Images\*" />
        <MauiImage Update="Resources\Images\dotnet_bot.svg"
                   BaseSize="168,208" />

        <!-- Custom Fonts -->
        <MauiFont Include="Resources\Fonts\*" />

        <!-- Raw Assets (also remove the "Resources\Raw" prefix) -->
        <MauiAsset Include="Resources\Raw\**"
                   LogicalName="%(RecursiveDir)%(Filename)%(Extension)" />
    </ItemGroup>
</Project>
```

---

<br>

## <a name="six">Gestion de la navigation : </a>

Les fichiers AppShell.xaml et AppShell.xaml.cs qui spécifient la page initiale de l’application et gèrent l’inscription des pages pour le routage de la navigation et la façon dont la navigation s'affiche un exemple pour la nav à gauche et le second pour une nav plus classique en haut de l'app.

```xml
<Shell
    x:Class="MauiApp1.AppShell"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:MauiApp1"
    Shell.FlyoutBehavior="Locked"
    Title="MauiApp1">

    <FlyoutItem Title="Home">
        <ShellContent ContentTemplate="{DataTemplate local:MainPage}" />
    </FlyoutItem>
    <FlyoutItem Title="New Page">
        <ShellContent ContentTemplate="{DataTemplate local:NewPage1}" />
    </FlyoutItem>

</Shell>
```

**Shell.FlyoutBehavior="Locked"**  
ou  
**Shell.FlyoutBehavior="Flyout"**  

Affiche une navigation sous forme de side barre à gauche.

```xml
<Shell
    x:Class="MauiApp1.AppShell"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:MauiApp1"
    Title="MauiApp1">

    <TabBar>
        <Tab Title="Home">
            <ShellContent ContentTemplate="{DataTemplate local:MainPage}" />
        </Tab>
        <Tab Title="Settings">
            <ShellContent ContentTemplate="{DataTemplate local:NewPage1}" />
        </Tab>
    </TabBar>
```

</Shell>

`<TabBar>` + `<Tab>` permet d'avoir une nav barre hoizontale au top de l'app

Supprimer la balise _Shell.FlyoutBehavior=""_ si on utilise la balise `<TabBar>`  

### Bonus : Personnaliser l'apparence des onglets

```xml
<Tab Title="Home" Icon="home_icon.png">
    <ShellContent ContentTemplate="{DataTemplate local:MainPage}" />
</Tab>
```
  
`Icon` ajoute une icone pour personalisé la nav.

---

<br>

## <a name="seven">InitializeComponent : </a>

```c#
namespace MauiXaml;

public partial class Page1 : ContentPage
{
    public Page1()
    {
        InitializeComponent();
    }
}
```

La méthode **InitializeComponent()** dans le constructeur de _page1_ lit la description XAML de la page, charge les divers contrôles sur cette page et définit leurs propriétés. Ce qui nous permet après d'implémenter des logiques en ciblant les éléments souhaiter.   

*Résumer ça bind la vue (MonFichier.xaml) avec la logique C# (MonFichier.xaml.cs)*  

**Important le code logique dois se trouver après cette méthode.**

---

<br>

## <a name="eight">Comprendre les espace de nom dans les fichiers XAML </a>  

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:mycode="clr-namespace:MyMauiApp"
             x:Class="MyMauiApp.Page1" >
</ContentPage>
```
- `xmlns="http://schemas.microsoft.com/dotnet/2021/maui"` :
  -  Définit l'espace de noms principal pour les contrôles MAUI comme Button, Label, etc.

- `xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"` :
  - Définit l'espace de noms pour les fonctionnalités spécifiques à XAML, comme **x:Class** qui relie le fichier XAML à la classe code-behind ou **x:Name** attribue un nom unique à un élément dans le code XAML, ce qui permet de faire référence à cet élément directement depuis le code C#.

- `xmlns:mycode="clr-namespace:MyMauiApp"` :
  - Définit un préfixe mycode pour référencer les classes dans le namespace MyMauiApp de votre code C#. Cela permet d'utiliser des éléments définis dans ce namespace directement dans votre fichier XAML. Le clr-namespace indique que le namespace est un namespace Common Language Runtime (CLR) de .NET.

- `x:Class="MyMauiApp.Page1"` :
  - Spécifie que le code-behind pour cette page XAML est la classe Page1 dans le namespace MyMauiApp.

<br>

## <a name="nine">Créer une extension de balisage </a>

Tout dabord avant de commencer de créer sa propre classe pour gérer des propriétés de façon global il faut savoir que MAUI nous met à la disposition un pannel pré-fabriquer pour nous aider à construire les vues .xaml facilement, on peut consulter le fichier dans le folder **Ressources** -> **Styles** -> **Styles.xaml**.

Dans le template de base générer par Visual Studio, on peut voire l'utilisation de **Styles.xaml** dans le fichier **MainPage.xaml**.  

_L'utilisation est un peu spécial car il s'agit d'un fichier static commun pour toute l'application_

```xml
    <Label
        Text="Hello, World!"
        Style="{StaticResource Headline}"
        SemanticProperties.HeadingLevel="Level1" />
```

Création de sa propre classe d'extension.

```c#
namespace MyMauiApp;

public partial class MainPage : ContentPage
{
    public const double MyFontSize = 28;

    public MainPage()
    {
        InitializeComponent();
    }
}

// Méthode d'extension
public class GlobalFontSizeExtension : IMarkupExtension
{
    public object ProvideValue(IServiceProvider serviceProvider)
    {
        return MainPage.MyFontSize;
    }
}
```

Utilisation dans le code .xaml (ne pas oublier de déclarer le namespace en premier)
```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:mycode="clr-namespace:MyMauiApp"
             x:Class="MyMauiApp.MainPage">

    <Label Text="Hello, World!"
        FontSize="{mycode:GlobalFontSize}" />
</ContentPage>
```

Avantage de cette méthode ?

- Permet de faire une généricité au niveau de la propriété _FontSize_ ce qui permet d'adapter une FontSize plus grande ou plus petite pour un certain cas de figure de façon très facile pour l'ensemble des éléments qui comporte la variable **MyFontSize**car il n'y auras qu'une valeur à changer, celle-ci est la valeur de la constante déclarer au début de notre classe `public const double MyFontSize = 28;`

---

<br>

## <a name="ten">Hiérarchie des balises + liste de celle les plus utiliser dans un fichier .xaml : </a>

Balises de base

- `<ContentPage>` : Conteneur principal pour une page dans une application MAUI.
- `<ScrollView>` : Permet de faire défiler son contenu.
- `<VerticalStackLayout>` : Organise les vues enfants dans une pile verticale.
- `<Grid>` : Conteneur de mise en page flexible qui permet de disposer les éléments en lignes et colonnes.
- `<StackPanel>` : Conteneur de mise en page qui empile les éléments enfants horizontalement ou verticalement.
- `<Canvas>` : Conteneur de mise en page qui permet de positionner les éléments enfants en utilisant des coordonnées absolues.
  Balises de contrôle
- `<Button>` : Représente un bouton cliquable.
- `<TextBox>` : Représente une zone de texte éditable.
- `<Label>` : Représente un texte non éditable.
- `<Image>` : Affiche une image.
- `<ListView>` : Affiche une liste d’éléments.
- `<ComboBox>` : Représente une liste déroulante.
  Balises pour l’affichage dynamique de données
- `<CollectionView>` : Affiche une collection de données sous forme de liste ou de grille.
- `<DataTemplate>` : Définit la structure visuelle pour chaque élément de données.
- `<Binding>` : Lie une propriété d’un contrôle à une source de données.

Hiérarchie example

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="">
    <ScrollView>
        <VerticalStackLayout>
            <Image Source="" />
            <Label Text="" />
            <Button Text="" />
            <ListView ItemsSource="">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <TextCell Text="" Detail="" />
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </VerticalStackLayout>
    </ScrollView>
</ContentPage>
```

## <a name="eleven">Exemple d’affichage dynamique de données : </a>

1. Fichier C#

```c#
public class MainViewModel : INotifyPropertyChanged
{
    private ObservableCollection<Item> items;
    public ObservableCollection<Item> Items
    {
        get => items;
        set
        {
            items = value;
            OnPropertyChanged();
        }
    }

    public MainViewModel()
    {
        LoadData();
    }

    private async void LoadData()
    {
        // Appel à l'API pour récupérer les données
        var data = await ApiService.GetDataAsync();
        Items = new ObservableCollection<Item>(data);
    }

    public event PropertyChangedEventHandler PropertyChanged;
    protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

2. Fichier XAML

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MyMauiApp.MainPage"
             BindingContext="{StaticResource MainViewModel}">
    <ScrollView>
        <VerticalStackLayout>
            <ListView ItemsSource="{Binding Items}">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <TextCell Text="{Binding Name}" Detail="{Binding Description}" />
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </VerticalStackLayout>
    </ScrollView>
</ContentPage>
```

Explication

- **ViewModel** : Contient la logique pour charger les données depuis une API et les exposer via une collection observable.
- **BindingContext** : Définit le contexte de liaison pour la page.
- **ListView** : Affiche les éléments de la collection observable avec un modèle de données défini par **<DataTemplate>**

---

<br>

## <a name=twelve>Package CommunityToolkit.Mvvm </a>
Voici le lien pour installer le package : https://www.nuget.org/packages/CommunityToolkit.Mvvm  
Voici la doc officiel par Microsoft : https://learn.microsoft.com/fr-fr/dotnet/communitytoolkit/mvvm/

### Petite démo de la puissance du package : 

L'utilisation de ce package simplifie le code il suffit d'utiliser par exemple l'attribut : 
```c#
[ObservableProperty]
private string _titlePage = "Accueil";
```

Pour ne pas à avoir les étapes habituelle pour un binding de propriété d'un ViewModel à une View 

```c#
private string _title = "Accueil";
public string Title
{
    get => _title;
    set
    {
        if (_title != value)
        {
            _title = value;
            OnPropertyChanged();
        }
    }
}
```

Un autre exemple pour le biding d'event avec le package :
```c#
public partial class MonViewModel : ObservableObject
{
    [RelayCommand]
    private async Task NavigateToDetails()
    {
        // Ici on décide de rediriger le user au clique
        await Shell.Current.GoToAsync("//details");
    }
}
```

Sans le package :
```c#
public class MonViewModel : INotifyPropertyChanged
{
    private ICommand _navigateToDetailsCommand;
    public ICommand NavigateToDetailsCommand => _navigateToDetailsCommand ??= new Command(async () => await NavigateToDetails());

    private async Task NavigateToDetails()
    {
        // Ici on décide de rediriger le user au clique
        await Shell.Current.GoToAsync("//details");
    }

    public event PropertyChangedEventHandler PropertyChanged;
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```


---

<br>
