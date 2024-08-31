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
- [Explication des balises de binding dans le xaml](#thirteen)
- [Utilisation d'un Modèle](#fourteen)
- [Tuto MAUI, créer une App facilement c'est ici !](#fifteen)

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

_Résumer ça bind la vue (MonFichier.xaml) avec la logique C# (MonFichier.xaml.cs)_

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

  - Définit l'espace de noms principal pour les contrôles MAUI comme Button, Label, etc.

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

Utilisation dans la view :

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MAUIAppTest.MaPageView">

<ContentPage.BindingContext>
    <vm:MaPageViewModel />
</ContentPage.BindingContext>

    <Label
        Text="{Binding Title}"
    />

</ContentPage>
```

**Remarque** : dans le template xaml on utilise la propriétés public '**T**itle', il faut garder à l'esprit que `[ObservableProperty]` fait exactement ce qui est expliquer dans la partie 'deux' sans le package `CommunityToolkit.Mvvm`.

Un autre exemple pour les events avec le package :

```c#
public partial class MonViewModel : ObservableObject
{
    [ObservableProperty]
    private string _titlePage = "Accueil";

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

Utilisation dans la View :

```xml
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="MAUIAppTest.MaPageView">

<ContentPage.BindingContext>
    <vm:MaPageViewModel />
</ContentPage.BindingContext>

    <Button
        Text="{Binding Title}"
        Command="{Binding NavigateToDetailsCommand"}
    />

</ContentPage>
```

**Remarque** : ici pour utiliser notre méthode ont utilise son nom + **Command**

---

<br>

##<a name="thirteen">Explication des balises de binding dans le xaml</a>

1. BindingContext

```xml
<ContentPage.BindingContext>
    <vm:ConnectionPageViewModel />
</ContentPage.BindingContext>
```

- But : Cette section établit le contexte de liaison (BindingContext) pour la page. Elle permet de lier les propriétés de l'interface utilisateur aux propriétés et commandes du ViewModel.
- Utilisation : Ici, la page utilise le ConnectionPageViewModel comme contexte de données. Cela permet aux contrôles sur la page d'accéder aux propriétés et commandes définies dans ce ViewModel.

2. Resources

```xml
<ContentPage.Resources>
    <ResourceDictionary>
        <Color x:Key="PrimaryColor">#007ACC</Color>
        <Style x:Key="PrimaryButtonStyle" TargetType="Button">
            <Setter Property="BackgroundColor" Value="{StaticResource PrimaryColor}" />
        </Style>
        <converters:InverseBoolConverter x:Key="InverseBoolConverter" />
    </ResourceDictionary>
</ContentPage.Resources>
```

- But : La section Resources contient des ressources partagées, comme des couleurs, des styles ou des convertisseurs, qui peuvent être réutilisés dans la page.
- Utilisation :
  - Color (PrimaryColor) : Définit une couleur réutilisable pour d'autres éléments de l'interface.
  - Style (PrimaryButtonStyle) : Crée un style réutilisable pour les boutons, appliquant la couleur définie par PrimaryColor.
  - Convertisseur (InverseBoolConverter) : Déclare un convertisseur de valeur personnalisé qui peut être utilisé dans les liaisons pour inverser une valeur booléenne.

### Résumé

`<ContentPage.BindingContext>` : Associe la page à un ViewModel pour les liaisons de données.  
`<ContentPage.Resources>` : Déclare des ressources réutilisables (couleurs, styles, convertisseurs) dans un dictionnaire de ressources.

---

<br>

##<a name="fourteen">Utilisation d'un Modèle</a>

1. Utilisation d'un modèle pour récupérer des données :

```cs
// Création d'un objet LogUser
namespace MAUIAppTest.Models
{
    public record LogUser(string Email, string Password);
}
```

```cs
// Mise en place de la logique de récupération des données
// + Vérif des datas
// + Construction de l'objet 'Model' LogUser
// + Envoye des datas
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

using MAUIAppTest.Models;

namespace MAUIAppTest.ViewModels
{
    public partial class ConnectionPageViewModel : ObservableObject
    {
        [ObservableProperty]
        private string _email = string.Empty;

        [ObservableProperty]
        private string _password = string.Empty;


        [RelayCommand]
        private async Task SubmitDataFromConnectionForm()
        {
            if (string.IsNullOrEmpty(_email))
                await Application.Current.MainPage.DisplayAlert("Erreur", "le champ email doit être compléter", "OK");

            if (string.IsNullOrEmpty(_password))
                await Application.Current.MainPage.DisplayAlert("Erreur", "le champ password doit être compléter", "OK");

            LogUser logUser = new(Email, Password);
        }
    }
}
```

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:vm="clr-namespace:MAUIAppTest.ViewModels"
             x:Class="MAUIAppTest.Views.ConnectionPageView"
             Title="Identifie toi">

    <ContentPage.BindingContext>
        <vm:ConnectionPageViewModel />
    </ContentPage.BindingContext>

    <VerticalStackLayout Spacing="30" Padding="10">

        <Label Text="Entre ton mail :" />

        <Entry
            Text="{Binding Email}"
            Placeholder="example@mail.be"
            ClearButtonVisibility="WhileEditing"
            Keyboard="Email"
            HorizontalOptions="FillAndExpand" />

        <Label Text="Entre ton pass :" />

        <Entry
            Text="{Binding Password}"
            IsPassword="{Binding IsPasswordVisible}"
            Placeholder="Enter your password"
            HorizontalOptions="FillAndExpand"
            VerticalOptions="Center" />

        <Button
            Text="C'est parti !"
            Command="{Binding SubmitDataFromConnectionFormCommand}" />

    </VerticalStackLayout>

</ContentPage>

```

##<a name="fifteen"> Début du tuto '_création d'une app MAUI_' (avec le package CommunityToolkit.Mvvm)</a>

1. Construire notre architecture MVVM

0_projet_base.png

### Avant de commencer à coder nous allons créer 3 folders :

    - Model
    - ViewModel
    - View

![one](/Screen/0_projet_base.png)

1.1 Maintenant que nous avons notre architecture en place, déplaçons les fichiers déjà présent dans l'app de base dans leur folder :

- `MainPage.xaml` avec son code behind `MainPage.xaml.cs` dans le folder `View`.
  - Vérifier si les `namespace` sont toujours correct normalement votre IDE vous propose de les changer automatiquement.

![two](/Screen/1_ajout_folders_MVVM.png)

Voici à quoi doit ressembler votre fichier :

```cs
namespace Tuto_MAUI.View   <----------------------------
{
    public partial class MainPage : ContentPage
    {
        int count = 0;

        public MainPage()
        {
            InitializeComponent();
        }

        private void OnCounterClicked(object sender, EventArgs e)
        {
            count++;

            if (count == 1)
                CounterBtn.Text = $"Clicked {count} time";
            else
                CounterBtn.Text = $"Clicked {count} times";

            SemanticScreenReader.Announce(CounterBtn.Text);
        }
    }
}
```

**Ne pas oublier de mettre à jour le code XAML**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Tuto_MAUI.View.MainPage">  <----------------------------

    <ScrollView>
        <VerticalStackLayout
            Padding="30,0"
            Spacing="25">
            <Image
                Source="dotnet_bot.png"
                HeightRequest="185"
                Aspect="AspectFit"
                SemanticProperties.Description="dot net bot in a race car number eight" />

            <Label
                Text="Hello, World!"
                Style="{StaticResource Headline}"
                SemanticProperties.HeadingLevel="Level1" />

            <Label
                Text="Welcome to &#10;.NET Multi-platform App UI"
                Style="{StaticResource SubHeadline}"
                SemanticProperties.HeadingLevel="Level2"
                SemanticProperties.Description="Welcome to dot net Multi platform App U I" />

            <Button
                x:Name="CounterBtn"
                Text="Click me"
                SemanticProperties.Hint="Counts the number of times you click"
                Clicked="OnCounterClicked"
                HorizontalOptions="Fill" />
        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

**Idem pour le code déjà présent dans le fichier `AppShell.xaml`**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Shell
    x:Class="Tuto_MAUI.AppShell"
    xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:local="clr-namespace:Tuto_MAUI.View"  <----------------------------
    Shell.FlyoutBehavior="Disabled"
    Title="Tuto_MAUI">

    <ShellContent
        Title="Home"
        ContentTemplate="{DataTemplate local:MainPage}"
        Route="MainPage" />

</Shell>
```

Lancer l'app pour vérifier que celle fonctionne toujours correctement.

1.2 Crétion d'une simple classe _cs_ => `MainPageViewModel.cs`.

![three](/Screen/3_deplacer_dans_viewmodel.png)

1.3 Déplacer la logique du code du fichier behind de la View dans notre nouvelle classe :

```cs
namespace Tuto_MAUI.ViewModel
{
    internal class MainPageViewModel
    {
        int count = 0;
        private void OnCounterClicked(object sender, EventArgs e)
        {
            count++;

            if (count == 1)
                CounterBtn.Text = $"Clicked {count} time";
            else
                CounterBtn.Text = $"Clicked {count} times";

            SemanticScreenReader.Announce(CounterBtn.Text);
        }
    }
}
```

Le fichier du code behind devrais ressembler à ceci :

```cs
namespace Tuto_MAUI.View
{
    public partial class MainPage : ContentPage
    {
        public MainPage()
        {
            InitializeComponent();
        }
    }
}
```

A ce stade vous constatez que des erreurs sont apparu dans notre classe **MainPageViewModel** au niveau des lignes de code suivante :

- CounterBtn.Text = $"Clicked {count} time";
- CounterBtn.Text = $"Clicked {count} times";
- SemanticScreenReader.Announce(CounterBtn.Text);
  C'est tout à fait normal puisque avant ont récupèrais le **Button** dans le code behind et les deux fichiers était lié via cette ligne dans le xaml :
- x:Class="Tuto_MAUI.View.MainPage"
  Et l'on pouvais changer le texte du **Button** via un évenement sur le **Button**
  Voici comment résoudre se problème :
- En premier retournons du côté **View** dans notre fichier **MainPage.xaml** et ajoutons ceci :

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Tuto_MAUI.View.MainPage"
             xmlns:vm="clr-namespace:Tuto_MAUI.ViewModel" <------------------- on fait connaitre au template xaml le folder ViewModel
             >

    <ContentPage.BindingContext>  <------------------- on lui bind un contexte
        <vm:MainPageViewModel/>   <------------------- on lui précise le fichier du contexte
    </ContentPage.BindingContext> <-------------------

    <ScrollView>
        <VerticalStackLayout
            Padding="30,0"
            Spacing="25">
            <Image
                Source="dotnet_bot.png"
                HeightRequest="185"
                Aspect="AspectFit"
                SemanticProperties.Description="dot net bot in a race car number eight" />

            <Label
                Text="Hello, World!"
                Style="{StaticResource Headline}"
                SemanticProperties.HeadingLevel="Level1" />

            <Label
                Text="Welcome to &#10;.NET Multi-platform App UI"
                Style="{StaticResource SubHeadline}"
                SemanticProperties.HeadingLevel="Level2"
                SemanticProperties.Description="Welcome to dot net Multi platform App U I" />

            <Button
                x:Name="CounterBtn"
                Text="Click me"
                SemanticProperties.Hint="Counts the number of times you click"
                Clicked="OnCounterClicked"
                HorizontalOptions="Fill" />
        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

Cool mais ça n'a pas résolu notre problème de base, le fichier **MainPageViewModel** indique toujours les même erreurs :(
Modfions donc notre fichier **MainPageViewModel** comme ceci :

```cs
using CommunityToolkit.Mvvm.ComponentModel; // using du package
using CommunityToolkit.Mvvm.Input;

namespace Tuto_MAUI.ViewModel
{
    internal partial class MainPageViewModel : ObservableObject // Ajout de mot clé 'partial' + ':ObservableObject'
    {
        [ObservableProperty] // Ajout d'un attribut pour obtenir une variable Observable dans notre View
        private int _count = 0;

        [ObservableProperty]
        private string _buttonText = "Click me";

        [RelayCommand]  // Ajout d'un attribut pour obtenir un evenement utilisable dans notre View
        private void OnCounterClicked()
        {
            Count++;

            if (Count == 1)
                ButtonText = $"Clicked {Count} time";
            else
                ButtonText = $"Clicked {Count} times";

            SemanticScreenReader.Announce(ButtonText);
        }
    }
}
```

Bonne nouvelle plus d'erreurs dans notre ViewModel =D
Maitenant retournon dans notre View pour lui ajouter notre varibale **ButtonText** et son évènement **OnCounterClicked**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Tuto_MAUI.View.MainPage"
             xmlns:vm="clr-namespace:Tuto_MAUI.ViewModel"
             >

    <ContentPage.BindingContext>
        <vm:MainPageViewModel/>
    </ContentPage.BindingContext>

    <ScrollView>
        <VerticalStackLayout
            Padding="30,0"
            Spacing="25">
            <Image
                Source="dotnet_bot.png"
                HeightRequest="185"
                Aspect="AspectFit"
                SemanticProperties.Description="dot net bot in a race car number eight" />

            <Label
                Text="Hello, World!"
                Style="{StaticResource Headline}"
                SemanticProperties.HeadingLevel="Level1" />

            <Label
                Text="Welcome to &#10;.NET Multi-platform App UI"
                Style="{StaticResource SubHeadline}"
                SemanticProperties.HeadingLevel="Level2"
                SemanticProperties.Description="Welcome to dot net Multi platform App U I" />

            <Button
                Text="{Binding ButtonText}" <------------------------------------------------------ ici notre variable
                SemanticProperties.Hint="Counts the number of times you click"
                Command="{Binding CounterClickedCommand}" <---------------------------------------- ici notre évent
                HorizontalOptions="Fill" />
        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

Tester !
Si vous avez bien suivis les étapes, l'application à exactement le même comportement qu'au tout début.
Alors pourquoi avoir fait tout ça si c'est pour avoir le même résulat ??
Tout simplement pour une meilleur séparation des **Responsabilité**, en effet maintenant notre View ne fait que afficher des données 'bêtement',
elle ne connais rien à la logique de l'évènement ni au contenu des variables qu'elle affiche.
En résumer :

- La **View** comme sont nom l'indique est une vue, c'est ici que tu met en place le visuel des éléments de ton écran rien de plus.
- La **ViewModel** elle s'est le cerveau, elle vas s'occuper d'implémenter la logique spécifique de chaque éléments, que se soit des variable ou des évents.

### On continue ? =D

2. Ajoutons une nouvelle page pour comprendre la logique de navigation.  
   Plusieurs solutions s'offre à nous, je vous en présente une mais n'hésitez à découvrire les autres par vous même.

- Création de ma nouvelle **View** dans le dossier **View**, clique droit sur le folder **View** et choisir _Nouvel élément_ voire image ci-dessous.

![four](/Screen/4_ajout_nouvelle_view.png)

-

![five](/Screen/5_ajout_nouvelle_view.png)

Nous avons donc une nouvelle page j'ai rajouter un petit **Label** et fait une séparation visuel entre les deux éléments voici le code :

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Tuto_MAUI.View.NewPage1"
             Title="NewPage1">
    <VerticalStackLayout>

        <Label
            Text="Welcome to .NET MAUI!"
            VerticalOptions="Center"
            HorizontalOptions="Center" />

        <Label
            Text="Cool une nouvelle page !"
            FontSize="Large" />

    </VerticalStackLayout>
</ContentPage>
```

C'est bien beau tout ça on a une nouvelle page mais comment y accède ? ^^
Direction le fichier **MainPageViewModel.cs** et ajoutons ceci :

```cs
using CommunityToolkit.Mvvm.ComponentModel;
using CommunityToolkit.Mvvm.Input;

using Tuto_MAUI.View;

namespace Tuto_MAUI.ViewModel
{
    internal partial class MainPageViewModel : ObservableObject
    {
        [ObservableProperty]
        private int _count = 0;

        [ObservableProperty]
        private string _buttonText = "Click me";

        [RelayCommand]
        private void OnCounterClicked()
        {
            Count++;

            if (Count == 1)
                ButtonText = $"Clicked {Count} time";
            else
                ButtonText = $"Clicked {Count} times";

            SemanticScreenReader.Announce(ButtonText);
        }

        [RelayCommand] // <-------------------------------- Notre évent pour la navigation vers notre nouvelle page
        private async Task NavigateToMyNewPage()
        {
            await Shell.Current.GoToAsync(nameof(NewPage1));
        }
    }
}
```

Vous l'avez surement devinez mais maintenant que notre logique de navigation est en place, il nous faut un bouton visuel pour déclencher la méthode.
Direction notre view **MainPage.xaml**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="Tuto_MAUI.View.MainPage"
             xmlns:vm="clr-namespace:Tuto_MAUI.ViewModel"
             >

    <ContentPage.BindingContext>
        <vm:MainPageViewModel/>
    </ContentPage.BindingContext>

    <ScrollView>
        <VerticalStackLayout
            Padding="30,0"
            Spacing="25">
            <Image
                Source="dotnet_bot.png"
                HeightRequest="185"
                Aspect="AspectFit"
                SemanticProperties.Description="dot net bot in a race car number eight" />

            <Label
                Text="Hello, World!"
                Style="{StaticResource Headline}"
                SemanticProperties.HeadingLevel="Level1" />

            <Label
                Text="Welcome to &#10;.NET Multi-platform App UI"
                Style="{StaticResource SubHeadline}"
                SemanticProperties.HeadingLevel="Level2"
                SemanticProperties.Description="Welcome to dot net Multi platform App U I" />

            <Button
                Text="{Binding ButtonText}"
                SemanticProperties.Hint="Counts the number of times you click"
                Command="{Binding CounterClickedCommand}"
                HorizontalOptions="Fill" />

            <Button <----------------------------------------------------------- Ajout du nouveau Bouton
                Text="Direction ma nouvelle page"
                Command="{Binding NavigateToMyNewPageCommand}" <---------------- Ajout de mon nouvelle évent
                HorizontalOptions="Fill" />

        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

Il nous faut aussi référencer cette nouvelle page dans notre application, mais ou le faire ? dans le fichier **AppShell.xaml.cs**, ajouter ceci :

```cs
using Tuto_MAUI.View;

namespace Tuto_MAUI
{
    public partial class AppShell : Shell
    {
        public AppShell()
        {
            InitializeComponent();
            Routing.RegisterRoute(nameof(NewPage1), typeof(NewPage1)); // <-------- Ajout de notre nouvelle page
        }
    }
}

```

On test ?
Parfait ! on à mis en place une navigation simple et efficace, si vous êtes observateur vous avez vu qu'il y a une flèche qui nous permet de revenir sur la page précédente, trop facile MAUI =D

### Petit récap :

On est maintenant capable de faire

- du **Binding** c'est à dire d'afficher des variables dynamiquement dans notre **View** depuis un **ViewModel**.
- d'ajouter des évent.
- de créer une navigation.

## Pour le moment nous avons utiliser le folder **View** et le folder **ViewModel**, il est temp de voire la partie **Model**.

Dans cette partie du tuto nous allons voire comment appeler et récupérer des données depuis une API
