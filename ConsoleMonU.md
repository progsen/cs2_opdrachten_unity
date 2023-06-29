## ConsoleMonU

We hebben nu een mooie Console applicatie

Kunnen we deze code ook in unity gebruiken?

## opzet

1. maak een nieuw unity project `ConsoleMonU`
2. maak in het unity project een nieuw `Empty` object, noem deze `Start`
3. maak een script, noem deze even `Program`
4. hang het `Program` script aan de nieuwe `Empty`
5. maak een map in assets aan en noem deze `ConsoleCode`

## Code in unity hangen

1. `kopieer` je consolemon code naar de map `ConsoleCode`
    - neem alleen `Program.cs` NIET mee
2. nu krijg misschien wat errors, fix deze:


- Error CS0246: The type or namespace name 'List<>' could not be found (are you missing a using directive or an assembly reference?)`
kunnen we oplossen door in die `ConsoleMon.cs` file `using System.Collections.Generic;`

- Error	CS0103	The name 'File' does not exist in the current context	Assembly-CSharp	Assets\Scripts\ConsoleCode\ConsoleMonFactory.cs	24	Active
kunnen we oplossen door in die `ConsoleMonFactory.cs` file `using System.IO;`

- Error	CS0122	'JsonSerializer' is inaccessible due to its protection level	Assembly-CSharp	Assets\Scripts\ConsoleCode\ConsoleMonFactory.cs	26	Active
 maak er voor nu dit van:

```
//List<ConsoleMon> templates = JsonSerializer.Deserialize<List<ConsoleMon>>(json);
List<ConsoleMon>  templates = JsonUtility.FromJson<DataFile>(json);
```

3. `Console.Writeline` moeten we overal in de code vervangen door `Debug.Log`
- `TIP` zoek de find and replace in all files shortkey uit voor visual studio, die is heel handig!


## test?

in `Program` pas `Start()` aan zodat het deze code wordt:

```
 void Start()
{
    ConsoleMonFactory consoleMonFactory = new ConsoleMonFactory();
    consoleMonFactory.LoadJson(??????);//zet hier voor nu even het volledige pad naar jou monsterdata.json file
}
```

## Json in unity

De `JsonSerialzer` is een deel van `.net 5-7` maar is anders in `.net framework 3-4`. De `JsonSerializer` was later aangepast en `Unity` gebruikt een ouder framework van `.net`
- zie ook https://docs.unity3d.com/2020.3/Documentation/Manual/JSONSerialization.html

voorbeeld uit de documentatie:
```
[Serializable]
public class MyClass
{
    public int level;
    public float timeElapsed;
    public string playerName;
}

myObject = JsonUtility.FromJson<MyClass>(json);
```

Hoe passen we dit toe?

Onze `class` files hebben niet de [Serializable] attribute die moeten we erbij zetten op:
- ConsoleMon
- Skill

- haal de `{get;set;}` weer weg bij alle class variables, deze utility kan dat niet aan
- zorg ook dat de namen `precies` zo zijn als in de .json datafile (open de file en check!)

## testen

- `Attach` Visual studio aan unity (zorg dat je het project open hebt in visual studio)
- zet een `breakpoint` in de `ConsoleMonFactory` NA het laden van de json

`https://learn.microsoft.com/en-us/visualstudio/debugger/using-breakpoints?view=vs-2022`

- kijk wat er gebeurt als je op `play` drukt in unity

## ArgumentException: JSON must represent an object type?

- wat betekent deze fout?
<details>
<summary>antwoord</summary>

<p>het betekent dat onze json file niet gelezen kan worden</p>

</details>

Dit komt omdat wij een [] aan het begin hebben, dat snapt de `JsonUtility` niet
we moeten er dus nog een `object` omheen zetten, een `wrapper`
voeg deze class toe in een nieuwe file:

```
 [Serializable]
    internal class DataFile
    {
        public List<ConsoleMon> data;
    }
```


pas je `monsterdata.json` aan zet om alles het volgende heen

```
{
  "data":
  [] 
}
```

de `[]` is alle content van de file zoals we die in het `console` project gebruikte, dat stond al in je file

## laden aanpassen

ok file aangepast, wrapper geschreven, wat nu?

- de json file heeft nu een `ander` object wat erin staat
- dus moet de laad code aangepast worden, want er komt iets anders uit

```
        internal List<ConsoleMon> LoadJson(string datafile)
        {
            string json = File.ReadAllText(datafile);
            Console.WriteLine(json);
           
            DataFile templates = JsonUtility.FromJson<DataFile>(json);
            Console.WriteLine(templates.data[0].name);
            return templates.data;
        }
```
 
 - zie je dat we de return van FromJson aangepast hebben? en nu dus het extra object er nog omheen hebben?


## testen

gebruik de debugger, logs en breakpoints om te kijken wat de laad code nu oplevert

- ziet het object er goed uit?
- is het gelijk aan wat je in de .json file ziet?

## tijd voor een ConsoleMon Battle!

- in je `Program` in de `Start` function, voeg je nu de `Arena` toe
- gebruik de code zoals in het console project om 2 consolemon te laten vechten
hint `Fight(ConsoleMon fighterA, ConsoleMon fighterB)`




## Files laden in unity

nu komt het lastige deel
Files laden in unity werkt net als in C#... op jou PC totdat je een `build` maakt en die naar een andere PC overzet

hiervoor kunnen we een nieuwe directory in `assets` maken die Unity kent:
- Resources
- maak deze directory 
- maak in Resources nu een nieuwe directory 'data'
- zet monsterdata.json in die data directory

nu build deze tekstfile mee in de resources van jouw unity project

## laad code

In je Program gaan we wat dingen aanpassen:

1. eerst hebben we een nieuwe using nodig:
zet `using System.Linq;` deze boven in de file bij de usings

2. File laden aanpassen

in `Program` moeten we de function `Start()` aanpassen:
```
 void Start()
{
    ????
    ConsoleMonFactory consoleMonFactory = new ConsoleMonFactory();
    consoleMonFactory.LoadJson("monsterdata.json");//zet hier voor nu even het volledige pad naar jou monsterdata.json file
}
```

zetten we nu bovenaan in de function waar de ???? staan:

```
List<TextAsset> list = Resources.LoadAll<TextAsset>("data").ToList();// haal alle files in data op die een tekst zijn

string json = list.First().text;
Debug.Log(json);
```

3. test of je de json ziet verschijnen in je log
4. pas nu het volgende aan

- `consoleMonFactory.LoadJson("monsterdata.json");` wordt:
- `List<ConsoleMon> templates = consoleMonFactory.LoadJson(json);`


## ConsoleMonFactory aanpassen

- open ConsoleMonFactory
- we passen de function ` LoadJson(string datafile)` aan

- we krijgen nu geen datafile pad mee maar json:
- `internal List<ConsoleMon> LoadJson(string json)`
- haal nu ` string json = File.ReadAllText(datafile);` weg, we hoeven nu hier de file niet meer te lezen

## nu zou het weer moeten werken

zowel in build als in developermode ^^
