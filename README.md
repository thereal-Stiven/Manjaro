# SWE MonsterCardTradingGame Stepanovic

## Table of contents
* [Intro](#intro)
* [Datenbank](#datenbank)
* [Exe](#.exe)
* [Use](#Use)

## Intro

Da ich Quereinsteiger bin war ich nicht sicher wie der Aufbau der Solution sein soll da ich noch aus der HTL
weiss, dass einige darauf schauen die Logik in eigene Projektmappen zu geben etc., da dies mein erstes Programm 
seit langem war entschloss ich einfachkeitshalber alles in einer Projektmappe zu machen.

## Datenbank

Die Datenbank ist eine PostgreSQL Datenbank, hierbei habe ich mich nach oftmaligem neuinstallieren und Fehlschlaegen
entschlossen die Portable Version zu verwenden, da diese mit pgAdmin4 ausfuehrbar war.
Das Skript zum erstellen der Tabellen in der Datenbank befindet sich im ProjektOrdner
unter: exercise-StivenSt\dotnet\swe1\MyServer\SQLskript

```sql
create table cards (
	ID VARCHAR(100) primary key,
	Name VARCHAR(100),
	Damage integer,
	packageID integer
);
```
Eine Tabelle, in der die Karten mit ID,Namen,Damage und einer PackageId gespeichert werden.

```sql
create table users(
	username VARCHAR(100) primary key,
	userpwd VARCHAR(100)
);
```

Eine Tabelle, in der die Informationen des Users mit Namen und einem Passwort gespeichert werden.

```sql
create table userCredits(
	username VARCHAR(100) REFERENCES users,
	usercredit integer DEFAULT 20,
	primary key (username)
);
```

Die Tabelle usercredit Speichert eine Refferenz auf den usernamen der Tabelle users und
setzt usercredit auf 20 damit jeder neue Benutzer direkt mit 20 Credits anfaengt (zum kaufen der Packs).

```sql
create table loggedIn(
	username VARCHAR(100) REFERENCES users,
	usertoken VARCHAR(110) unique,
	primary key(username)
);
```

Mithilfe der Tabelle loggedIn wird im Code ueberprueft ob der User bereits ein usertoken hat, welches
beim einloggen erzeugt wird.

```sql
create table owns(
	username VARCHAR(100) REFERENCES users,
	cID VARCHAR(100) REFERENCES cards
);
```

Die Tabelle owns wurde im Verlauf des Projektes eingefuehrt um zu sehen welcher User welche Karten besitzt.

```sql
create table userprofiles(
	username VARCHAR(100) REFERENCES users primary key,
	Bio VARCHAR(300),
	Image VARCHAR(100)
);
```

Die Tabelle userprofiles wurde hinzugefuegt um das editieren der User Data im nachinein zu implementieren,
dies wollte ich allerdings nicht in der Tabelle users direkt machen.

Wie bereits erwaehnt hatte ich am Anfang Probleme beim installieren von PostgreSQL, welches einige Zeit
in anspruch nahm, auch beim Deserializieren der Karten hatte ich Probleme bis ich nach kurzem recherchieren
auf "Newtonsoft.Json" gestossen bin. Damit konnte ich einfach  

Card[] p = JsonConvert.DeserializeObject<Card[]>(r.ContentString);

alle Karten Informationen in ein Card-Array abspeichern.

Installierte NuGet Packete in Visual Studio
-Npgsql 
-Newtonsoft.Json

Beim ausetzen der Datenbank in pgAdmin4 muessen Lediglich folgende Informationen eingetragen werden
ein Server=localhost <-- muss eingetragen werden
Port=5432 <-- Default Value in meinem Fall
User Id=postgres <-- Angabe des Datenbank Users, kann auch ein selbsterzeugter sein mit bestimmten Berechtigungen
Password=pwdizzda <-- das Passwort des Besitzer Users der Datenbank 
Database=MonsterCardTradingGame <-- die Datenbank welche verwendet werden soll.

```csharp
 public static NpgsqlConnection GetConnection()
        {
            NpgsqlConnection con = new NpgsqlConnection(@"Server=localhost;Port=5432;User Id=postgres;Password=pwdizzda;Database=MonsterCardTradingGame;");
            return con;
        }
```

Methode zum Testen ob die Connection erfolgreich hergestellt wurde
```csharp
  public void TestConnectionToDBS()
        {
            NpgsqlConnection con = GetConnection();
            con.Open();
            if (con.State == ConnectionState.Open)
            {
                Console.WriteLine(">>>>>>> DEBUG Connected to PostgreSQL Database <<<<<<<<");
            }
            else
            {
                Console.WriteLine("ERROR while connecting");
                Console.ReadLine();
            }
        }
```

## .exe 

Die exe habe ich direkt in Visual Studio erstellt (GUI nicht Developer Console)
1) rechtsklick auf die Projektmappe
2) als Target -> Folder
3) Specific target -> Folder
4) Folder location angeben "bin\Release\netcoreapp3.1\publish\"
5) Finish
6) Publish

### Visual Studio output
1>------ Publish started: Project: MyServer, Configuration: Release Any CPU ------
1>MyServer -> C:\Users\Administrator\Documents\GitHub\exercise-StivenSt\dotnet\swe1\MyServer\bin\Release\netcoreapp3.1\MyServer.dll
1>MyServer -> C:\Users\Administrator\Documents\GitHub\exercise-StivenSt\dotnet\swe1\MyServer\bin\Release\netcoreapp3.1\publish\
========== Build: 0 succeeded, 0 failed, 1 up-to-date, 0 skipped ==========
========== Publish: 1 succeeded, 0 failed, 0 skipped ==========

## Use
### User erstellen
```curl
curl -X POST http://localhost:10001/users --header "Content-Type: application/json" -d "{\"Username\":\"NAME_DES_USERS\", \"Password\":\"PASSWORT_DES_USERS\"}"
```
### User login

curl -X POST http://localhost:10001/sessions --header "Content-Type: application/json" -d "{\"Username\":\"NAME_DES_USERS\", \"Password\":\"PASSWORT_DES_USERS\"}"

### create Package (only admin, Packs can only be created with 5 cards)

curl -X POST http://localhost:10001/packages --header "Content-Type: application/json" --header "Authorization: Basic NAME_DES_ADMINS-mtcgToken" -d "[{\"Id\":\"845f0dc7-37d0-426e-994e-43fc3ac83c08\", \"Name\":\"WaterGoblin\", \"Damage\": 10.0}, {\"Id\":\"99f8f8dc-e25e-4a95-aa2c-782823f36e2a\", \"Name\":\"Dragon\", \"Damage\": 50.0}, {\"Id\":\"e85e3976-7c86-4d06-9a80-641c2019a79f\", \"Name\":\"WaterSpell\", \"Damage\": 20.0}, {\"Id\":\"1cb6ab86-bdb2-47e5-b6e4-68c5ab389334\", \"Name\":\"Ork\", \"Damage\": 45.0}, {\"Id\":\"dfdd758f-649c-40f9-ba3a-8657f4b3439f\", \"Name\":\"FireSpell\",    \"Damage\": 25.0}]"

### acquire packages
curl -X POST http://localhost:10001/transactions/packages --header "Content-Type: application/json" --header "Authorization: Basic NAME_DES_USERS-mtcgToken" -d ""

