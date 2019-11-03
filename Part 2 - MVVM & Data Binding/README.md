## MVVM & Data Binding
In Part 2 we will introduce full data binding with MVVM and retrieve the monkeys from an internet data source.

### Implementing INotifyPropertyChanged

*INotifyPropertyChanged* is important for data binding in MVVM Frameworks. This is an interface that, when implemented, lets our view know about changes to the model. We will implement it once in our `BaseViewModel` so all other view models that we can create can inherit from it.

1. In Visual Studio, open `ViewModel/BaseViewModel.cs`
2. In `BaseViewModel.cs`, implement INotifyPropertyChanged by changing this

```csharp
public class BaseViewModel
{

}
```

to this

```csharp
public class BaseViewModel : INotifyPropertyChanged
{

}
```

3. In `BaseViewModel.cs`, right click on `INotifyPropertyChanged`
4. Implement the `INotifyPropertyChanged` Interface
   - (Visual Studio Mac) In the right-click menu, select Quick Fix -> Implement Interface
   - (Visual Studio PC) In the right-click menu, select Quick Actions and Refactorings -> Implement Interface
5. In `BaseViewModel.cs`, ensure this line of code now appears:

```csharp
public event PropertyChangedEventHandler PropertyChanged;
```

6. In `BaseViewModel.cs`, create a new method called `OnPropertyChanged`
    - Note: We will call `OnPropertyChanged` whenever a property updates

```csharp
private void OnPropertyChanged([CallerMemberName] string name = null)
{

}
```

7. Add code to `OnPropertyChanged`:

```csharp
private void OnPropertyChanged([CallerMemberName] string name = null) =>
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
```

### Implementing Title, IsBusy, and IsNotBusy

We will create a backing field and accessors for a few properties. These properties will allow us to set the title on our pages and also let our view know that our view model is busy so we don't perform duplicate operations (like allowing the user to refresh the data multiple times). They are in the `BaseViewModel` because they are common for every page.

1. In `BaseViewModel.cs`, create the backing field:

```csharp
public class BaseViewModel : INotifyPropertyChanged
{
    bool isBusy;
    string title;
    //...
}
```

2. Create the properties:

```csharp
public class BaseViewModel : INotifyPropertyChanged
{
    //...
    public bool IsBusy
    {
        get => isBusy;
        set
        {
            if (isBusy == value)
                return;
            isBusy = value;
            OnPropertyChanged();
        }
    }

    public string Title
    {
        get => title;
        set
        {
            if (title == value)
                return;
            title = value;
            OnPropertyChanged();
        }
    }
    //...
}
```

Notice that we call `OnPropertyChanged` when the value changes. The Xamarin.Forms binding infrastructure will subscribe to our **PropertyChanged** event so the UI will be notified of the change.

We can also create the inverse of `IsBusy` by creating another property called `IsNotBusy` that returns the opposite of `IsBusy` and then raising the event of `OnPropertyChanged` when we set `IsBusy`

```csharp
public class MonkeysViewModel : INotifyPropertyChanged
{
    //...
    public bool IsBusy
    {
        get => isBusy;
        set
        {
            if (isBusy == value)
                return;
            isBusy = value;
            OnPropertyChanged();
            // Also raise the IsNotBusy property changed
            OnPropertyChanged(nameof(IsNotBusy));
        }
    } 

    public bool IsNotBusy => !IsBusy;
    //...
}
```

### Create MonkeysViewModel for Main Page

We will use an `ObservableCollection<Monkey>` that will be cleared and then loaded with **Monkey** objects. We use an `ObservableCollection` because it has built-in support to raise `CollectionChanged` events when we Add or Remove items from the collection. This means we don't call `OnPropertyChanged` when updating the collection.

1. In `MonkeysViewModel.cs` declare an auto-property which we will initialize to an empty collection. Also, we can set our Title to `Monkey Finder`.

```csharp
public class MonkeysViewModel : BaseViewModel
{
    //...
    public ObservableCollection<Monkey> Monkeys { get; }
    public MonkeysViewModel()
    {
        Title = "Monkey Finder";
        Monkeys = new ObservableCollection<Monkey>();
    }
    //...
}
```

### Create GetMonkeysAsync Method

We are ready to create a method named `GetMonkeysAsync` which will retrieve the monkey data from the internet. We will first implement this with a simple HTTP request using HttpClient!

1. In `MonkeysViewModel.cs`, create a method named `GetMonkeysAsync` with that returns `async Task`:

```csharp
public class MonkeysViewModel : BaseViewModel
{
    //...
    HttpClient httpClient;
    HttpClient Client => httpClient ?? (httpClient = new HttpClient());

    async Task GetMonkeysAsync()
    {
    }
    //...
}
```

2. In `GetMonkeysAsync`, first ensure `IsBusy` is false. If it is true, `return`

```csharp
async Task GetMonkeysAsync()
{
    if (IsBusy)
        return;
}
```

3. In `GetMonkeysAsync`, add some scaffolding for try/catch/finally blocks
    - Notice, that we toggle *IsBusy* to true and then false when we start to call to the server and when we finish.

```csharp
async Task GetMonkeysAsync()
{
    if (IsBusy)
        return;

    try
    {
        IsBusy = true;

    }
    catch (Exception ex)
    {

    }
    finally
    {
       IsBusy = false;
    }

}
```

4. In the `try` block of `GetMonkeysAsync`, we can get the monkeys from our Data Service.

```csharp
async Task GetMonkeysAsync()
{
    ...
    try
    {
        IsBusy = true;

        var json = await Client.GetStringAsync("https://montemagno.com/monkeys.json");
        var monkeys =  Monkey.FromJson(json);
    }
    ... 
}
```

6. Inside of the `using`, clear the `Monkeys` property and then add the new monkey data:

```csharp
async Task GetMonkeysAsync()
{
    //...
    try
    {
        IsBusy = true;

        var json = await Client.GetStringAsync("https://montemagno.com/monkeys.json");
        var monkeys =  Monkey.FromJson(json);

        Monkeys.Clear();
        foreach (var monkey in monkeys)
            Monkeys.Add(monkey);
    }
    //...
}
```

7. In `GetMonkeysAsync`, add this code to the `catch` block to display a popup if the data retrieval fails:

```csharp
async Task GetMonkeysAsync()
{
    //...
    catch(Exception ex)
    {
        Debug.WriteLine($"Unable to get monkeys: {ex.Message}");
        await Application.Current.MainPage.DisplayAlert("Error!", ex.Message, "OK");
    }
    //...
}
```

8. Ensure the completed code looks like this:

```csharp
async Task GetMonkeysAsync()
{
    if (IsBusy)
        return;

    try
    {
        IsBusy = true;

        var json = await Client.GetStringAsync("https://montemagno.com/monkeys.json");
        var monkeys =  Monkey.FromJson(json);

        Monkeys.Clear();
        foreach (var monkey in monkeys)
            Monkeys.Add(monkey);
    }
    catch (Exception ex)
    {
        Debug.WriteLine($"Unable to get monkeys: {ex.Message}");
        await Application.Current.MainPage.DisplayAlert("Error!", ex.Message, "OK");
    }
    finally
    {
        IsBusy = false;
    }
}
```

Our main method for getting data is now complete!

#### Create GetMonkeys Command

Instead of invoking this method directly, we will expose it with a `Command`. A `Command` has an interface that knows what method to invoke and has an optional way of describing if the Command is enabled.

1. In `MonkeysViewModel.cs`, create a new Command called `GetMonkeysCommand`:

```csharp
public class MonkeysViewModel : BaseViewModel
{
    //...
    public Command GetMonkeysCommand { get; }
    //...
}
```

2. Inside of the `MonkeysViewModel` constructor, create the `GetMonkeysCommand` and pass it two methods
    - One to invoke when the command is executed
    - Another that determines if the command is enabled. Both methods can be implemented as lambda expressions as shown below:

```csharp
public class MonkeysViewModel : BaseViewModel
{
    //...
    public MonkeysViewModel()
    {
        //...
        GetMonkeysCommand = new Command(async () => await GetMonkeysAsync());
    }
    //...
}
```

## Build The Monkeys User Interface
It is now time to build the Xamarin.Forms user interface in `View/MainPage.xaml`. Our end result is to build a page that looks like this:

![](../Art/FinalUI.PNG)

1. In `MainPage.xaml`, add a `BindingContext` between the `ContentPage` tags, which will enable us to get binding intellisense:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage>

    <!-- Add this -->
    <d:ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </d:ContentPage.BindingContext>

</ContentPage>
```

In the code behind for the project we will create a new BindingContext:

```csharp
public MainPage()
{
    InitializeComponent();

    // Add This
    BindingContext = new MonkeysViewModel();
}
```

2. We can create our first binding on the `ContentPage` by adding the `Title` Property:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
    Title="{Binding Title}"> <!-- Update this -->

    <ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </ContentPage.BindingContext>

</ContentPage>
```

2. In the `MainPage.xaml`, we can add a `Grid` between the `ContentPage` tags with 2 rows and 2 columns. We will also set the `RowSpacing` and `ColumnSpacing` to

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
             Title="{Binding Title}">

    <d:ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </d:ContentPage.BindingContext>

    <!-- Add this -->
    <Grid RowSpacing="0" ColumnSpacing="5">
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
    </Grid>
</ContentPage>
```

3. In the `MainPage.xaml`, we can add a `ListView` between the `Grid` tags that spans 2 Columns. We will also set the `ItemsSource` which will bind to our `Monkeys` ObservableCollection and additionally set a few properties for optimizing the list.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
             Title="{Binding Title}">

    <d:ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </d:ContentPage.BindingContext>

    <!-- Add this -->
    <Grid RowSpacing="0" ColumnSpacing="5">
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
         <ListView ItemsSource="{Binding Monkeys}"
                  HasUnevenRows="True"
                  Grid.ColumnSpan="2">

        </ListView>
    </Grid>
</ContentPage>
```

4. In the `MainPage.xaml`, we can add a `ItemTemplate` to our `ListView` that will represent what each item in the list displays:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
             Title="{Binding Title}">

    <d:ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </d:ContentPage.BindingContext>

    <Grid RowSpacing="0" ColumnSpacing="5">
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
         <ListView ItemsSource="{Binding Monkeys}"
                  HasUnevenRows="True"
                  Grid.ColumnSpan="2">
            <!-- Add this -->
            <ListView.ItemTemplate>
                <DataTemplate x:DataType="model:Monkey">
                    <ViewCell>
                         <Frame Visual="Material" 
                               IsClippedToBounds="True"
                               BackgroundColor="White"
                               InputTransparent="True"
                               Margin="10,5"
                               Padding="0"
                               CornerRadius="10"
                               HeightRequest="125">
                            <Grid ColumnSpacing="10" Padding="0">
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="125"/>
                                    <ColumnDefinition Width="*"/>
                                </Grid.ColumnDefinitions>
                                <Image Source="{Binding Image}"
                                       Aspect="AspectFill"/>
                                <StackLayout Grid.Column="1"
                                             Padding="10"
                                             VerticalOptions="Center">
                                    <Label Text="{Binding Name}" FontSize="Large"/>
                                    <Label Text="{Binding Location}" FontSize="Medium"/>
                                </StackLayout>
                            </Grid>
                        </Frame>
                    </ViewCell>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </Grid>
</ContentPage>
```

5. In the `MainPage.xaml`, we can add a `Button` under our `ListView` that will enable us to click it and get the monkeys from the server:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
    xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
    xmlns:d="http://xamarin.com/schemas/2014/forms/design"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    xmlns:viewmodel="clr-namespace:MonkeyFinder.ViewModel"
    xmlns:model="clr-namespace:MonkeyFinder.Model"
    x:Class="MonkeyFinder.View.MainPage"
    x:DataType="viewmodel:MonkeysViewModel"
    Title="{Binding Title}">

    <d:ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </d:ContentPage.BindingContext>


    <Grid RowSpacing="0" ColumnSpacing="5">
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
         <ListView ItemsSource="{Binding Monkeys}"
                  HasUnevenRows="True"
                  Grid.ColumnSpan="2">
            <ListView.ItemTemplate>
                <DataTemplate x:DataType="model:Monkey">
                    <ViewCell>
                        <Frame Visual="Material" 
                               IsClippedToBounds="True"
                               BackgroundColor="White"
                               InputTransparent="True"
                               Margin="10,5"
                               Padding="0"
                               CornerRadius="10"
                               HeightRequest="125">
                            <Grid ColumnSpacing="10" Padding="0">
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="125"/>
                                    <ColumnDefinition Width="*"/>
                                </Grid.ColumnDefinitions>
                                <Image Source="{Binding Image}"
                                       Aspect="AspectFill"/>
                                <StackLayout Grid.Column="1"
                                             Padding="10"
                                             VerticalOptions="Center">
                                    <Label Text="{Binding Name}" FontSize="Large"/>
                                    <Label Text="{Binding Location}" FontSize="Medium"/>
                                </StackLayout>
                            </Grid>
                        </Frame>
                    </ViewCell>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
        <!-- Add this -->
        <Button Text="Search" 
                Command="{Binding GetMonkeysCommand}"
                IsEnabled="{Binding IsNotBusy}"
                Grid.Row="1"
                Grid.Column="0"
                Style="{StaticResource ButtonOutline}"
                Margin="8"/>
    </Grid>
</ContentPage>
```


6. Finally, In the `MainPage.xaml`, we can add a `ActivityIndicator` above all of our controls at the very bottom or `Grid` that will show an indication that something is happening when we press the Search button.

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage 
             Title="{Binding Title}">

    <d:ContentPage.BindingContext>
        <viewmodel:MonkeysViewModel/>
    </d:ContentPage.BindingContext>

    <Grid RowSpacing="0" ColumnSpacing="5">
        <Grid.RowDefinitions>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
         <ListView ItemsSource="{Binding Monkeys}"
                  HasUnevenRows="True"
                  Grid.ColumnSpan="2">
            <ListView.ItemTemplate>
                 <DataTemplate x:DataType="model:Monkey">
                    <ViewCell>
                         <Frame Visual="Material" 
                               IsClippedToBounds="True"
                               BackgroundColor="White"
                               InputTransparent="True"
                               Margin="10,5"
                               Padding="0"
                               CornerRadius="10"
                               HeightRequest="125">
                            <Grid ColumnSpacing="10" Padding="0">
                                <Grid.ColumnDefinitions>
                                    <ColumnDefinition Width="125"/>
                                    <ColumnDefinition Width="*"/>
                                </Grid.ColumnDefinitions>
                                <Image Source="{Binding Image}"
                                       Aspect="AspectFill"/>
                                <StackLayout Grid.Column="1"
                                             Padding="10"
                                             VerticalOptions="Center">
                                    <Label Text="{Binding Name}" FontSize="Large"/>
                                    <Label Text="{Binding Location}" FontSize="Medium"/>
                                </StackLayout>
                            </Grid>
                        </Frame>
                    </ViewCell>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>

        <Button Text="Search" 
                Command="{Binding GetMonkeysCommand}"
                IsEnabled="{Binding IsNotBusy}"
                Grid.Row="1"
                Grid.Column="0"
                Style="{StaticResource ButtonOutline}"
                Margin="8"/>


        <!-- Add this -->
        <ActivityIndicator IsVisible="{Binding IsBusy}"
                           IsRunning="{Binding IsBusy}"
                           HorizontalOptions="FillAndExpand"
                           VerticalOptions="CenterAndExpand"
                           Grid.RowSpan="2"
                           Grid.ColumnSpan="2"/>
    </Grid>
</ContentPage>
```

### Run the App

1. In Visual Studio, set the iOS or Android project as the startup project 

2. In Visual Studio, click "Start Debugging"
