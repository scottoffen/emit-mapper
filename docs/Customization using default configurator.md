# Customization using default configurator

[Customization overview](Customization-using-default-configurator#customization_overview)
[Custom converters](Customization-using-default-configurator#Custom_converters)
[Custom converters_for_generics](Customization-using-default-configurator#Custom_converters_for_generics)
[Null substitution](Customization-using-default-configurator#Null_substitution)
[Ignoring members](Customization-using-default-configurator#Ignoring_members)
[Custom constructors](Customization-using-default-configurator#Custom_constructors)
[Shallow and_deep_mapping](Customization-using-default-configurator#Shallow_and_deep_mapping)
[Names matching](Customization-using-default-configurator#Names_matching)
[Post processing](Customization-using-default-configurator#Post_processing)

## {anchor:customization_overview}Customization overview.
Default configurator is the easiest way to customize mapping. Using is rather simple: create instance of type DefaultMapConfig, setup it and pass it to the method GetMapper of ObjectMapperManager. For example:

{code:c#}
var mapper = ObjectMapperManager
	.DefaultInstance
	.GetMapper<Source, Destination>(
		new DefaultMapConfig().NullSubstitution<int?, int>(() => -1)
	);
...
Destination d = mapper.Map(new Source());
{code:c#}

Class "DefaultMapConfig" contains number of configuration methods some of them are generic methods and are parameterized by argument types. This types are the types of fields and properties to which this configuration rule should be applied. For example the configuration method: 
{code:c#}
new DefaultMapConfig().PostProcess<Foo>( 
    (value, state) => Console.WriteLine("Post processing a Foo or it's descendant: " + value.ToString()) 
)
{code:c#}
will be applied to all fields and properties that are instances of class Foo or any it's descendants. 

If you want the configuration method to apply to any type then use "object" as an argument type. For example: 
{code:c#}
new DefaultMapConfig().PostProcess<object>( (value, state) => Console.WriteLine("Post processing: " + value.ToString()) )
{code:c#}
will be applied to any field or property and will write to the console all mapped values.

## {anchor:Custom_converters}Custom converters.

You can setup custom converters for every type of source member and destination member. It is possible using the following method:

{code:c#}
/// <summary>
/// Define custom type converter
/// </summary>
/// <typeparam name="From">Source type</typeparam>
/// <typeparam name="To">Destination type</typeparam>
/// <param name="converter">Function which converts an inctance of the source type to an instance of the destination type</param>
/// <returns></returns>
DefaultMapConfig ConvertUsing<From, To>(Func<From, To> converter)
{code:c#}

Example:

{code:c#}
public class A
{
	public string fld1;
	public string fld2;
}

public class B
{
	public string fld1 = "B::fld1";
	public string fld2 = "B::fld2";
}

[TestMethod](TestMethod)
public void Test_CustomConverter2()
{
	A a = Context.objMan.GetMapper<B, A>(
		new DefaultMapConfig().ConvertUsing<object, string>(v => "converted " + v.ToString())
	).Map(new B());
	Assert.AreEqual("converted B::fld1", a.fld1);
	Assert.AreEqual("converted B::fld2", a.fld2);
}
{code:c#}

## {anchor:Custom_converters_for_generics}Custom converters for generics.

You may want to specify a custom conversion method for one gneric class to another, for example for "HashSet<T>" to "List<T>" where "T" can be anything. Because you don't know concrete argument type you cannot using method "ConvertUsing". Instead of this method "ConvertGeneric" should be used. 

{code:c#}
/// <summary>
/// Define conversion for a generic. It is able to convert not one particular class but all generic family
/// providing a generic converter.
/// </summary>
/// <param name="from">Type of source. Can be also generic class or abstract array.</param>
/// <param name="to">Type of destination. Can be also generic class or abstract array.</param>
/// <param name="converterProvider">Provider for getting detailed information about generic conversion</param>
/// <returns></returns>
public IMappingConfigurator ConvertGeneric(Type from, Type to, ICustomConverterProvider converterProvider);
...
/// <summary>
/// Provider for getting detailed information about generic conversion
/// </summary>
public interface ICustomConverterProvider
{
	/// <summary>
	/// Getting detailed information about generic conversion
	/// </summary>
	/// <param name="from">Type of source. Can be also generic class or abstract array.</param>
	/// <param name="to">Type of destination. Can be also generic class or abstract array.</param>
	/// <param name="mappingConfig">Current mapping configuration</param>
	/// <returns>Detailed description of a generic converter.</returns>
    CustomConverterDescriptor GetCustomConverterDescr(Type from, Type to, MapConfigBaseImpl mappingConfig);
}
...
/// <summary>
/// Detailed description of a generic converter. 
/// </summary>
public class CustomConverterDescriptor
{
	/// <summary>
	/// Type of class which performs conversion. This class can be generic which will be parameterized with types 
	/// returned from "ConverterClassTypeArguments" property.
	/// </summary>
    public Type ConverterImplementation { get; set; }

	/// <summary>
	/// Name of conversion method of class which is specified by "ConverterImplementation" property.
	/// </summary>
    public string ConversionMethodName { get; set; }
	
	/// <summary>
	/// Type arguments for parameterizing generic converter specified by "ConverterImplementation" property.
	/// </summary>
    public Type[]() ConverterClassTypeArguments { get; set; }
}
{code:c#}

In order to setup custom generic conversion you should create a generic class with conversion method:

{code:c#}
class HashSetToListConverter<T>
{
	public List<T> Convert(HashSet<T> from, object state)
	{
		if (from == null)
		{
			return null;
		}
		return from.ToList();
	}
}
{code:c#}

The next step is to specify a custom converter provider for the "HashSetToListConverter<T>". In our case we can directly use class "DefaultCustomConverterProvider" which returns string "Convert" as method name for conversion method and type arguments of the source generic object. So, if we are converting HashSet<int> to List<int> then DefaultCustomConverterProvider returns array of one item: "new Type[](){ typeof(int) }" and this type item from the array will be used as type argument for class "HashSetToListConverter<T>"

Using custom generic converter:

{code:c#}
var mapper = ObjectMapperManager
	.DefaultInstance
	.GetMapper<Source, Destination>(
		new DefaultMapConfig()
		.ConvertGeneric(
			typeof(HashSet<>), 
			typeof(List<>), 
			new DefaultCustomConverterProvider(typeof(HashSetToListConverter<>))
		)
	);
{code:c#}

## {anchor:Null_substitution}Null substitution.

There can be situations when if a source object has null in a field (or property) then  mapper should write some value (not null!) to the destination field. For example we have the following classes:

{code:c#}
class Source
{
	public field int? i;
}
class Destination
{
	public field int i;
}
{code:c#}

If we map instance of class "Source" to instance of class "Destination", Emit Mapper library should decide what to write to Destination::i when Source::i is null. It can be specified using the  following code:

{code:c#}
var mapper = ObjectMapperManager
	.DefaultInstance
	.GetMapper<Source, Destination>(
		new DefaultMapConfig().NullSubstitution<int?, int>(() => -1)
	);
var d = mapper.Map(new Source());
Assert.AreEqual(-1, d.i);
{code:c#}

## {anchor:Ignoring_members}Ignoring members

If you don't want to map some fields or properties use method "IgnoreMembers":

{code:c#}
[TestMethod](TestMethod)
public class B
{
	public string str1 = "B::str1";
}

public class A
{
	public string str1 = "A::str1";
}

public void GeneralTests_Ignore()
{
	var a = ObjectMapperManager.DefaultInstance.GetMapper<B, A>(
		new DefaultMapConfig().IgnoreMembers<B, A>(new[](){"str1"})
	).Map(new B());
	Assert.AreEqual("A::str1", a.str1);
}
{code:c#}

## {anchor:Custom_constructors}Custom constructors

In some cases you don't want to use default constructor for a destination member but instead some custom construction method or non-default constructor.

{code:c#}
public class ConstructBy_Source
{
	public class NestedClass
	{
		public string str = "ConstructBy_Source::str";
	}
	public NestedClass field = new NestedClass();
}

public class ConstructBy_Destination
{
	public class NestedClass
	{
		public string str;
		public int i;
		public NestedClass(int i)
		{
			str = "ConstructBy_Destination::str";
			this.i = i;
		}
	}
	public NestedClass field;
}
[TestMethod](TestMethod)
public void ConstructByTest()
{
	var mapper = ObjectMapperManager
		.DefaultInstance
		.GetMapper<ConstructBy_Source, ConstructBy_Destination>(
			new DefaultMapConfig().ConstructBy <ConstructBy_Destination.NestedClass>(
				() => new ConstructBy_Destination.NestedClass(3)
			)
		);
	var d = mapper.Map(new ConstructBy_Source());
	Assert.AreEqual("ConstructBy_Source::str", d.field.str);
	Assert.AreEqual(3, d.field.i);
}
{code:c#}

## {anchor:Shallow_and_deep_mapping}Shallow and deep mapping

If a field of destination object is the same type as field of source object and this type is complex then the field can be copied by reference (shallow copying) or for the destination field can be constructed separate object and then all nested fields and properties will be copied recursively (deep copying). This behavior can be defined using methods "ShallowMap" or "DeepMap"

{code:c#}
public class A1
{
	public int fld1;
	public string fld2;
}

public class A2
{
	public string[]() fld3;
}

public class A3
{
	public A1 a1 = new A1();
	public A2 a2 = new A2();
}

public class B3
{
	public A1 a1 = new A1();
	public A2 a2 = new A2();
}


b = new B3();
Context.objMan.GetMapper<B3, A3>(new DefaultMapConfig().ShallowMap<A1>().DeepMap<A2>()).Map(b, a);
b.a1.fld1 = 333;
b.a2.fld3 = new string[1](1);

Assert.AreEqual(333, a.a1.fld1); // shallow map
Assert.IsNull(a.a2.fld3); // deep map
{code:c#}

## {anchor:Names_matching}Names matching

If member names of source object don't exactly match to member names of destination object (for example, destination names can have prefix "m_") you can define a function which matches names using method "MatchMembers"

{code:c#}
public class Source
{
	public string field1 = "Source::field1";
	public string field2 = "Source::field2";
	public string field3 = "Source::field3";
}

public class Destination
{
	public string m_field1;
	public string m_field2;
	public string m_field3;
}

[TestMethod](TestMethod)
public void GeneralTests_Example2()
{
	var mapper = ObjectMapperManager.DefaultInstance.GetMapper<Source, Destination>(
		new DefaultMapConfig().MatchMembers((m1, m2) => "m_" + m1 == m2)
	);

	Source src = new Source();
	Destination dst = mapper.Map(src);
	Assert.AreEqual(src.field1, dst.m_field1);
	Assert.AreEqual(src.field2, dst.m_field2);
	Assert.AreEqual(src.field3, dst.m_field3);
}
{code:c#}

The first argument of the matching function is neme of source member, the second argument is name of destination member.

## Post processing

If special actions should be executed after mapping you can use the following method:

{code:c#}
/// <summary>
/// Define postprocessor for specified type
/// </summary>
/// <typeparam name="T">Objects of this type and all it's descendants will be postprocessed</typeparam>
/// <param name="postProcessor"></param>
/// <returns></returns>
public IMappingConfigurator PostProcess<T>(ValuesPostProcessor<T> postProcessor)
{code:c#}

Example:

{code:c#}
public class Destination
{
	public List<int> list;
}


public class Source
{
	public int[]() list;
}

var mapper = ObjectMapperManager
	.DefaultInstance
	.GetMapper<Source, Destination>(
		new DefaultMapConfig()
		.PostProcess<Destination>( (destination, state) => { destination.list.Sort(); return destination;} )
	);
{code:c#}