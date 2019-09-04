The most important goal of EmitMapper library is to provide the best performance. If you don't need perfomance you can just easily use the System.Reflection library for automatic mapping. But if you need perfomance you have to use handwritten mappers or some mapping library which leverages the System.Emit library (for example EmitMapper or AutoMapper). Unfortunately using the Emit library don't guarantee good perfomance. 

I used the following classes for benchmark:

{code:c#}
public class BenchSource
{
	public class Int1
	{
		public string str1 = "1";
		public string str2 = null;
		public int i = 10;
	}

	public class Int2
	{
		public Int1 i1 = new Int1();
		public Int1 i2 = new Int1();
		public Int1 i3 = new Int1();
		public Int1 i4 = new Int1();
		public Int1 i5 = new Int1();
		public Int1 i6 = new Int1();
		public Int1 i7 = new Int1();
	}

	public Int2 i1 = new Int2();
	public Int2 i2 = new Int2();
	public Int2 i3 = new Int2();
	public Int2 i4 = new Int2();
	public Int2 i5 = new Int2();
	public Int2 i6 = new Int2();
	public Int2 i7 = new Int2();
	public Int2 i8 = new Int2();

	public int n2;
	public long n3;
	public byte n4;
	public short n5;
	public uint n6;
	public int n7;
	public int n8;
	public int n9;

	public string s1 = "1";
	public string s2 = "2";
	public string s3 = "3";
	public string s4 = "4";
	public string s5 = "5";
	public string s6 = "6";
	public string s7 = "7";

}

public class BenchDestination
{
	public class Int1
	{
		public string str1;
		public string str2;
		public int i;
	}

	public class Int2
	{
		public Int1 i1;
		public Int1 i2;
		public Int1 i3;
		public Int1 i4;
		public Int1 i5;
		public Int1 i6;
		public Int1 i7;
	}

	public Int2 i1 { get; set; }
	public Int2 i2 { get; set; }
	public Int2 i3 { get; set; }
	public Int2 i4 { get; set; }
	public Int2 i5 { get; set; }
	public Int2 i6 { get; set; }
	public Int2 i7 { get; set; }
	public Int2 i8 { get; set; }

	public long n2 = 2;
	public long n3 = 3;
	public long n4 = 4;
	public long n5 = 5;
	public long n6 = 6;
	public long n7 = 7;
	public long n8 = 8;
	public long n9 = 9;

	public string s1;
	public string s2;
	public string s3;
	public string s4;
	public string s5;
	public string s6;
	public string s7;
}
{code:c#}

It is rather huge class with number of nested classes, structures and fields. 

Handwritten mapper for these classes:

{code:c#}
static BenchDestination.Int1 Map(BenchSource.Int1 s, BenchDestination.Int1 d)
{
	if (s == null)
	{
		return null;
	}
	if (d == null)
	{
		d = new BenchDestination.Int1();
	}
	d.i = s.i;
	d.str1 = s.str1;
	d.str2 = s.str2;
	return d;
}
static BenchDestination.Int2 Map(BenchSource.Int2 s, BenchDestination.Int2 d)
{
	if (s == null)
	{
		return null;
	}

	if (d == null)
	{
		d = new BenchDestination.Int2();
	}
	d.i1 = Map(s.i1, d.i1);
	d.i2 = Map(s.i2, d.i2);
	d.i3 = Map(s.i3, d.i3);
	d.i4 = Map(s.i4, d.i4);
	d.i5 = Map(s.i5, d.i5);
	d.i6 = Map(s.i6, d.i6);
	d.i7 = Map(s.i7, d.i7);

	return d;
}
static BenchDestination Map(BenchSource s, BenchDestination d)
{
	if (s == null)
	{
		return null;
	}
	if (d == null)
	{
		d = new BenchDestination();
	}
	d.i1 = Map(s.i1, d.i1);
	d.i2 = Map(s.i2, d.i2);
	d.i3 = Map(s.i3, d.i3);
	d.i4 = Map(s.i4, d.i4);
	d.i5 = Map(s.i5, d.i5);
	d.i6 = Map(s.i6, d.i6);
	d.i7 = Map(s.i7, d.i7);
	d.i8 = Map(s.i8, d.i8);

	d.n2 = s.n2;
	d.n3 = s.n3;
	d.n4 = s.n4;
	d.n5 = s.n5;
	d.n6 = s.n6;
	d.n7 = s.n7;
	d.n8 = s.n8;
	d.n9 = s.n9;

	d.s1 = s.s1;
	d.s2 = s.s2;
	d.s3 = s.s3;
	d.s4 = s.s4;
	d.s5 = s.s5;
	d.s6 = s.s6;
	d.s7 = s.s7;

	return d;
}
{code:c#}

Results of the benchmark:

{{Handwritten Mapper: 475 milliseconds
Emit Mapper: 469 milliseconds
Auto Mapper: 205256 milliseconds}}

The results are very interesting:
# AutoMapper is 437 times slower than EmitMapper. Probably it is not the best solution for time-critical tasks.
# Even handwritten code is a bit slower than EmitMapper, but why? The answer is that in respect to the programming practices i did not use the "Copy/Paste" technique and made submappers as separate methods:

{code:c#}
static BenchDestination.Int1 Map(BenchSource.Int1 s, BenchDestination.Int1 d)
{
	if (s == null)
	{
		return null;
	}
	if (d == null)
	{
		d = new BenchDestination.Int1();
	}
	d.i = s.i;
	d.str1 = s.str1;
	d.str2 = s.str2;
	return d;
}
{code:c#}

And then just used these mappers:

{code:c#}
d.i1 = Map(s.i1, d.i1);
d.i2 = Map(s.i2, d.i2);
d.i3 = Map(s.i3, d.i3);
d.i4 = Map(s.i4, d.i4);
d.i5 = Map(s.i5, d.i5);
...
{code:c#}

But EmitMapper as a bad programmer uses "Copy\Paste" and deals without additional calls. So, in this test EmitMapper is even faster than usual handwritten code.