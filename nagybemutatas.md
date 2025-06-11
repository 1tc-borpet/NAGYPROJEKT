#  Könyvtárkezelő alkalmazás (WPF)

Ez a WPF-alapú C# alkalmazás egy egyszerű **könyvtárkezelő rendszer**, amellyel könyveket lehet:
- hozzáadni,
- műfaj és év szerint szűrni,
- valamint kölcsönözni.

##  Funkciók

-  Könyv hozzáadása (cím, szerző, év, műfaj)
-  Dinamikus műfajlista
-  Műfaj és év szerinti szűrés
-  Könyvek mentése fájlba (`konyvek.txt`)
-  Kölcsönzések naplózása (`kolcsonzesek.txt`)

##  Képernyőképek


| Főképernyő | Könyv hozzáadása |
|-----------|------------------|
| ![image](https://github.com/user-attachments/assets/a7ed15b6-cd74-496b-98ba-42b4fcda14a3)| ![image](https://github.com/user-attachments/assets/108ce17e-81a4-4ef2-aa6a-f1d6b1ba2469)



##  Használat

1. Nyisd meg a `MainWindow.xaml` fájlt Visual Studio-ban.
2. Futtasd az alkalmazást (F5).
3. Add meg a könyv adatait, majd kattints a **Hozzáadás** gombra.
4. Szűréshez válassz műfajt vagy írd be az évet.
5. Könyv kölcsönzéséhez válassz ki egy könyvet, majd kattints a **Kölcsönzés** gombra.

---

##  Teljes forráskód

<details>
<summary><strong> Kattints a kód megnyitásához</strong></summary>

### `MainWindow.xaml`
```xml
<Window x:Class="konyvtar.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Könyvtár" Height="500" Width="800">

    <Grid Margin="10">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="200"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <StackPanel Grid.Column="0" Margin="0,0,10,0">
            <TextBlock Text="Műfajok:" FontWeight="Bold" Margin="0,0,0,5"/>
            <ListBox x:Name="genreListBox" Height="200" SelectionChanged="genreListBox_SelectionChanged"/>

            <TextBlock Text="Év szerinti szűrés:" Margin="0,20,0,2"/>
            <TextBox x:Name="yearFilterBox" TextChanged="yearFilterBox_TextChanged"/>
        </StackPanel>

        <StackPanel Grid.Column="1">
            <TextBlock Text="Könyv hozzáadása:" FontWeight="Bold" Margin="0,0,0,5"/>

            <StackPanel Orientation="Horizontal" Margin="0,0,0,5">
                <TextBlock Text="Cím:" Width="50" VerticalAlignment="Center"/>
                <TextBox x:Name="titleBox" Width="200"/>
            </StackPanel>

            <StackPanel Orientation="Horizontal" Margin="0,0,0,5">
                <TextBlock Text="Szerző:" Width="50" VerticalAlignment="Center"/>
                <TextBox x:Name="authorBox" Width="200"/>
            </StackPanel>

            <StackPanel Orientation="Horizontal" Margin="0,0,0,5">
                <TextBlock Text="Év:" Width="50" VerticalAlignment="Center"/>
                <TextBox x:Name="yearBox" Width="100"/>
            </StackPanel>

            <StackPanel Orientation="Horizontal" Margin="0,0,0,10">
                <TextBlock Text="Műfaj:" Width="50" VerticalAlignment="Center"/>
                <ComboBox x:Name="genreComboBox" Width="150" IsEditable="True"/>
            </StackPanel>

            <StackPanel Orientation="Horizontal" Margin="0,0,0,10" >
                <Button Content="Hozzáadás" Click="AddBook_Click" Width="100" Margin="0,0,5,0"/>
                <Button Content="Kölcsönzés" Click="BorrowBook_Click" Width="100"/>
            </StackPanel>

            <TextBlock Text="Könyvek:" FontWeight="Bold" Margin="0,0,0,5"/>
            <ListView x:Name="bookListView" Height="250" >
                <ListView.View>
                    <GridView>
                        <GridViewColumn Header="Cím" Width="200" DisplayMemberBinding="{Binding Title}"/>
                        <GridViewColumn Header="Szerző" Width="150" DisplayMemberBinding="{Binding Author}"/>
                        <GridViewColumn Header="Év" Width="50" DisplayMemberBinding="{Binding Year}"/>
                        <GridViewColumn Header="Műfaj" Width="100" DisplayMemberBinding="{Binding Genre}"/>
                    </GridView>
                </ListView.View>
            </ListView>
        </StackPanel>
    </Grid>
</Window>
```
### `MainWindow.xaml.cs`
```xaml
using System;
using System.Collections.ObjectModel;
using System.IO;
using System.Linq;
using System.Windows;
using System.Windows.Controls;

namespace konyvtar
{
    public class Book
    {
        public string Title { get; set; }
        public string Author { get; set; }
        public int Year { get; set; }
        public string Genre { get; set; }

        public override string ToString() => $"{Title} - {Author} ({Year})";
    }

    public partial class MainWindow : Window
    {
        private ObservableCollection<Book> allBooks = new ObservableCollection<Book>();
        private ObservableCollection<string> genres = new ObservableCollection<string>();

        public MainWindow()
        {
            InitializeComponent();

            genres.Add("Életrajz");
            genres.Add("Történelem");

            allBooks.Add(new Book { Title = "HErBY - A magyar vlogger", Author = "Egyed Anikó", Year = 2023, Genre = "Életrajz" });
            allBooks.Add(new Book { Title = "Rengéshullámok", Author = "Helikon Kiadó", Year = 2010, Genre = "Történelem" });
            allBooks.Add(new Book { Title = "Paprikaföldektől a HÉTCSILLAGOS életig", Author = "Jákob Zoltán", Year = 2023, Genre = "Életrajz" });

            genreListBox.ItemsSource = genres;
            genreComboBox.ItemsSource = genres;
            bookListView.ItemsSource = allBooks;
        }

        private void AddBook_Click(object sender, RoutedEventArgs e)
        {
            string title = titleBox.Text.Trim();
            string author = authorBox.Text.Trim();
            string genre = genreComboBox.Text.Trim();
            bool validYear = int.TryParse(yearBox.Text, out int year);

            if (string.IsNullOrWhiteSpace(title) || string.IsNullOrWhiteSpace(author) ||
                string.IsNullOrWhiteSpace(genre) || !validYear)
            {
                MessageBox.Show("Tölts ki minden mezőt helyesen.");
                return;
            }

            if (year > DateTime.Now.Year)
            {
                MessageBox.Show("Az év nem lehet a jövőben.");
                return;
            }

            Book book = new Book { Title = title, Author = author, Year = year, Genre = genre };
            allBooks.Add(book);

            if (!genres.Contains(genre))
                genres.Add(genre);

            SaveBookToFile(book);
            ClearInputs();
            ApplyFilters();
        }

        private void BorrowBook_Click(object sender, RoutedEventArgs e)
        {
            if (bookListView.SelectedItem is Book book)
            {
                SaveBorrowedBook(book);
                allBooks.Remove(book);
                ApplyFilters(); 
                MessageBox.Show($"A(z) \"{book.Title}\" könyv kölcsönözve.");
            }
            else
            {
                MessageBox.Show("Válassz ki egy könyvet a kölcsönzéshez.");
            }
        }


        private void SaveBookToFile(Book book)
        {
            Directory.CreateDirectory("Data");
            File.AppendAllText("Data/konyvek.txt", $"{book.Title};{book.Author};{book.Year};{book.Genre}{Environment.NewLine}");
        }

        private void SaveBorrowedBook(Book book)
        {
            Directory.CreateDirectory("Data");
            File.AppendAllText("Data/kolcsonzesek.txt", $"{DateTime.Now:yyyy-MM-dd HH:mm}: {book.Title} - {book.Author} ({book.Year}) [{book.Genre}]{Environment.NewLine}");
        }

        private void ClearInputs()
        {
            titleBox.Text = authorBox.Text = yearBox.Text = "";
            genreComboBox.Text = "";
        }

        private void genreListBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            ApplyFilters();
        }

        private void yearFilterBox_TextChanged(object sender, TextChangedEventArgs e)
        {
            ApplyFilters();
        }

        private void ApplyFilters()
        {
            IEnumerable<Book> filtered = allBooks.AsEnumerable();

            if (genreListBox.SelectedItem is string selectedGenre)
                filtered = filtered.Where(b => b.Genre == selectedGenre);

            if (int.TryParse(yearFilterBox.Text, out int filterYear))
                filtered = filtered.Where(b => b.Year == filterYear);

            bookListView.ItemsSource = new ObservableCollection<Book>(filtered);
        }
    }
}
```
</details>

##  Fejlesztette

Morzsa Milán Dominik,Boros Péter,ChatGPT
