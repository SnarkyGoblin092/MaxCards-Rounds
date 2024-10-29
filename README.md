# Tartalomjegyzék

- [Fejlesztéshez használt dolgok](#fejlesztéshez-használt-dolgok)
- [Program build-eléséhez szükséges lépések sorban VS Code terminálban](#program-build-eléséhez-szükséges-lépések-sorban-vs-code-terminálban)
- [C# Backend](#c-backend)
   - [Backend Modellek](#backend-modellek)
   - [Kontrollerek](#kontrollerek)
- [Angular Frontend](#angular-frontend)
   - [Komponensek](#komponensek)
   - [Frontend Modellek](#frontend-modellek)
   - [Érdekességek](#érdekességek)
      - [Modals](#modals)
      - [Jogok](#jogok)
         - [Jogok deklarálása](#jogok-deklarálása)
         - [Jogok lekérése](#jogok-lekérése)
- [Suffix](#suffix)

# Überbase

## Fejlesztéshez használt dolgok

- NodeJS v16.16.0 - https://nodejs.org/docs/latest-v16.x/api/index.html

- Angular CLI 14.1.0 - https://v14.angular.io/docs

- Bootstrap 5 - https://getbootstrap.com/docs/5.0/

- JQuery Datatables - https://datatables.net/manual/

- ASP.NET - https://learn.microsoft.com/en-us/aspnet/core/?view=aspnetcore-8.0

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

Az .NET keretrendszer funkcióinak használatához a fent linkelt dokumentációban találsz segítséget.

### Backend Modellek

A modellek az objektumok osztályait tartalmazzák, az összes változójukkal és metódusaikkal.

Fájl: `Models/Award.cs`
```C#
public class Award
{
  public int id { get; set; }
  public string name { get; set; }
}
```

### Kontrollerek

A kontrollerek kezelik a különböző típusok kezelését az adatbázissal való kommunikáció közben.

Fájl: `Controllers/AwardController.cs`
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

A komponensek hozzáadásához és az *Angular* specifikus funkciók használatához használd a fent linkelt **dokumentációt**.

### Komponensek

A különböző oldalak komponensként vannak jelen kódszinten. Ezek mappákba vannak rendezve, melyek tartalmaznak egy ***.html***, ***.css*** és ***.ts*** fájlt. 

```
ClientApp/
├── src/
│   ├── app/
│   │   ├── awards/
│   │   │   ├── awards.component.html
│   │   │   ├── awards.component.css
│   │   │   └── awards.component.ts
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

### Frontend Modellek

A C# modelleknek megfelelően, itt is létre kell hoznunk a programban használt objektumok osztályait.

Fájl: `ClientApp/src/app/award.model.ts`
```typescript
export class Award {
    public id?: number;
    public name: string;

    constructor(name?: string, id?: number) {

        this.id = id;
        this.name = name;
    }
}
```

## Érdekességek

### Modals

Több oldalon is felugró ablakokban jelenik meg az információ. Ezek küzül vannak amelyeknek a gombja a navigációs sávon található. Az ilyen modal-ok kódja a navigációs sáv *.html* fájljában van, hiába tartozik valamelyik oldalhoz. Az összes olyan modal melynek a megjelenítéshez használt vezérlője nem a navigációs sávon található, a megjelenített oldal *.html* fájljában található. Erre azért volt szükség, hogy a gombok linkjei megfelelően működjenek.

### Jogok

#### Jogok deklarálása

Az oldalon a láthatóságot és a különböző műveleteket jogok védig, melyek egy külön fájlban vannak létrehozva. Minden joghoz tarozik egy **static readonly** kulcs, amivel könnyebben lekérhetjük betöltésekor a felhasználó az adott oldalhoz releváns jogait.

Fájl: `ClientApp/src/app/permissiongroup.model.ts`
```typescript
static readonly ACCESS_EMPLOYEES = 2;
```

Ugyanakkor egy rövid leírás is tartozik a jogokhoz, mely a jogok kiosztásakor jelenik meg.

```typescript
public static readonly permissions = [
   "Access employees page"
]
```

A jogokat az egyszerűbb megjelenítés érdekében külön csoportokba rakjuk és a megjelenítés sorrendjében hozzáadjuk az egyes csoportokhoz az adott jog ID-jét.

```typescript
public static readonly groupedPermissions = [
   { "name": "Super Users", "permissions": [0, 1, 11, 12, 13, 14, 15, 17, 19] },
   { "name": "Employees", "permissions": [2, 3, 4, 5, 6, 7, 8, 9, 10, 16, 18] },
   { "name": "StoreIt", "permissions": [20, 21, 22, 23, 24, 25, 26] }
]
```

#### Jogok lekérése

A jogok lekéréséhez a *NavMenuComponent* rendelkezik egy `checkPermission()` metódussal. Ezt használhatjuk bármelyik oldalon ha ellenőrizni akarjuk, hogy egy adott tevékenységhez rendelkezik-e megfelelő jogokkal a felhasználó.

A *NavMenuComponent*-en belül a következő módon tudunk ellenőrizni jogokat:

```typescript
// Permission.ACCESS_EMPLOYEES az előzőleg említett jog azonosítójára mutat
this.can_access_employees = this.checkPermission(Permission.ACCESS_EMPLOYEES);
```

Más komponensekből ennek módosításával tudunk ellenőrizni jogokat:

```typescript
this.can_access_employees = NavMenuComponent.instance.checkPermission(Permission.ACCESS_EMPLOYEES);
```

Ha az előző megoldást akarjuk használni, akkor a következő kódrésszel egészíthetjük ki a komponenseket, melynek hozzáadása után úgy használhatjuk a függvényt, mintha a *NavMenuComponent*-ben dolgoznánk.

```typescript
public checkPermission(key: number);
public checkPermission(key: string)
public checkPermission(...args: any) {
   return NavMenuComponent.instance.checkPermission(args[0]);
}
```

Ezek után a `can_access_employees` változó **True** vagy **False** értékével tudunk megjeleníteni vagy eltűntetni dolgokat.

## Suffix

Fejlesztés előtt/során alaposan tanulmányozd a különböző kódrészek működését a megértésükhöz, illetve ötleteket szerezz a felhasználásukhoz/bővítésükhöz. Néhány megoldás nem a legjobb, illetve nem a legoptimálisabb teljesítményre lett fejlesztve, így javasolt a problémák újbóli átgondolása és megoldása, refaktorálása.
