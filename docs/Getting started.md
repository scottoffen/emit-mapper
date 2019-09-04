Let's start from the classic "Hello, wrold!!!" application :)

{code:c#}
public class A
{
	public string message;
}
public class B
{
	public string message = "Hello, world!!!";
}
{code:c#}

What if we have an instance of class B and we are going to convert it to an similar instance of class A and we don't want to copy fields manually? Solution is rather simple:

{code:c#}
B b = new B();
A a = ObjectMapperManager.DefaultInstance.GetMapper<B, A>().Map(b);
{code:c#}

The main class of the library is ObjectMapperManager. This class performs mappers building (run-time IL code emitting) and contains mappers cache.

{code:c#}
/// <summary>
/// Class for maintaining and generating Mappers.
/// </summary>
public class ObjectMapperManager
{
    public static ObjectMapperManager DefaultInstance
    public ObjectMapperManager();
    
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

In general you don't have to create instance of ObjectMapperManager if you don't need separate mappers cache and you can just use thread-save ObjectMapperManager.DefaultInstance member.

Using class ObjectMapperManager you can create instance of type ObjectsMapper which is responsible for mapping objects of concrete types.

{code:c#}
public class ObjectsMapper<TFrom, TTo>
{
    public TTo Map(TFrom from, TTo to);
    public TTo Map(TFrom from);
}
{code:c#}

The ordinary scenario of using the Emit Objects Library contains 2 steps:

**Step 1**: Create an ObjectsMapper using class ObjectMapperManager:
{code:c#}
var mapper = ObjectMapperManager.DefaultInstance.GetMapper<B, A>();
{code:c#}

**Step 2**: Map objects using mapper created in step 1:
{code:c#}
A a = mapper.Map(new B());
{code:c#}

For example we have two different classes "A" and "B" with similar structure.

{code:c#}
public class A
{
	public enum En
	{
		En1,
		En2,
		En3
	}
	public class AInt
	{
		internal int intern = 13;
		public string str = "AInt";

		public AInt()
		{
			intern = 13;
		}
	}

	private string m_str1 = "A::str1";

	public string str1
	{
		get
		{
			return m_str1;
		}
		set
		{
			m_str1 = value;
		}
	}

	public string str2 = "A::str2";
	public AInt obj;
	public En en = En.En3;

	int[]() m_arr;

	public int[]() arr
	{
		set
		{
			m_arr = value;
		}
		get
		{
			return m_arr;
		}
	}

	public AInt[]() objArr;

	public string str3 = "A::str3";

	public A()
	{
		Console.WriteLine("A::A()");
	}
}

public class B
{
	public enum En
	{
		En1,
		En2,
		En3
	}
	public class BInt
	{
		public string str = "BInt";
	}

	public string str1 = "B::str1";
	public string str2
	{
		get
		{
			return "B::str2";
		}

	}
	public BInt obj = new BInt();
	public En en = En.En2;

	public BInt[]() objArr;

	public int[]() arr
	{
		get
		{
			return new int[]() { 1, 5, 9 };
		}
	}

	public object str3 = null;


	public B()
	{
		Console.WriteLine("B::B()");

		objArr = new BInt[2](2);
		objArr[0](0) = new BInt();
		objArr[0](0).str = "b objArr 1";
		objArr[1](1) = new BInt();
		objArr[1](1).str = "b objArr 2";
	}
}
{code:c#}

Mapping from class "B" to class "A" can be done as below:

{code:c#}
A a = new A();
B b = new B();
ObjectMapperManager.DefaultInstance.GetMapper<B, A>().Map(b, a);

Assert.AreEqual(a.en, A.En.En2);
Assert.AreEqual(a.str1, b.str1);
Assert.AreEqual(a.str2, b.str2);
Assert.AreEqual(a.obj.str, b.obj.str);
Assert.AreEqual(a.obj.intern, 13);
Assert.AreEqual(a.arr.Length, b.arr.Length);
Assert.AreEqual(a.arr[0](0)(0), b.arr[0](0)(0));
Assert.AreEqual(a.arr[1](1)(1), b.arr[1](1)(1));
Assert.AreEqual(a.arr[2](2)(2), b.arr[2](2)(2));
Assert.AreEqual(a.objArr.Length, b.objArr.Length);
Assert.AreEqual(a.objArr[0](0)(0).str, b.objArr[0](0)(0).str);
Assert.AreEqual(a.objArr[1](1)(1).str, b.objArr[1](1)(1).str);
Assert.IsNull(a.str3);
{code:c#}

