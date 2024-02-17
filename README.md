# Combining-Models
Объединение двух моделей по имени полей и их типам.
Самый быстрый способ на данный момент, это руками пройтись по модели.
Но для каждого свойства необходимо писать кучу кода руками.
Самая главная проблема в том, что если модель сменится, то код придется переписывать, либо редактировать. 
Есть предложение использую кодогенерацию, реализовать через атрибуты возможности, слияния двух моделей в одну.
Атрибуты будут из разряда Join's для sql.
[LeftMerge] - левая модель берется за основу, и в неё вливаются данные из правой модели, но не заменяются те данные которые уже имеются.
[RightMerge] - правая модель берется за основу, и в неё вливаются данные из левой модели, но не заменяются те данные которые уже имеются.
[Merge] - Делается слияние двух моделей. Из правой в левую, заменяю данные если они имеются.
Код должен генерироваться, на подобии метода "руками"

Вот информация по генерации кода.
# Кодогенерация. 
https://learn.microsoft.com/ru-ru/dotnet/csharp/roslyn-sdk/source-generators-overview
так же я делал проект, типа своего компилятора с генерации и анализом кода.
https://github.com/Zombach/Compiler

Можно посмотреть тут. Сам маппер реализован через кодогенерацию.
Через Mapperly в теории возможно https://mapperly.riok.app/docs/configuration/existing-target/
Automaper примерно такой же способ, но нужно найти реализацию

Немного жути. но будет работать, трабл в том, что рефлексия заведомо тяжелая.
# Отражение примеры
НЕПРОВЕРЕННЫЙ, но использующий Reflection.Emit API, что-то вроде этого должно сработать:
```cs
public Type MergeTypes(params Type[] types)
{
    AppDomain domain = AppDomain.CurrentDomain;
    AssemblyBuilder builder = 
        domain.DefineDynamicAssembly(new AssemblyName("CombinedAssembly"),
        AssemblyBuilderAccess.RunAndSave);
    ModuleBuilder moduleBuilder = builder.DefineDynamicModule("DynamicModule");
    TypeBuilder typeBuilder = moduleBuilder.DefineType("CombinedType");
    foreach (var type in types)
    {
        var props = GetProperties(type);
        foreach (var prop in props)
        {
            typeBuilder.DefineField(prop.Key, prop.Value, FieldAttributes.Public);
        }
    }

    return typeBuilder.CreateType();
}

private Dictionary<string, Type> GetProperties(Type type)
{
    return type.GetProperties().ToDictionary(p => p.Name, p => p.PropertyType);
}
```

### ИСПОЛЬЗОВАНИЕ:
```cs
Type combinedType = MergeTypes(typeof(Foo), typeof(Bar));
```


# Руками
```cs
public static void UpdateResponseData(this ResponseData source, ResponseData resource)
{
    if (resource.ExchangeId is not null)
    {
        source.ExchangeId = resource.ExchangeId;
    }
    if (resource.MarketName is not null)
    {
        source.MarketName = resource.MarketName;
    }
    if (resource.PercentChange is not null)
    {
        source.PercentChange = resource.PercentChange;
    }
    if (resource.PreviousCloseDate is not null)
    {
        source.PreviousCloseDate = resource.PreviousCloseDate;
    }
    if (resource.PreviousClosePrice is not null)
    {
        source.PreviousClosePrice = resource.PreviousClosePrice;
    }
    if (resource.BidYield is not null)
    {
        source.BidYield = resource.BidYield;
    }
}
```

# Ожидание
```cs
[Merge]
public partial class DataMerge
{
    [LeftMerge]
    public partial ResponseData LeftMerge(ResponseData right);
}
```
### Вызов
```cs
var responseData = new ResponseData();
responseData.LeftMerge(resource)
```
