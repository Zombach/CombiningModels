# Combining-Models
Предлагаю сделать, что-то типа Join'ов у базы. Ну типа Left то оснавная модель первая и так далее.

# Кодогенерация. 
https://learn.microsoft.com/ru-ru/dotnet/csharp/roslyn-sdk/source-generators-overview
так же я делал проект, типа своего компилятора с генерации и анализом кода. потом ссыль скину.
https://github.com/Zombach/Compiler

Через Mapperly в теории возможно https://mapperly.riok.app/docs/configuration/existing-target/
Automaper примерно такой же способ, но нужно найти реализацию

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

## ИСПОЛЬЗОВАНИЕ:
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

# Ожиданеи
```cs
[Merge]
public class ResponseData
{
    ///
}
```
## Вызов
```cs
var responseData = new ResponseData();
responseData.LoadResource(resource)
```
