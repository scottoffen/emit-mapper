# Overview.

There are lot of situations when you have to perform some action with each member (field or property) of an object. The classic case is DTO objects. Let's imagine that you have a Data Access Layer. Inside this layer you use some tool to access to database. It can be for example LINQ to SQL, or Entities Framework or some other ORM tool. These tools can expose classes that represent database tables or in more general case database entities. The problem is that these classes contain a lot of technical details about your ORM tool. It can be specific base class, attributes, properties, fields and so on. It is not good idea to expose these classes to outside your Data Access Library because sometime it is lack of encapsulation (what if you decide to change ORM tool?) and sometimes it is impossible at all. The acceptably solution is to use DTO (**D**ata **T**ransition **O**bjects).

For example we have the following table:
{{CREATE TABLE [dbo](dbo).[Customers](Customers)(
	[CustomerID](CustomerID) uniqueidentifier NOT NULL PRIMARY KEY,
	[ContactName](ContactName) [nvarchar](nvarchar)(30) NULL,
	[Address](Address) [nvarchar](nvarchar)(60) NULL,
	[Phone](Phone) [nvarchar](nvarchar)(24) NULL
)}}

Inside our Data Access Layer we use LINQ to SQL to access this table. The Visual Studio Designer generated the following class which represents customers:

{code:c#}
[Table(Name="dbo.Customers")](Table(Name=_dbo.Customers_))
public partial class Customer : INotifyPropertyChanging, INotifyPropertyChanged
{
	private static PropertyChangingEventArgs emptyChangingEventArgs = new PropertyChangingEventArgs(String.Empty);
	private string _CustomerID;
	private string _ContactName;
	private string _Address;
	private string _Phone;
	
#region Extensibility Method Definitions
...
#endregion
	
	public Customer()
	{
		OnCreated();
	}
	
	[Column(Storage="_CustomerID", DbType="NChar(5) NOT NULL", CanBeNull=false, IsPrimaryKey=true)](Column(Storage=__CustomerID_,-DbType=_NChar(5)-NOT-NULL_,-CanBeNull=false,-IsPrimaryKey=true))
	public string CustomerID
	{
		get
		{
			return this._CustomerID;
		}
		set
		{
			if ((this._CustomerID != value))
			{
				this.OnCustomerIDChanging(value);
				this.SendPropertyChanging();
				this._CustomerID = value;
				this.SendPropertyChanged("CustomerID");
				this.OnCustomerIDChanged();
			}
		}
	}
	
	[Column(Storage="_ContactName", DbType="NVarChar(30)")](Column(Storage=__ContactName_,-DbType=_NVarChar(30)_))
	public string ContactName
	{
		get
		{
			return this._ContactName;
		}
		set
		{
			if ((this._ContactName != value))
			{
				this.OnContactNameChanging(value);
				this.SendPropertyChanging();
				this._ContactName = value;
				this.SendPropertyChanged("ContactName");
				this.OnContactNameChanged();
			}
		}
	}
	...
	public event PropertyChangingEventHandler PropertyChanging;
	public event PropertyChangedEventHandler PropertyChanged;

	protected virtual void SendPropertyChanging()
	{
		if ((this.PropertyChanging != null))
		{
			this.PropertyChanging(this, emptyChangingEventArgs);
		}
	}
	
	protected virtual void SendPropertyChanged(String propertyName)
	{
		if ((this.PropertyChanged != null))
		{
			this.PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
		}
	}
}
{code:c#}

So, as you can see here is lot of database-specific information which should not be visible outside of Data Access Layer. In order to achieve this we create our own Customer class which looks like:

{code:c#}
public class DTOCustomer
{
    public Guid CustomerId { get; set; }
    public string ContactName { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }
}
{code:c#}

Now a data access method can look like:

{code:c#}
public DTOCustomer GetCustomer(Guid customerId)
{
    using (var dc = new DataContext())
    {
        var customer = dc.Customers.Where(c => c.CustomerID == customerId).Single();
        return new DTOCustomer // <-- this is our DTO class which is visible from outside
        {
            CustomerID = customer.CustomerID,
            ContactName = customer.ContactName,
            Address = customer.Address,
            Phone = customer.Phone
        };
    }
}
{code:c#}

This solution has problems:

* It is boring. We have to manually copy properties.
* It is buggy. If we add a new field to the Customer table and update the LINQ data context, we will get a bug because this new field will not be copied to the DTO object.

That problem can be solved using the Emit Mapper library:

{code:c#}
public DTOCustomer GetCustomer(Guid customerId)
{
    using (var dc = new DataContext())
    {
        var customer = dc.Customers.Where(c => c.CustomerID == customerId).Single();
        return ObjectMapperManager.DefaultInstance.GetMapper<Customer, DTOCustomer>().Map(customer);
    }
}
{code:c#}

The Emit Mapper is rather powerful customisable tool. You can fully customise source (items, naming, getters), type conversion, destination (items, naming, setters). The Emit Mapper library doesn't know anything about System.Data namespace, but it is so customisable that you can use it to map automatically abstract DbDatareader to your classes using mapping strategy that you need or you like. If you like you can use some attribute to specify name of field from DbDatareader, or you can use the same name of field from DTO object, or you can implement any logic you like.

Copieng is not only one action that can be applied to object members during mapping. For example it is possible to create using Emit Mapper the objects change tracker:
{code:c#}
public class A
{
    public string f1 = "f1";
    public int f2 = 2;
    public bool f3 = true;
}
...
var tracker = new ObjectsChangeTracker();
var a = new A();
tracker.RegisterObject(a);
a.f2 = 3;
string[]() changes = tracker.GetChanges(a);
Assert.IsTrue( changes[0](0) == "f2");
{code:c#}

Another usefull tool that can be built using Emit Mapper is automatic SQL operators builder for INSERT and UPDATE commands.

{code:c#}
using (DbConnection connection = CreateConnection())
{
    DBTools.InsertObject(
        connection,
        new Customer
        {    
            CustomerID = Guid.NewGuid(),
            ContactName = "John Smith",
            Address = "Some Street 15",
            Phone = "1234567890"
        },
        "Customers",
        DbSettings.MSSQL
    );
}
{code:c#}

Essential point of the source code above is not that it is another one "Super ORM". Essential point is that you can yourself develop very effective and light database access utilities in respect to your project requirements. Sometimes it is more preferable than using some huge generic ORM.