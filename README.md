### Notes Taken while working through Effective C++, More Effective C++, Effective Modern C++ and Effective STL by Scott Meyers

This file contains detailed knowledge points of Scott Meyers four books.

## Effective C++

# 1. View C++ as a federation of languages

C++ was developed from four languages:

- C
- Object Oriented C++
- Template C++
- The STL

Summarization:

- The rules for efficient programming in C++ may vary from situation to situation depending on which sublanguage is used
- Therefore when theses four sublanguages are switched to each other, the efficiency has to be considered, e.g pass-by-value and pass-by-reference have different efficiencies in different languages.

# 2. Prefer consts, enums, and inlines to #defines

Prefer the compiler to the preprocessor

The preprocessor should be replaced by the compiler whenever possible, because the variables defined by the preprocessor don't enter the symbol table. So the compiler sometimes doesn't see the preprocessor definitions

use

	const double AspectRatio = 1.653;

instead of

	#define Ratio 1.653

Pointers can also be considered: pointers need to be written as const char * const name = "name"; So you have to use const twice.

To prevent constants in classes to be copied multiple times, they need to be defined as members of the class(adding static)

	class GamePlayer
	{
		static const int num = 5;
	}

Instead of using function like macros, you should use inline functions instead e.g. :

	#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))

	template<typename T>
	inline void callWithMax(const T& a, const T& b)
	{
		f(a > b a : b);
	}

Summarization:

- It is better to replace #define with consts and enums for simple constants
- It is better to replace function-like macros with inline functions

# 3. Use const whenever possible

If const appears to the left of the asteriks, it means that the reference is constant. If it appears to the right it means that the pointer itself is a constant. If it appears on both sides of the asterisk is means that both the reference and the pointer is contant

If the return value of a function is set to const, or the return pointer is set to const, many problems caused by user error can be avoided.

	class CTextBlock
	{
	public:
		char& operator[](std::size_t position) const
		{
			return pText[position];
		}
	private:
		char *pText;
	}

	const CTextBlock ccbt("Hello");
	char *cp = &cctb[0];
	*pc = 'J';

	In this case no error will be reported, but on one hand, it's said to be const when it 's declared, and on the other hand, the value is modified. Although this logic is problematic, the compiler will not report an error.

Howerver, there will be sitations where you want to modify a varibale during the use of const, and another part of the code does not need to be modiefied. The first thing that comes to mind at this time is to overload a non-const version.
But there are other ways such as calling the non-copnst version of the code to the const code.

Summarization:
- Declaring something as const helps the compuler to check out errors
- When the const and non-const versions have substantially equivalent implementations, letting the non-const version call the const version helps avoiding code duplication

# 4. Make sure that objects are initialized before they're used

For the C language in C++, initializingf variables may lead to lower efficieny of the runtime, but the C++ part should manually ensure initalization.

The initalization function is usually on the constructor(There is a difference between initialization and assignment, initizliazastion is effcientm assignment is inefficient, and these initialazations are in order)

In addition to these, if we have two files A and B, which need to be compiled separetely, and the objects in B are used in the constructor of A , then the order in which A and B are initialized is important. These varibales are called non local static objects

Initialization should be done before the body of a constructor is entered

	ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
	: theName(name).
	  theAdress(address),
	  thePhones(phones),
	  numTimesConsulted(0)
	{}

Summarization:
- Manual initialization for built in objects, because C++ does not guarentee that they are initalized
- Constructors are prerably initialized with columns rather than copying, and they are initialized in order
- To avoid initialzation order issues for cross-file compilation, cono loact static obkjects should be replaced with local static objects

# 5. Know what function C++ silently writes and calls

C++ declares their own versions of a copy constructor, a copy assignment opeerator, and a destructor if you don't declare them yourself

	class Empty();

is essentaillz the same as if you'd written:

	class Empty
	{
	public:
		Empty() { ... }
		Empty(const Empty& rhs) { ... }

		~Empty() { ... }
		
		Empty& perator=(const Empty& rhs) { ... }
	};

These functions are generated only if they are needed.

	Empty e1; // creates the default constructor and destructor(non virtual)

	Empty e1(e1); // creates the copy constructor

	e2 = e1; // creates the copy assignment operator

Summarization:
- The compiler can automatically generate default constructors, copy constructors, cpopy assignment operators and destructors for classes

# 6. Explicitly disallow the use of compiler-generated functions you do not want

When a class represents a unique transaction record, thjen the copy constructor and the copy assignment operator functions generated by the compiler are uselkess and unwanted.

Summarization:
- You casn set the unncessary default auto-generated function as private or make a private parent class and let your class inherit from it.

# 7. Declare destructors virtual in polymorphic base classes

If the base class does not have a virtual destructor, then twhenm the deriuved class is destruced i.e. if a pointer to the base class is delted, the derived object, will not be destroyed completly, causing memory leaks

	class Timekepper
	{
	public:
		TimeKepper();
		~TimeKepper();
		virtual getTimeKeeper();
	};

	class AtomicClock : public TimerKeeper {}
	TimeKepper *ptk = getTimeKeeper();
	delete ptk;

In addition to the destructoir, there are many functions. If a function has the virtual keyword, then its destructor must be virtual, but if the class does not contain a virtual function, the destructor should not add viretual. Because once the virtual function is implemented, the obnject must carry a pouinter called vptr(virtual table pointer), which points to an array of functiuon pointers, called vtbl(virtual table), so the size of the object will become larger.

	class Point
	{
	public:
		// Destrucotrs and Constructors
	private:
		int x, y;
	}

The above code should only 64bits (assuming an int is 32bits), and storiung a vptr it becomes 96bits.

Summarization:
- If a fuinction is a polymorphic base class, it should have avirtual destructor
- If a class has any virtual functions, it should have a virtual destructors
- If a class is not a polymorphic base class and has no virtual functions, it should not have a virtual destructor

# 8. Prevent exceptions from leaving destructors

If 10 Widgets are destructed in a looop, and each Widget throws and exception when destructing, multiple exception will exist at the same time. This is how you can control eachj exception in the destructor, to solve this problem :

	// Original Code
	class DBConn
	{
	public:
		~DBConn()
		{
			db.close();
		}
	private:
		DBConnection db;
	}

	// Better Code
	class DBConn
	{
	public:
		void close()
		{
			db.close();
			closed = true;
		}

		~DBCoinn()
		{
			if (!closed)
			{
				try
				{
					db.close();
				}	catch(...)
				{
					std::abort();
				}
			}
		}
	private:
		bool closed;
		DBConnection db;
	};

That way the close method can be handed over to the user, and when the user ignores it, it can also "force the end of the program" or "swallow the exception".

Summarization:
- Destructors should not throw exceptions, because they should be caught internally
- If a client needs to react to an exception thrown by an operation, the operation should be placed in a normal function(not the destructor).

# 9. Never call virtual functions during construction or destruction

You should not call virtual functions in the destructor and constructor, because the wrong version of the function is called when there is inheritance such as:

	// Original Code
	class Transaction
	{
	public:
		Transaction()
		{
			logTransaction();
		}
		virtual void logTransaction const() = 0;
	};

	class BuyTransaction : public Transaction
	{
	public:
		virtual void logTransaction() const;
	};

	BuyTransaction bt;

	class Transaction{
	public:
		Transaction()
		{
			init();
		}
	private:
		void init()
		{
			logTransaction();
		}
	};

At this time, the code will call the Transaction version of logTransaction. Because the construtor of the parent class is called first int the constructor, the logTransaction vertsion of the parent class will be fcalled first. The solutiuon is to not call it in the constructor nor to call the virtual one. But to make it non-virtual.

	// Better Code
	class Transaction
	{
	public:
		explicit Transaction(const std::string& logInfo);
		void logTransaction(const std::string& logInfo) const; // non virtual funtion
	};

	Transaction::Transaction(const std::strinbg* logInfo)
	{
		logTransaction(logInfo);	// non virtual
	}

	class BuyTransaction : public Transaction {
	public:
		BuyTransaction(parameters) : Transactrion(createLogString(parameters)) {...}
	private:
		static std::string createLogString(paramaeters);
	};

Summarization:
- Do not call virtual functions during construction and destruction, as such calls never secend tro the derivced class's version

# 10. Have assignment operators return a reference to *this

You should have assignment operators return a reference to *this to support continous reading and writing.

	class Widget
	{
	public:
		Widget& operator=(int rhs)
		{
			return *this;
		}
	}

	// a = b = c;

# 11. Handle assignment to self in operator=

You should mainly handle assignment to self in operator= because there may be a securtity issue or the original object is destroyed before being used.

	// Original Code
	class Bitmap {...};
	class Widget
	{
	private:
		Bitmap *pb;
	};

	Widget& Widget::operator=(const Widget& rhs)
	{
		delete pb;
		pb = new Bitmap(*rhs.pb);
		return *this;
	}



	// Better Code
	class Widget
	{
		void swap(Widget& rhs);
	};

	Widget& Widget::operator=(const Widget* rhs)
	{
		Widget temp(rhs);
		swap(temp);
		return *this;
	}

There is also a less efficient version:

	Widget& Widget::operator=(const Widget& rhs)
	{
		Bitmap *pOrig = pb;
		pb = new Bitmap(*rhs.pb);
		delete pOrig;
		return *this;
	}

Summarization:
- Ensure that operator= behaves well when objects are self-assigned, inlcuding addresses of two objects, statement order and copy-and-swap
- Make sure that any function is still correct if it operates on more than one object, many of which may point to the same object

# 12. Copy all parts of an object

Summarization:
- When writing a copy constructor or copy assignment constructor, you should make sure to copy all variables inside the members, as well as all base class memebers
- Don't try to use one copy constructor to call another copy constructor, if you want to somplify the code, you should put all the functions in a third function(init) and call it by both copy constructors
- When a new varibale is added or a class is inherited, it is easy to forget the copy constructor, so each time a varbiale is added, the corresponding method needs to be modified in the copy constructor

# 13. Use objects to manage resources

You should use objects to manage resources to prevent a return before thge delete statement is executes, so objects need to be used to manage these resources. In this way, when the flow of control leaves f, the object's destructor will automaticallyt release those resources. For example shared_ptr and auto_ptr are such objetc's that manage those resources. They do thge delete operation in their own destructor.

Summarization:
- It is recommended to use shared_ptr
- If you need a customized shared_ptr, you should manage resources by defining your own resource management class

# 14. Think carefully about copying behavior in resource-managing classes

In the resource managing class, if there is a copy behavior, you need to pay attention to the specific meaning of this copy, to ensure you get the effect you want.

	class Lock
	{
	public:
		explicit Lock(Mutex *pm) : mutexPtr(pm)
		{
			lock(mutexPtr);
		}
		~Lock()
		{
			unlock(mutexPtr);
		}
	private:
		Mutex *mutexPtr;
	};

	Lock m1(&m);
	Lock m2(m1);

The copy function may be gets automatically created by the compiler, so when using it, we must pay attention to whether the atomtically generated function fulfills our expectations.

Summarization:
- Copying the RAII object(Resource Aquisition is Initialization) must also copy the resource it manages(deep copy)
- Common RAII practise is: Prohibit copying, use the reference counting method

# 15. Provide accsess to raw resources in resource-managing classes

Provide accsess to raw resources in resource-managing classes with methods such as shared_ptr<>.get(), or -> and * to obtain values. But such a metrhod can be slightly cumbersome, some people will use an implicit conversion, but those often go wrong.

	class Font;
	class FontHandle;
	void changeFontSize(FontHandle f, int newSize) {...}

	Font f(getFont());
	int newFontSize = 3;
	changeFontSize(f.get(), newFontSize); // explicit

	class Font
	{
		operator FontHandle() const
		{
			return f; // implicit
		}
	};
	changeFontSize(f, newFontSize);
	Font f1(getFont());
	FontHandle f2 = f1;

Summarization:
- Every resource management class should have a method to obtain resources directly
- Implicit coversion is more convieneiet for clients. and explicit conversion is safer, you should choose depeneding on the requirements

# 16. Use the same form in corresponding uses of new and delete

Summarization:
- When using new[], use delete[], when using new don't use delete[]

# 17. Store newed objetcs in smart pointers in standalone statements

	int priority();
	void processWidget(sharded_ptr<Widget> pw, int priority);
	processWidget(new Widget, priority()); // error needs normal raw pointer
	processWidget(shared_ptr<Widget>(new Widget), priority()); // may cause memory leaks

	/*	The reason for the memory leak is that first it first executes the new for the widget, then it calls priority and then finally it calls the shared_ptr constructor. And when an exception occurs in the priority call, the pointer
		returned by the new Widget will be lost. Of course, different compilerers exceute the above code in different order. But there is a safe way to do this.
	*/

	shared_ptr<Widget> pw(new Widget);
	processWidget(pw, priority());

Summarization:
- Whenever there is a new statement, try to put it in a separate statement, especially when using the new object to put it in a smart pointer

# 18. Make interfaces easy to use correctly and hard to use incorrectly
