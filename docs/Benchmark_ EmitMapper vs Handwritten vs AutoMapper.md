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
	public struct Int1
	{
		public string str1;
		public string str2;
		public int i;
	}

	public struct Int2
	{
		public Int1 i1;
		public Int1 i2;
		public Int1 i3;
		public Int1 i4;
		public Int1 i5;
		public Int1 i6;
		public Int1 i7;
	}

	public Int2 i1;
	public Int2 i2;
	public Int2 i3;
	public Int2 i4;
	public Int2 i5;
	public Int2 i6;
	public Int2 i7;
	public Int2 i8;

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
Handwritten mapper:

{code:c#}
static void Map(BenchSource.Int1 s, ref BenchDestination.Int1 result)
{
	result.i = s.i;
	result.str1 = s.str1;
	result.str2 = s.str2;
}
static void Map(BenchSource.Int2 s, ref BenchDestination.Int2 result)
{
	Map(s.i1, ref result.i1);
	Map(s.i2, ref result.i2);
	Map(s.i3, ref result.i3);
	Map(s.i4, ref result.i4);
	Map(s.i5, ref result.i5);
	Map(s.i6, ref result.i6);
	Map(s.i7, ref result.i7);
}
static BenchDestination Map(BenchSource s)
{
	var result = new BenchDestination();
	Map(s.i1, ref result.i1);
	Map(s.i2, ref result.i2);
	Map(s.i3, ref result.i3);
	Map(s.i4, ref result.i4);
	Map(s.i5, ref result.i5);
	Map(s.i6, ref result.i6);
	Map(s.i7, ref result.i7);
	Map(s.i8, ref result.i8);

	result.n2 = s.n2;
	result.n3 = s.n3;
	result.n4 = s.n4;
	result.n5 = s.n5;
	result.n6 = s.n6;
	result.n7 = s.n7;
	result.n8 = s.n8;
	result.n9 = s.n9;

	result.s1 = s.s1;
	result.s2 = s.s2;
	result.s3 = s.s3;
	result.s4 = s.s4;
	result.s5 = s.s5;
	result.s6 = s.s6;
	result.s7 = s.s7;

	return result;
}
{code:c#}

Results of the benchmark:

{{Handwritten Mapper: 398 milliseconds
Emit Mapper: 358 milliseconds
Auto Mapper: 229061 milliseconds}}

The results are very interesting:
# AutoMapper is 640 times slower than EmitMapper. Probably it is not the best solution for time-critical tasks.
# Even handwritten code is a bit slower than EmitMapper, but why? The answer is that in respect to the programming practices i did not use the "Copy/Paste" technique and made submappers as separate methods:

{code:c#}
static void Map(BenchSource.Int1 s, ref BenchDestination.Int1 result)
{
	result.i = s.i;
	result.str1 = s.str1;
	result.str2 = s.str2;
}
{code:c#}

And then just used these mappers:

{code:c#}
Map(s.i1, ref result.i1);
Map(s.i2, ref result.i2);
Map(s.i3, ref result.i3);
Map(s.i4, ref result.i4);
...
{code:c#}

But EmitMapper as a bad programmer uses "Copy\Paste" and deals without additional calls. So, in this test EmitMapper is even faster than usual handwritten code.