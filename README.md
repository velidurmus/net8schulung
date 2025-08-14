# C# Migration Schulung: Von .NET Framework 4.8 zu .NET 8.0

## Überblick
Die wichtigsten C# Features der Versionen 8.0 bis 12.0 im Kontext der WAWI-Migration von .NET Framework 4.8 zu .NET 8.0.

**Ausgangslage**: C# 7.3 (.NET Framework 4.8)  
**Ziel**: C# 12.0 (.NET 8.0)

**Das ist ein großer Sprung!** 5 Major-Versionen mit vielen revolutionären Features.

### Versionsübersicht
- **C# 8.0** (2019): Nullable Reference Types, Switch Expressions, Async Streams
- **C# 9.0** (2020): Records, Init-only Properties, Top-level Statements  
- **C# 10.0** (2021): Global Usings, File-scoped Namespaces, Record Structs
- **C# 11.0** (2022): Raw Strings, List Patterns, Required Members
- **C# 12.0** (2023): Primary Constructors, Collection Expressions
- **C# 13.0** (2024): Escape Sequence, Params Collections
- **C# 14.0** (Preview): Field Keyword, Partial Properties

### - Was wir gewinnen
Die neuen C# Features machen Code:
- **Sicherer** (Nullable Reference Types, Required Members)
- **Lesbarer** (Switch Expressions, Records, Raw Strings)
- **Wartbarer** (weniger Boilerplate, mehr Expressions)
- **Performanter** (Async Streams, Span<T>, strukturelle Verbesserungen)
- **Moderner** (Current Best Practices, Community Standards)

### - Impact auf die WAWI-Codebase
- **-30% Code-Zeilen** durch Records, Primary Constructors, Global Usings
- **+90% Type Safety** durch Nullable Reference Types
- **-50% NullRef Exceptions** durch bessere Null-Handling
- **+100% Lesbarkeit** bei Business Logic (Switch Expressions, Pattern Matching)

### - Sofortige Quick Wins
1. File-scoped Namespaces = Weniger Einrückung
2. Global Usings = Keine using-Duplikation
3. Collection Expressions = Einheitliche Syntax
4. Primary Constructors = Weniger DI-Boilerplate

### - Lernkurve
- **Easy**: File-scoped, Global Usings, Collection Expressions
- **Medium**: Records, Primary Constructors, Switch Expressions
- **Advanced**: Nullable Types, Pattern Matching, Async Streams

**Wichtig**: Schrittweise Migration, Features dort einsetzen wo sie Sinn machen!

---

## C# 8.0 - Die Foundation für moderne C# Entwicklung

### 1. Nullable Reference Types 

**Das Problem**: NullReferenceException ist einer der häufigsten Fehler in C#.

```csharp
// Vorher (.NET Framework 4.8)
public class KundenService
{
    public string GetKundenName(int kundenId)
    {
        var kunde = Repository.GetKunde(kundenId);
        return kunde.Name; // Potentielle NullReferenceException!
    }
}
```

```csharp
// Nachher (C# 8.0 mit nullable reference types)
public class KundenService
{
    public string? GetKundenName(int kundenId)
    {
        Kunde? kunde = Repository.GetKunde(kundenId);
        return kunde?.Name; // Compiler warnt vor möglichem null
    }
    
    public string GetKundenNameSafe(int kundenId)
    {
        Kunde? kunde = Repository.GetKunde(kundenId);
        return kunde?.Name ?? "Unbekannter Kunde"; // Sichere Variante
    }
}
```

**Aktivierung**: In der .csproj Datei
```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

**WAWI-Empfehlung**: Aktivieren für neue Klassen, schrittweise Migration

---

### 2. Switch Expressions 

**Das Problem**: Verbose switch statements mit viel Boilerplate Code.

```csharp
// Vorher - Traditionelles switch statement
public string GetArtikelStatus(ArtikelStatusEnum status)
{
    switch (status)
    {
        case ArtikelStatusEnum.Verfuegbar:
            return "Verfügbar";
        case ArtikelStatusEnum.NichtVerfuegbar:
            return "Nicht verfügbar";
        case ArtikelStatusEnum.Auslaufartikel:
            return "Auslaufartikel";
        default:
            return "Unbekannt";
    }
}
```

```csharp
// Nachher - Switch expression
public string GetArtikelStatus(ArtikelStatusEnum status) => status switch
{
    ArtikelStatusEnum.Verfuegbar => "Verfügbar",
    ArtikelStatusEnum.NichtVerfuegbar => "Nicht verfügbar",
    ArtikelStatusEnum.Auslaufartikel => "Auslaufartikel",
    _ => "Unbekannt"
};

// Komplex: Mit Pattern Matching
public decimal GetVersandkostenRabatt(Kunde kunde, decimal bestellwert) => (kunde.Typ, bestellwert) switch
{
    (KundenTyp.Premium, > 100) => 1.0m,
    (KundenTyp.Premium, _) => 0.5m,
    (_, > 200) => 0.3m,
    _ => 0m
};
```

**WAWI-Empfehlung**: Verwenden für einfache Mappings und Berechnungen

---

### 3. Property Patterns 

```csharp
// WAWI-Beispiel: Bestellstatus prüfen
public bool IstBestellungVersandfertig(Bestellung bestellung) => bestellung switch
{
    { Status: BestellStatus.Bezahlt, LagerVerfuegbar: true, VersandArt: not null } => true,
    _ => false
};

// Verschachtelte Properties
public string GetRabattKategorie(Kunde kunde) => kunde switch
{
    { Adresse.Land: "DE", Umsatz: > 10000 } => "Premium DE",
    { Adresse.Land: "AT", Umsatz: > 5000 } => "Premium AT",
    { Registrierungsdatum: var datum } when datum > DateTime.Now.AddYears(-1) => "Neukunde",
    _ => "Standard"
};
```

**WAWI-Empfehlung**: Gut lesbar für Business Logic

---

### 4. Async Streams (IAsyncEnumerable) 

```csharp
// Vorher (C# 7.3) - Alle Daten auf einmal laden
public async Task<List<Artikel>> GetAlleArtikelAsync()
{
    var artikel = new List<Artikel>();
    var page = 1;
    while (true)
    {
        var batch = await Repository.GetArtikelPageAsync(page, 1000);
        if (!batch.Any()) break;
        artikel.AddRange(batch);
        page++;
    }
    return artikel;
}

// Nachher (C# 8.0) - Streaming mit yield
public async IAsyncEnumerable<Artikel> GetAlleArtikelStreamAsync()
{
    var page = 1;
    while (true)
    {
        var batch = await Repository.GetArtikelPageAsync(page, 1000);
        if (!batch.Any()) yield break;
        
        foreach (var artikel in batch)
        {
            yield return artikel;
        }
        page++;
    }
}

// Verwendung
await foreach (var artikel in service.GetAlleArtikelStreamAsync())
{
    Console.WriteLine(artikel.Name); // Artikel wird verarbeitet sobald verfügbar
}
```

**WAWI-Empfehlung**: Perfekt für große Datenmengen, Export-Funktionen

---

### 5. Using Declarations 

```csharp
// Vorher (C# 7.3) - Explizite using statements
public void ExportArtikelData()
{
    using (var connection = new SqlConnection(connectionString))
    using (var command = new SqlCommand(sql, connection))
    using (var writer = new StreamWriter("export.csv"))
    {
        connection.Open();
        var reader = command.ExecuteReader();
        // Export logic
    } // Alle Ressourcen werden hier disposed
}

// Nachher (C# 8.0) - Using declarations
public void ExportArtikelData()
{
    using var connection = new SqlConnection(connectionString);
    using var command = new SqlCommand(sql, connection);
    using var writer = new StreamWriter("export.csv");
    
    connection.Open();
    var reader = command.ExecuteReader();
    // Export logic
    
    // Automatisches Dispose am Ende der Methode
}
```

**WAWI-Empfehlung**: Deutlich sauberer, weniger Verschachtelung

---

### 6. Null-coalescing Assignment (??=) 

```csharp
// Vorher (C# 7.3)
if (_cache == null)
{
    _cache = new Dictionary<int, Artikel>();
}

// Vorher alternative
_cache = _cache ?? new Dictionary<int, Artikel>();

// Nachher (C# 8.0) - Kürzer und klarer
_cache ??= new Dictionary<int, Artikel>();

// WAWI-Beispiele
public class ArtikelService
{
    private List<Artikel>? _artikelCache;
    private ILogger? _logger;
    
    public List<Artikel> GetArtikel()
    {
        _artikelCache ??= LoadArtikelFromDatabase();
        return _artikelCache;
    }
    
    public void Log(string message)
    {
        _logger ??= LogManager.GetLogger("ArtikelService");
        _logger.Info(message);
    }
}
```

**WAWI-Empfehlung**: Perfekt für Lazy Loading und Caching

---

### 7. Ranges und Indices 

```csharp
// Vorher
public string[] GetLetzteBestellungen(string[] bestellungen, int anzahl)
{
    var result = new string[anzahl];
    Array.Copy(bestellungen, bestellungen.Length - anzahl, result, 0, anzahl);
    return result;
}

// Nachher - viel klarer
public string[] GetLetzteBestellungen(string[] bestellungen, int anzahl)
{
    return bestellungen[^anzahl..]; // Letzte 'anzahl' Elemente
}

// Weitere Beispiele
var artikelListe = new string[] { "Artikel1", "Artikel2", "Artikel3", "Artikel4", "Artikel5" };

var letzterArtikel = artikelListe[^1];        // "Artikel5"
var ersteZwei = artikelListe[..2];            // ["Artikel1", "Artikel2"]  
var letzteZwei = artikelListe[^2..];          // ["Artikel4", "Artikel5"]
var mittlere = artikelListe[1..^1];           // ["Artikel2", "Artikel3", "Artikel4"]
```

**WAWI-Empfehlung**: Spart Code, aber Vorsicht bei Performance-kritischen Bereichen

---

### 8. Static Local Functions 

```csharp
// Problem: Lokale Funktionen können versehentlich lokale Variablen capturen
public decimal BerechneGesamtpreis(BestellPosition[] positionen)
{
    var mwstSatz = 0.19m; // Diese Variable könnte versehentlich captured werden
    
    // C# 7.3 - Lokale Funktion mit potentiellem Capture
    decimal BerechnePositionspreis(BestellPosition pos)
    {
        return pos.Menge * pos.Einzelpreis * (1 + mwstSatz); // Capture!
    }
    
    return positionen.Sum(BerechnePositionspreis);
}

// C# 8.0 - Static local function verhindert versehentliches Capture
public decimal BerechneGesamtpreis(BestellPosition[] positionen)
{
    var mwstSatz = 0.19m;
    
    // Muss explizit als Parameter übergeben werden
    static decimal BerechnePositionspreis(BestellPosition pos, decimal mwst)
    {
        return pos.Menge * pos.Einzelpreis * (1 + mwst);
    }
    
    return positionen.Sum(pos => BerechnePositionspreis(pos, mwstSatz));
}
```

**WAWI-Empfehlung**: Verhindert Bugs, bessere Performance

---

### 9. Readonly Members in Structs 

```csharp
// WAWI Value Objects als readonly structs
public struct Geldwert
{
    public decimal Wert { get; }
    public string Waehrung { get; }
    
    public Geldwert(decimal wert, string waehrung)
    {
        Wert = wert;
        Waehrung = waehrung;
    }
    
    // Readonly member - garantiert keine Mutation
    public readonly string FormatAsCurrency() 
    {
        return $"{Wert:C} {Waehrung}";
    }
    
    // Readonly member für Berechnungen
    public readonly Geldwert Add(Geldwert other)
    {
        if (Waehrung != other.Waehrung)
            throw new ArgumentException("Währungen müssen gleich sein");
            
        return new Geldwert(Wert + other.Wert, Waehrung);
    }
}
```

**WAWI-Empfehlung**: Perfekt für Value Objects und DTOs

---

### 10. Default Interface Methods 

```csharp
// Erweitert bestehende Interfaces ohne Breaking Changes
public interface IArtikelService
{
    Task<Artikel> GetArtikelAsync(int id);
    
    // Neue Methode mit Default-Implementation
    Task<Artikel[]> GetArtikelBatchAsync(int[] ids)
    {
        var tasks = ids.Select(id => GetArtikelAsync(id));
        return Task.WhenAll(tasks);
    }
}
```

**WAWI-Empfehlung**: Sparsam verwenden, kann Architecture verwässern

---

### 11. Interpolated Verbatim Strings Enhancement 

```csharp
// C# 7.3 - Verbatim strings mit Interpolation waren umständlich
var templateOld = $@"SELECT * 
                     FROM Artikel 
                     WHERE Name LIKE '%{suchtext}%' 
                     AND Preis > {minPreis}";

// C# 8.0 - Beide Reihenfolgen funktionieren
var template1 = $@"SELECT * 
                   FROM Artikel 
                   WHERE Name LIKE '%{suchtext}%' 
                   AND Preis > {minPreis}";
                   
var template2 = @$"SELECT * 
                   FROM Artikel 
                   WHERE Name LIKE '%{suchtext}%' 
                   AND Preis > {minPreis}";

// WAWI-Beispiel: SQL-Template für dynamische Queries
var dynamicSql = @$"SELECT 
                        a.ArtikelId,
                        a.Name,
                        a.Preis
                     FROM Artikel a
                     WHERE 1=1
                     {(kategorieId.HasValue ? $"AND a.KategorieId = {kategorieId}" : "")}
                     {(nurVerfuegbar ? "AND a.Verfuegbar = 1" : "")}
                     ORDER BY a.Name";
```

**WAWI-Empfehlung**: Sehr praktisch für SQL-Queries und Templates

---

## C# 9.0 - Records und Immutability (Kollegen-Teil - Überblick)

> **Hinweis**: Dein Kollege behandelt C# 9.0 im Detail. Hier nur die wichtigsten Features im Überblick.

### Records 
```csharp
// C# 7.3 - Viel Boilerplate für Value Objects
public class ArtikelInfoOld
{
    public string Name { get; set; }
    public decimal Preis { get; set; }
    
    public override bool Equals(object obj) { /* 10 Zeilen Code */ }
    public override int GetHashCode() { /* 5 Zeilen Code */ }
    // ToString, Copy-Constructor etc.
}

// C# 9.0 - Records
public record ArtikelInfo(string Name, decimal Preis);

// Automatisch: Equals, GetHashCode, ToString, Deconstruct, Copy (with)
var artikel = new ArtikelInfo("Laptop", 999m);
var teurer = artikel with { Preis = 1299m }; // Non-destructive mutation
```

### Init-only Properties 
```csharp
// Immutable Objects ohne Constructor-Overhead
public class Bestellung
{
    public int Id { get; init; }
    public string KundenName { get; init; }
    public DateTime Datum { get; init; }
}

var bestellung = new Bestellung 
{ 
    Id = 1, 
    KundenName = "Max Mustermann", 
    Datum = DateTime.Now 
};
// bestellung.Id = 2; // Compile Error!
```

---

## C# 10.0 - Moderne Syntax und weniger Boilerplate

### 1. Interpolated String Handlers 

**Das Problem**: String-Interpolation kann Performance-Probleme verursachen, besonders beim Logging.

```csharp
// C# 7.3 - Problematisch bei conditional logging
public void ProcessArtikel(Artikel artikel)
{
    // String wird IMMER aufgebaut, auch wenn nicht geloggt!
    logger.LogDebug($"Processing article {artikel.Name} with price {artikel.Preis:C} at {DateTime.Now:yyyy-MM-dd}");
}

// C# 10.0 - String Handlers optimieren das
public void ProcessArtikel(Artikel artikel)
{
    // String wird nur aufgebaut wenn Debug-Level aktiv ist!
    logger.LogDebug($"Processing article {artikel.Name} with price {artikel.Preis:C} at {DateTime.Now:yyyy-MM-dd}");
}

// Eigene String Handler für WAWI-spezifische Formate
[InterpolatedStringHandler]
public ref struct ArtikelLogHandler
{
    private StringBuilder builder;
    
    public ArtikelLogHandler(int literalLength, int formattedCount)
    {
        builder = new StringBuilder(literalLength);
    }
    
    public void AppendLiteral(string s) => builder.Append(s);
    public void AppendFormatted(Artikel artikel) => builder.Append($"[{artikel.Id}] {artikel.Name}");
    public void AppendFormatted(decimal preis) => builder.Append(preis.ToString("C"));
    
    public override string ToString() => builder.ToString();
}
```

**WAWI-Empfehlung**: Automatische Optimierung, bessere Performance

---

### 2. File-scoped Namespaces 

```csharp
// Vorher
namespace JTL.Wawi.ArtikelVerwaltung.Services
{
    public class ArtikelService
    {
        // Implementation
    }
}

// Nachher - Spart eine Einrückungsebene
namespace JTL.Wawi.ArtikelVerwaltung.Services;

public class ArtikelService
{
    // Implementation
}
```

**WAWI-Empfehlung**: Für alle neuen Dateien verwenden

---

### 3. Global Using Directives 

```csharp
// GlobalUsings.cs (neue Datei)
global using System;
global using System.Collections.Generic;
global using System.Linq;
global using System.Threading.Tasks;
global using JTL.Framework.Datenbank;
global using JTL.Framework.Logging;
global using JTL.Wawi.Entities;

// In anderen Dateien sind diese usings automatisch verfügbar
namespace JTL.Wawi.ArtikelVerwaltung.Services;

public class ArtikelService // Keine using statements nötig!
{
    private readonly ILogger _logger; // aus JTL.Framework.Logging
    private readonly IDatabaseContext _context; // aus JTL.Framework.Datenbank
}
```

**WAWI-Empfehlung**: Reduziert Duplikation erheblich

---

### 4. Record Structs 

```csharp
// Für DTOs und Value Objects
public readonly record struct ArtikelSuchFilter(
    string? Suchtext,
    int? KategorieId,
    decimal? MinPreis,
    decimal? MaxPreis,
    bool NurVerfuegbare
);

// Verwendung
var filter = new ArtikelSuchFilter("Laptop", 5, 100m, 2000m, true);

// Immutability mit with-Expressions
var neuerFilter = filter with { MaxPreis = 1500m };
```

**WAWI-Empfehlung**: Perfekt für DTOs, API-Parameter, Filter-Objekte

---

### 5. Const Interpolated Strings 

```csharp
// C# 7.3 - Konstante Strings konnten nicht interpoliert werden
const string BaseUrl = "https://api.jtl-software.de";
const string ApiVersion = "v2";
// const string FullUrl = $"{BaseUrl}/{ApiVersion}"; // Compile Error!
const string FullUrl = BaseUrl + "/" + ApiVersion; // Ugly workaround

// C# 10.0 - Konstante interpolierte Strings
const string BaseUrl = "https://api.jtl-software.de";
const string ApiVersion = "v2";
const string FullUrl = $"{BaseUrl}/{ApiVersion}"; // Funktioniert!

// WAWI-Beispiele
const string DatabaseTablePrefix = "wawi_";
const string ArtikelTable = $"{DatabaseTablePrefix}artikel";
const string KundenTable = $"{DatabaseTablePrefix}kunden";
const string BestellungenTable = $"{DatabaseTablePrefix}bestellungen";

// Error Messages
const string ModuleName = "ArtikelVerwaltung";
const string ErrorPrefix = $"[{ModuleName}]"; 
const string NullArtikelError = $"{ErrorPrefix} Artikel darf nicht null sein";
const string InvalidPriceError = $"{ErrorPrefix} Preis muss positiv sein";
```

**WAWI-Empfehlung**: Perfekt für Konstanten und Error Messages

---

### 6. Lambda Improvements 

```csharp
// Natural type inference
var parse = (string s) => int.Parse(s);
var choose = object (bool b) => b ? 1 : "two"; // Explicit return type

// Mit Attributen
public void ProcessArtikel(Artikel[] artikel)
{
    var filtered = artikel
        .Where([MethodImpl(MethodImplOptions.AggressiveInlining)] 
               (artikel) => artikel.Verfuegbar && artikel.Preis > 0)
        .ToArray();
}
```

**WAWI-Empfehlung**: Sparsam verwenden, Lesbarkeit wichtiger als Kürze

---

### 7. Extended Property Patterns 

```csharp
// C# 8.0 - Basis Property Patterns
public string GetBestellStatus(Bestellung bestellung) => bestellung switch
{
    { Status: BestellStatus.Bezahlt } => "Bezahlt",
    { Status: BestellStatus.Versendet } => "Versendet",
    _ => "Unbekannt"
};

// C# 10.0 - Extended Property Patterns (ohne Wiederholung)
public string GetBestellStatus(Bestellung bestellung) => bestellung.Status switch
{
    BestellStatus.Bezahlt => "Bezahlt",
    BestellStatus.Versendet => "Versendet",
    BestellStatus.Storniert => "Storniert",
    _ => "Unbekannt"
};

// Komplexere Beispiele
public decimal GetVersandkosten(Bestellung bestellung) => bestellung switch
{
    { Kunde.Typ: KundenTyp.Premium } => 0m,
    { Gesamtwert: > 100 } => 0m,
    { LieferAdresse.Land: "DE" } => 4.99m,
    { LieferAdresse.Land: "AT" or "CH" } => 7.99m,
    _ => 12.99m
};

// Nested property patterns
public bool IstExpressVersand(Bestellung bestellung) => bestellung switch
{
    { VersandArt.Code: "EXPRESS", LieferAdresse.Land: "DE" } => true,
    { VersandArt: { Code: "OVERNIGHT" } } => true,
    _ => false
};
```

**WAWI-Empfehlung**: Sehr lesbar für Business Logic

---

### 8. CallerArgumentExpression Attribute 

```csharp
// Verbesserte Validierung und Logging
public static void ValidateNotNull<T>(T value, [CallerArgumentExpression("value")] string? paramName = null)
    where T : class
{
    if (value == null)
        throw new ArgumentNullException(paramName);
}

public static void ValidatePositive(decimal value, [CallerArgumentExpression("value")] string? paramName = null)
{
    if (value <= 0)
        throw new ArgumentException($"Wert muss positiv sein", paramName);
}

// Verwendung
public void CreateBestellung(Kunde kunde, decimal gesamtpreis)
{
    ValidateNotNull(kunde); // Exception message: "kunde darf nicht null sein"
    ValidatePositive(gesamtpreis); // Exception message: "gesamtpreis muss positiv sein"
    
    // Business logic
}

// WAWI Logging Helper
public static class WawiLogger
{
    public static void LogValue<T>(T value, [CallerArgumentExpression("value")] string? expression = null)
    {
        Console.WriteLine($"{expression} = {value}");
    }
}

// Verwendung
var artikel = GetArtikel(123);
WawiLogger.LogValue(artikel.Preis); // Output: "artikel.Preis = 49.99"
WawiLogger.LogValue(artikel.Name); // Output: "artikel.Name = Laptop"
```

**WAWI-Empfehlung**: Großartig für Validation und Debugging

---

## C# 11.0 - String-Verbesserungen und Patterns (Kollegen-Teil - Überblick)

> **Hinweis**: Dein Kollege behandelt C# 11.0 im Detail. Hier die wichtigsten Features.

### Raw String Literals 
```csharp
// C# 7.3 - Escape-Hell bei komplexen Strings
string sqlQuery = "SELECT * FROM Artikel WHERE Name LIKE '%\\%' AND Beschreibung LIKE '%\"Premium\"'";
string jsonTemplate = "{\"name\": \"{{NAME}}\", \"price\": {{PRICE}}}";

// C# 11.0 - Raw Strings
string sqlQuery = """
    SELECT * FROM Artikel 
    WHERE Name LIKE '%\%' 
    AND Beschreibung LIKE '%"Premium"'
    """;

string jsonTemplate = """
    {
        "name": "{{NAME}}",
        "price": {{PRICE}}
    }
    """;
```

### List Patterns 
```csharp
// Pattern Matching für Listen/Arrays
public string AnalyzeBestellpositionen(BestellPosition[] positionen) => positionen switch
{
    [] => "Leere Bestellung",
    [var single] => $"Einzelposition: {single.ArtikelName}",
    [var first, var second] => $"Zwei Artikel: {first.ArtikelName}, {second.ArtikelName}",
    [var first, .. var middle, var last] => $"Von {first.ArtikelName} bis {last.ArtikelName} ({middle.Length + 2} Artikel)",
    _ => "Komplexe Bestellung"
};
```

### Required Members 
```csharp
public class Kunde
{
    public required string Name { get; init; }
    public required string Email { get; init; }
    public string? Telefon { get; init; }
}

// var kunde = new Kunde(); // Compile Error!
var kunde = new Kunde { Name = "Max", Email = "max@test.de" }; // OK
```

---

## C# 12.0 - Noch mehr Syntax-Zucker

### 1. Ref Readonly Parameters 

```csharp
// Problem: Large structs als Parameter sind langsam
public struct ArtikelStatistik
{
    public int Anzahl { get; set; }
    public decimal Durchschnittspreis { get; set; }
    public decimal Gesamtwert { get; set; }
    public DateTime LetzteAktualisierung { get; set; }
    // ... viele weitere Properties
}

// C# 7.3 - Copy bei jedem Parameter-Pass
public void AnalyzeStatistik(ArtikelStatistik statistik) // Komplette Kopie!
{
    Console.WriteLine($"Anzahl: {statistik.Anzahl}");
}

// C# 12.0 - ref readonly vermeidet Kopie UND Mutation
public void AnalyzeStatistik(ref readonly ArtikelStatistik statistik)
{
    Console.WriteLine($"Anzahl: {statistik.Anzahl}");
    // statistik.Anzahl = 100; // Compile Error - readonly!
}

// Verwendung
var stats = new ArtikelStatistik { Anzahl = 1000, Gesamtwert = 50000m };
AnalyzeStatistik(in stats); // 'in' beim Aufruf
```

**WAWI-Empfehlung**: Performance-kritische Stellen mit großen Structs

---

### 2. Primary Constructors 

```csharp
// Vorher
public class ArtikelService
{
    private readonly IArtikelRepository _repository;
    private readonly ILogger _logger;
    
    public ArtikelService(IArtikelRepository repository, ILogger logger)
    {
        _repository = repository;
        _logger = logger;
    }
}

// Nachher - Primary Constructor
public class ArtikelService(IArtikelRepository repository, ILogger logger)
{
    public async Task<Artikel?> GetArtikelAsync(int id)
    {
        logger.LogInformation("Lade Artikel {Id}", id);
        return await repository.GetByIdAsync(id);
    }
}

// Kombiniert mit Properties
public class ArtikelViewModel(Artikel artikel)
{
    public string Name => artikel.Name;
    public decimal Preis => artikel.Preis;
    public string FormattedPreis => $"{artikel.Preis:C}";
}
```

**WAWI-Empfehlung**: Sehr gut für Services mit DI

---

### 3. Collection Expressions 

```csharp
// Vorher
var statusListe = new List<string> { "Aktiv", "Inaktiv", "Gesperrt" };
var artikelIds = new[] { 1, 2, 3, 4, 5 };

// Nachher
List<string> statusListe = ["Aktiv", "Inaktiv", "Gesperrt"];
int[] artikelIds = [1, 2, 3, 4, 5];

// Spread operator
int[] baseIds = [1, 2, 3];
int[] erweitertIds = [..baseIds, 4, 5, 6]; // [1, 2, 3, 4, 5, 6]

// WAWI-Beispiel: Filter kombinieren  
public ArtikelFilter CreateFullFilter()
{
    string[] baseFilter = ["Verfügbar", "Aktiv"];
    string[] erweitert = [..baseFilter, "Premium", "Neu"];
    return new ArtikelFilter(erweitert);
}
```

**WAWI-Empfehlung**: Sehr praktisch, einheitliche Syntax

---

### 4. Inline Arrays 

```csharp
// C# 12.0 - Inline Arrays für feste Buffer
[System.Runtime.CompilerServices.InlineArray(10)]
public struct ArtikelIdBuffer
{
    private int _element0;
}

// WAWI-Beispiel: Effizienter Buffer für häufige Operationen
public class ArtikelBatchProcessor
{
    public void ProcessBatch(ReadOnlySpan<int> artikelIds)
    {
        if (artikelIds.Length <= 10)
        {
            // Stack-allocated buffer für kleine Batches
            ArtikelIdBuffer buffer = default;
            Span<int> bufferSpan = buffer;
            artikelIds.CopyTo(bufferSpan);
            
            // Verarbeitung ohne Heap-Allocation
            ProcessOnStack(bufferSpan[..artikelIds.Length]);
        }
        else
        {
            // Heap allocation nur bei größeren Batches
            var array = artikelIds.ToArray();
            ProcessOnHeap(array);
        }
    }
    
    private void ProcessOnStack(Span<int> ids) { /* ... */ }
    private void ProcessOnHeap(int[] ids) { /* ... */ }
}
```

**WAWI-Empfehlung**: Nur für Performance-kritische Bereiche

---

### 5. Alias Any Type 

```csharp
// C# 7.3 - Nur named types
using StringDictionary = System.Collections.Generic.Dictionary<string, string>;

// C# 12.0 - Alle Typen!
using ArtikelList = System.Collections.Generic.List<JTL.Wawi.Entities.Artikel>;
using ArtikelDict = System.Collections.Generic.Dictionary<int, JTL.Wawi.Entities.Artikel>;
using PriceRange = (decimal Min, decimal Max);
using ProcessResult = (bool Success, string ErrorMessage, int ProcessedCount);
using ValidationFunc = System.Func<JTL.Wawi.Entities.Artikel, bool>;

// WAWI-Usage
public class ArtikelService
{
    private readonly ArtikelDict _cache = new();
    
    public ProcessResult ValidateArticles(ArtikelList artikel, ValidationFunc validator)
    {
        var processed = 0;
        foreach (var item in artikel)
        {
            if (!validator(item))
                return (false, $"Validation failed for {item.Name}", processed);
            processed++;
        }
        return (true, "All valid", processed);
    }
    
    public ArtikelList GetByPriceRange(PriceRange range)
    {
        return _cache.Values
            .Where(a => a.Preis >= range.Min && a.Preis <= range.Max)
            .ToList();
    }
}
```

**WAWI-Empfehlung**: Sehr praktisch für komplexe generische Typen

---

### 6. Optional Lambda Parameters 

```csharp
// Lambda mit Default-Werten
var berechneRabatt = (decimal preis, decimal rabatt = 0.1m) => preis * (1 - rabatt);

var endpreis1 = berechneRabatt(100m);        // 90€ (10% Rabatt)
var endpreis2 = berechneRabatt(100m, 0.2m);  // 80€ (20% Rabatt)
```

**WAWI-Empfehlung**: Nett, aber nicht übertreiben

---

### 7. Experimental Attribute & Interceptors 

```csharp
// Experimental Features markieren
[Experimental("WAWI001")]
public class NewArtikelImportFeature
{
    public void ImportFromExcel(string filePath)
    {
        // Experimentelle Implementation
    }
}

// Interceptors (Preview) - Code generation zur Compile-Zeit
// Mehr für Framework-Entwickler, weniger für Business Logic
```

**WAWI-Empfehlung**: Experimental nur für interne Tests, Interceptors erstmal ignorieren

---

## C# 13.0 & 14.0 - Vorschau (Kollegen-Teil - Überblick)

> **Hinweis**: Dein Kollege behandelt diese Versionen. Hier nur ein kurzer Ausblick.

### C# 13.0 (2024) 
```csharp
// Escape Sequences in Raw Strings
string template = """
    SELECT * FROM Artikel 
    WHERE Id = \{{artikelId}}
    """;

// Params Collections
public void LogArtikel(params ReadOnlySpan<Artikel> artikel) { }
```

### C# 14.0 (Preview) 
```csharp
// Field Keyword
public class ArtikelService
{
    private string field;
    public string Property
    {
        get => field;
        set => field = value.Trim(); // 'field' keyword
    }
}

// Partial Properties
public partial class Artikel
{
    public partial string Name { get; set; }
}
```

---

## Migration von C# 7.3: Was ändert sich fundamental?

### Große Paradigmen-Wechsel

#### 1. Von Mutability zu Immutability
```csharp
// C# 7.3 - Mutable by Default
public class ArtikelOld
{
    public string Name { get; set; }
    public decimal Preis { get; set; }
}

// Modern C# - Immutable by Design
public record Artikel(string Name, decimal Preis);
public class BestellungModern
{
    public required string Nummer { get; init; }
    public required decimal Betrag { get; init; }
}
```

#### 2. Von Null zu Nullable Reference Types
```csharp
// C# 7.3 - Null Everywhere
Kunde kunde = Repository.GetKunde(id); // Kann null sein!
string name = kunde.Name; // NullReferenceException möglich

// C# 8+ - Explicit Nullable
Kunde? kunde = Repository.GetKunde(id); // Explizit nullable
string name = kunde?.Name ?? "Unbekannt"; // Null-safe
```

#### 3. Von Statements zu Expressions
```csharp
// C# 7.3 - Verbose Statements
string GetStatus(ArtikelStatus status)
{
    switch (status)
    {
        case ArtikelStatus.Verfuegbar: return "Verfügbar";
        case ArtikelStatus.Ausverkauft: return "Ausverkauft";
        default: return "Unbekannt";
    }
}

// Modern C# - Concise Expressions
string GetStatus(ArtikelStatus status) => status switch
{
    ArtikelStatus.Verfuegbar => "Verfügbar",
    ArtikelStatus.Ausverkauft => "Ausverkauft",
    _ => "Unbekannt"
};
```

---

## Best Practices für die WAWI-Codebase

### Empfohlene Patterns (Priorität für WAWI)

**Sofort einführen (High Impact, Low Risk):**
1. **File-scoped Namespaces**: Überall, spart Einrückung
2. **Global Usings**: Reduziert Duplikation massiv
3. **Collection Expressions**: Einheitliche, lesbare Syntax
4. **Switch Expressions**: Für Status-Mappings, Enum-Handling

**Schrittweise einführen (High Impact, Medium Risk):**
5. **Nullable Reference Types**: Neue Module zuerst, dann Migration
6. **Primary Constructors**: Services, ViewModels, DTOs
7. **Records/Record Structs**: DTOs, Value Objects, API-Models
8. **Init-only Properties**: Immutable Entities

**Optional/Später (Nice to have):**
9. **Async Streams**: Große Datenmengen, Exports
10. **Raw Strings**: SQL-Queries, JSON-Templates

### Mit Vorsicht verwenden

1. **Default Interface Methods**: Nur wenn unbedingt nötig
2. **Komplexe Pattern Matching**: Nicht zu verschachtelt
3. **Lambda Attributes**: Performance-kritische Bereiche

### Vermeiden

1. **Überkomplexe Switch Expressions**: Lesbarkeit geht vor
2. **Ranges in Performance-kritischen Loops**: Array.Copy kann schneller sein
3. **Zu viele Inline-Features**: Code muss wartbar bleiben

---

## Migration Strategy für WAWI

### Phase 1: Quick Wins (C# 10.0 Features) - 2 Wochen
- **File-scoped Namespaces**: Alle neuen Dateien
- **Global Usings**: Pro Assembly einführen
- **Collection Expressions**: Bei Array/List-Initialisierung

### Phase 2: Syntax Modernisierung (C# 8.0 + 12.0) - 1 Monat
- **Switch Expressions**: Enum-Mappings, Status-Checks
- **Primary Constructors**: Services und DTOs
- **Record Structs**: Filter, Parameter, DTOs

### Phase 3: Type Safety (C# 8.0 + 9.0) - 2-3 Monate
- **Nullable Reference Types**: Modul für Modul aktivieren
- **Init-only Properties**: Neue Entities
- **Records**: Value Objects und APIs

### Phase 4: Advanced Features (C# 8.0 + 11.0) - Ongoing
- **Async Streams**: Export-Funktionen
- **Raw Strings**: SQL-Queries, Templates
- **Pattern Matching**: Complex Business Logic

---

## Übungsaufgaben für das Team

### Beginner (alle)
1. **File-scoped Namespace**: Wandle 5 bestehende Dateien um
2. **Global Usings**: Erstelle GlobalUsings.cs für ein Modul
3. **Collection Expressions**: Ersetze `new[] { }` und `new List<> { }` Syntax

### Intermediate (Entwickler mit Erfahrung)
4. **Switch Expression**: Wandle ArtikelStatus-Switch um
5. **Primary Constructor**: Refactore einen Service 
6. **Record Struct**: Erstelle DTO für API-Endpoint

### Advanced (Lead Entwickler)
7. **Nullable Migration**: Aktiviere für ein komplettes Modul
8. **Async Streams**: Implementiere für großen Export
9. **Pattern Matching**: Complex Business Rule mit modernem C#

### Team Challenge
10. **Before/After**: Jeder refactored eine Klasse und präsentiert das Ergebnis

---

## Nächste Schritte
1. **Team-Diskussion**: Welche Features wollen wir priorisieren?
2. **Pilot-Projekt**: Ein Modul als Referenz-Implementation
3. **Coding Guidelines**: StyleCop-Regeln für neue Features
5. **Hands-on Session**: Live-Refactoring einer echten WAWI-Klasse