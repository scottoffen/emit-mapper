# API description

The main class of the library is ObjectsMapperManager. This class performs mappers building (run-time IL code emitting) and contains mappers cache.

{code:c#}
/// <summary>
/// Class for maintaining and generating Mappers.
/// </summary>
public class ObjectsMapperManager
{
    public static ObjectsMapperManager DefaultInstance
    public ObjectsMapperManager();
    
    /// <summary>
    /// Returns a Mapper instance for specified types.
    /// </summary>
    /// <typeparam name="TFrom">Type of source object</typeparam>
    /// <typeparam name="TTo">Type of destination object</typeparam>
    /// <returns></returns>
    public ObjectsMapper<TFrom, TTo> GetMapper<TFrom, TTo>();

    /// <summary>
    /// Returns a Mapper instance for specified types.
    /// </summary>
    /// <typeparam name="TFrom">Type of source object</typeparam>
    /// <typeparam name="TTo">Type of destination object</typeparam>
    /// <param name="ShallowCopy">True, if reference members of same type should be copied by
    /// reference (shallow copy, without creating new instance for destination object)</param>
    /// <returns></returns>
    public ObjectsMapper<TFrom, TTo> GetMapper<TFrom, TTo>(bool ShallowCopy);

    /// <summary>
    /// Returns a Mapper instance for specified types.
    /// </summary>
    /// <typeparam name="TFrom">Type of source object</typeparam>
    /// <typeparam name="TTo">Type of destination object</typeparam>
    /// <param name="ShallowCopy">True, if reference members of same type should be copied by
    /// reference (shallow copy, without creating new instance for destination object)</param>
    /// <param name="mappingConfigurator">Delegate which configures mapping.</param>
    /// <param name="mapperName">Name of mapper. For two mappers with identical type parameters
    /// but different names will be generated two different classes.</param>
    /// <returns></returns>
    public ObjectsMapper<TFrom, TTo> GetMapper<TFrom, TTo>(
        bool ShallowCopy,
        MappingConfigurator mappingConfigurator,
        string mapperName);

    /// <summary>
    /// Returns a mapper implementation instance for specified types.
    /// </summary>
    /// <param name="from">Type of source object</param>
    /// <param name="to">Type of destination object</param>
    /// <param name="ShallowCopy">True, if reference members of same type should
    /// be copied by reference (shallow copy, without creating new instance for destination object)</param>
    /// <param name="mappingConfigurator">Delegate which configures mapping.</param>
    /// <returns></returns>
    public ObjectsMapperBaseImpl GetMapperImpl(
        Type from,
        Type to,
        bool ShallowCopy,
        MappingConfigurator mappingConfigurator);

    /// <summary>
    /// Returns a mapper implementation instance for specified types.
    /// </summary>
    /// <param name="from">Type of source object</param>
    /// <param name="to">Type of destination object</param>
    /// <param name="ShallowCopy">True, if reference members of same type should
    /// be copied by reference (shallow copy, without creating new instance for destination object)</param>
    /// <param name="mappingConfigurator">Delegate which configures mapping.</param>
    /// <returns></returns>
    public ObjectsMapperBaseImpl GetMapperImpl(
        Type from,
        Type to,
        bool ShallowCopy,
        MappingConfigurator mappingConfigurator,
        string mapperName);
}
{code:c#}

In general you don't have to create instance of ObjectsMapperManager if you don't need separate mappers cache and you can just use thread-save ObjectsMapperManager.DefaultInstance member.

Using class ObjectsMapperManager you can create instance of type ObjectsMapper which is responsible for mapping objects of concrete types.

{code:c#}
public class ObjectsMapper<TFrom, TTo>
{
    public TTo Map(TFrom from, TTo to);
    public TTo Map(TFrom from);
}
{code:c#}

The ordinary scenario of using the Emit Objects Library contains of 2 steps:

**Step 1**: Create an ObjectsMapper using class ObjectsMapperManager:
{code:c#}
var mapper = ObjectsMapperManager.DefaultInstance.GetMapper<B, A>();
{code:c#}

**Step 2**: Map objects using mapper created on step 1:
{code:c#}
A a = mapper.Map(new B());
{code:c#}
