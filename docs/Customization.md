Customization can be performed using methods of class ObjectsMapperManager passing as parameter instance of MappingConfigurator delegate. The signature of this delegate is:

{code:c#}
/// <summary>
/// Delegate which is used for customiza mapping.
/// </summary>
/// <param name="from">Source type</param>
/// <param name="to">Destination type</param>
/// <param name="mappingSettings">Mapping settings with fine-grained mapping configuration</param>
public delegate void MappingConfigurator(Type from, Type to, MappingSettings mappingSettings);
{code:c#}

MappingSettings is a class where the whole mapping configuration for concrete source and destination types can be specified. 

{code:c#}
public class MappingSettings
{
	public MappingItem[]() MappingItems { get; set; }
	public TargetConstructor CreateTargetInstance { get; set; }
	public MappingSettings RemoveItem(string itemName);
}
{code:c#}

Here you can specify delegate (CreateTargetInstance) which constructs destination object. Every atomic action which is applied to members of mapping objects is described in MappingItems. 

{code:c#}
public class MappingItem
{
	/// <summary>
	/// Delegate for getting value from source
	/// </summary>
	public MapperValueGetter MapperValueGetter { get; set; }
	
	/// <summary>
	/// Delegate for converting value
	/// </summary>
	public MapperValueSetter MapperValueSetter { get; set; }
	
	/// <summary>
	/// Delegate for setting value to destination
	/// </summary>
	public MapperValueConverter MapperValueConverter { get; set; }
	
	public MemberInfo SourceMember { get; }
	public MemberInfo DestinationMember { get; }
	
	public Type SourceMemberType { get; }
	public Type DestinationMemberType { get; }
	public MappingItem();
	public MappingItem(MemberInfo sourceMember, MemberInfo destinationMember);
}
{code:c#}

Here you can specify delegates for: 
1) getting value from source.
2) converting value.
3) writing value to destination.

Example 1:

{code:c#}
public class Source
{
	public string field1;
	public string field2;
	public string field3;
}

public class Destination
{
	public string field1;
	public string field2;
	public string field3;
}

[TestMethod](TestMethod)
public void GeneralTests_Example1()
{
	var mapper = ObjectsMapperManager.DefaultInstance.GetMapper<Source, Destination>(
		false,
		(from, to, settings) =>
		{
			foreach (var mappingItem in settings.MappingItems)
			{
				mappingItem.MapperValueGetter =
					(sourceObject, mi) =>
					{
						Console.WriteLine("Getting member '" + mi.SourceMember.Name + "'");
						return mi.SourceMember.Name;
					};

				mappingItem.MapperValueConverter =
					(mi, value) =>
					{
						Console.WriteLine("Converting value '" + value.ToString() + "'");
						return value;
					};

				mappingItem.MapperValueSetter =
					(destination, mi, value) =>
					{
						Console.WriteLine("Setting value '" + value.ToString() + "'");
					};
			}
		},
		"Example1"
	);

	Destination dest = mapper.Map(new Source());
}
{code:c#}

Output:

{{
Getting member 'field1'
Converting value 'field1'
Setting value 'field1'
Getting member 'field2'
Converting value 'field2'
Setting value 'field2'
Getting member 'field3'
Converting value 'field3'
Setting value 'field3'
}}

In the following example class members have different names. In the destination class each member has "m_" prefix, but the source class doesn't. We can automatically setup mapping strategy in respect to this naming as showed below:

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
	var mapper = ObjectsMapperManager.DefaultInstance.GetMapper<Source, Destination>(
		false, // ShallowCopy
		(from, to, settings) => // MappingConfigurator
		{
			settings.MappingItems = settings
				.MappingItems
				.Where( mi => mi.SourceMember != null)
				.Select(
					mi =>
						new MappingItem(
							mi.SourceMember,
							
							settings.MappingItems
								.Where(smi => smi.DestinationMember != null && smi.DestinationMember.Name == "m_" + mi.SourceMember.Name)
								.Single()
								.DestinationMember
						)
					)
				.ToArray();
		},
		"Example2" // Mapper name
	);

	Source src = new Source();
	Destination dst = mapper.Map(new Source());
	Assert.AreEqual(src.field1, dst.m_field1);
	Assert.AreEqual(src.field2, dst.m_field2);
	Assert.AreEqual(src.field3, dst.m_field3);
}
{code:c#}

Of course you don't have to write this lot of code each time you need mapping. It is reasonable to create class with mapping configurators.

{code:c#}
public static class MappingConfigurators
{
	public static void M_Naming(Type from, Type to, MappingSettings settings)
	{
		settings.MappingItems = settings
			.MappingItems
			.Where(mi => mi.SourceMember != null)
			.Select(
				mi =>
					new MappingItem(
						mi.SourceMember,
						settings.MappingItems
							.Where(smi => smi.DestinationMember != null && smi.DestinationMember.Name == "m_" + mi.SourceMember.Name)
							.Single()
							.DestinationMember
					)
				)
			.ToArray();
	}
}

[TestMethod](TestMethod)
public void GeneralTests_Example3()
{
	var mapper = ObjectsMapperManager.DefaultInstance.GetMapper<Source, Destination>(
		false, // ShallowCopy
		MappingConfigurators.M_Naming, // MappingConfigurator
		"Example3" // Mapper name
	);

	Source src = new Source();
	Destination dst = mapper.Map(src);
	Assert.AreEqual(src.field1, dst.m_field1);
	Assert.AreEqual(src.field2, dst.m_field2);
	Assert.AreEqual(src.field3, dst.m_field3);
}
{code:c#}
