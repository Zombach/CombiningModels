# Combining-Models
Предлагаю сделать, что-то типа Join'ов у базы. Ну типа Left то оснавная модель первая и так далее.

кодогенерация. 
https://learn.microsoft.com/ru-ru/dotnet/csharp/roslyn-sdk/source-generators-overview
так же я делал проект, типа своего компилятора с генерации и анализом кода. потом ссыль скину.
https://github.com/Zombach/Compiler

Через Mapperly в теории возможно https://mapperly.riok.app/docs/configuration/existing-target/
Automaper примерно такой же способ, но можно найти реализацию

Отрожение примеры
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

ИСПОЛЬЗОВАНИЕ:
```cs
Type combinedType = MergeTypes(typeof(Foo), typeof(Bar));
```


Руками
```cs
  public void UpdateResponseData(ResponseData responseData)
    {
        if (responseData.ExchangeId is not null)
        {
            ResponseData.ExchangeId = responseData.ExchangeId;
        }
        if (responseData.MarketName is not null)
        {
            ResponseData.MarketName = responseData.MarketName;
        }
        if (responseData.PercentChange is not null)
        {
            ResponseData.PercentChange = responseData.PercentChange;
        }
        if (responseData.PreviousCloseDate is not null)
        {
            ResponseData.PreviousCloseDate = responseData.PreviousCloseDate;
        }
        if (responseData.PreviousClosePrice is not null)
        {
            ResponseData.PreviousClosePrice = responseData.PreviousClosePrice;
        }
        if (responseData.BidYield is not null)
        {
            ResponseData.BidYield = responseData.BidYield;
        }
    }
```
