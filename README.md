# Tartalomjegyzék

- [Fejlesztéshez használt dolgok](#fejlesztéshez-használt-dolgok)
- [Program build-eléséhez szükséges lépések sorban VS Code terminálban](#program-build-eléséhez-szükséges-lépések-sorban-vs-code-terminálban)
- [C# Backend](#c-backend)
   - [Modellek](#modellek)
   - [Kontrollerek](#kontrollerek)
- [Angular Frontend](#angular-frontend)
   - [Komponensek](#komponensek)

# Überbase

## Fejlesztéshez használt dolgok

- NodeJS v16.16.0 - https://nodejs.org/docs/latest-v16.x/api/index.html

- Angular CLI 14.1.0 - https://v14.angular.io/docs

- Bootstrap 5 - https://getbootstrap.com/docs/5.0/

- JQuery Datatables - https://datatables.net/manual/

- VS Code extensions:
  - C# Dev Kit
  - C#
  - .NET Install Tool

## Program build-eléséhez szükséges lépések sorban VS Code terminálban

```
cd .\ClientApp\
ng build --prog --aot --output-hashing=all
cd ..
dotnet clean .\exceltowebgit.sln
dotnet restore .\exceltowebgit.sln
dotnet publish .\gyakorloprojekt.csproj -c debug -r win-x64 --self-contained true
```

## C# Backend

A backend egy .NET C# program, amely futtat egy webszervert, illetve az adatbázissal való kommunikálást biztosítja. A programban használt minden objektumtípus rendelkezik egy modellel és egy kontrollellel. Ezek a fájlok külön ***models*** és ***controllers*** mappákba vannak rendezve.

### Modellek

A modellek az objektumok osztályait tartalmazzák, az összes változójukkal és metódusaikkal.

```C#
public class Award
{
  public int id { get; set; }
  public string name { get; set; }
}
```

### Kontrollerek

A kontrollerek kezelik a különböző típusok kezelését az adatbázissal való kommunikáció közben.

```C#
[ApiController]
[Route("api/[controller]")]
public class AwardController : ControllerBase
{
    private readonly DatabaseContext cont;

    [HttpGet]
    public IEnumerable<Award> Get()
    {
        return cont.awards;
    }

    [HttpPost]
    public async Task<Award> addAwardConnection([FromBody] Award award)
    {
        cont.Add(award);
        await cont.SaveChangesAsync();
        return award;
    }

    [HttpPost("{id}")]
    public async Task<Award> editAwardConnection(string id, [FromBody] Award award)
    {
        var result = cont.awards.SingleOrDefault(a => a.id == Int32.Parse(id));
        if (result != null)
        {
            result.name = award.name;
            await cont.SaveChangesAsync();
            return award;
        }
        return null;

    }

    [HttpDelete("{id}")]
    public async Task deleteAward(string id)
    {
        var result = cont.awards.SingleOrDefault(a => a.id == Int32.Parse(id));
        if (result != null)
            cont.Remove(result);
        await cont.SaveChangesAsync();
    }

    public AwardController(DatabaseContext cont)
    {
        this.cont = cont;
    }
}
```

## Angular Frontend

### Komponensek

A különböző oldalak komponensként vannak jelen kódszinten. Ezek mappákba vannak rendezve, melyek tartalmaznak egy ***.html***, ***.css*** és ***.ts*** fájlt. 

```
app/
├── awards/
│   ├── awards.component.html
│   ├── awards.component.css
│   └── awards.component.ts
```

**A létrehozott komponensek mappái és leírásai a következők:**

- *app/aktiv* - Alkalmazottak táblázata
- *app/assets* - Adatbázisban tárolt tárgyak táblázata
- *app/awards* - Alkalmazottaknak kioszott jutalmak táblázata
- *app/cards* - Alkalmazottak adatainak kártyás nézete
- *app/collections* - Adatbázisban tárolt tárgyak megjelenítése tulajdonos szerint csoportosítva
- *app/competence* - Kompetencia táblázat (placeholder)
- *app/database-settings* - Adatbázis beállításai (listák, tárgyak típusai, jogosultságok, stb.)
- *app/feedback* - Visszajelzés oldal
- *app/home* - Bejelentkezés oldal
- *app/main* - Az eszköz főoldala
- *app/myassets* - Bejelentkezett alkalmazotthoz tartozó tárgyak
- *app/nav-menu* - Navigációs sáv
- *app/profile-settings* - Személyreszabott beállítások (placeholder)
