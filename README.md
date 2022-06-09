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

There are a lot of errors a client can make, you should create interfaces that are hard to use incorrectly

	Date(int month, int day, int year);
	/* There are a lot of problems with this piece of code. For example, the client could reverse the order of day and month */
	Date(const Month &m, const Day &d, const &year);
	// Note that each type of data is designed as a seperate class here, the month can be limited to only 12 months
	class Month
	{
	public:
		static Month Jan()
		{
			return Month(1);
		}
		// here functions are used instead of objects
	};

	// Functions that return pointers could also become an issue
	Investment *createInvestment(); // Smart pointers can prevent the user from forgetting to delete the pointer or from deleting the pointer twice
	std::shared_ptr<Investment> createInvestment(); // You can force the client to use smart pointers and even create custom deleters
	std::shared_ptr<Investment>pInv(static_cast<Investment*>(0), getRidOfInvestment);

# 19. Treat class design as type design

Consider the following when designing a class:
- How new class object should be created and constructed
- What should be the difference between initialization and assignment of objects (different function calls, constructors and assignment operators)
- What does it mean if the new class is passed by value?
- What are the resiotrctions on legal values for your new type?
- Does your new type fit into an inheritance graph?
- What kind of type conversions are allowed for your new type?
- What operators and functions make sense for the new type?
- What standard functions should be disallowed?
- Who should have access to the memebers of your new type?
- What is the "undeclared interface" of your new type?
- How general is your new type?
- Is a new type really what you need? Maybe it's better to just define a non-member function or template, if you just define a new derived clas or add functionality to the original class, 

# 20. Replce pass-by-value with pass-by-reference-to-const
Prefer pass-by=reference-to-const to pass-by-value, mainly to improve effciency, while avoiding the problem of parameters cutting between base clases and subclasses

	bool validateSudente(const Student &s);
	// Saves a lot of construction, destruction and copy assignment operations
	bool validateSudent(s);

	subStudent s;
	validateStudent(s);
	// After calling it the function only receives a student type, and there will be problems if there is an overloaded operation

Summarization:
- Prefer pass-by-reference-to-const over pass-by-value. It's typically more efficient and it avoids the slicing problem
- The rule doesn't apply to built-in types and STL iterator and function object types. For them pass-by-value is usually appropriate.

# 21. Don't try to return a reference when you must return an object

It is easy to return a local variable that has been destroyed. If you want to ceeate it with new on the heap, the user can't delete it. Even if you want to use static, there will be a lot of problem.

	inline const Rational operator*(const Rational &lhs, const Rational &rhs)
	{
		return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
	}

Of course, the cost of writing it this way is too high and effciencyt will be relativly low.

# 22. Declare data memeber private

You should make member variables private, and then use public memeber functions to access them. The advantage of this method is that you can control memeber variables more precisely, including controlling read-write, read-only access, etc.

At the same time, if the public variable is changed, and a variable is widely used in the code, then a lot of code is broken and in need to be rewritten.

In addition, protected is not more encapsulating than public, because protected varibales, when changed, their subclasss code will also be destroyed.

# 23. Prefer non-memeber non-friend functions to memeber functions

The difference is as follows

	class WebBrowser
	{
	public:
		void clearCache();
		void clearHistory();
		void removeCookies();
	};

	// member function
	class WebBrowser
	{
	public:
		// .....
		void clearEverything()
		{
			clearCache();
			clearHistory();
			removeCookies();
		}
	};

	// non-member non-friend function
	void clearBrowser(WebBrowser &wb)
	{
		wb.clearCache();
		wb.clearHistory();
		wb.removeCookies();
	}

Members can access private functions, enums, typedefs, etc. of the class, but non-memeber functions can't access those things, so non-member non-friend functions are better.

The usage of namespaces is also mentioned here. Namespaces can be used to devide some convenvience function and to put different typed of methods in the same namespace into different files(This is also how the C++ standard library is organized).

	#include "webbrowser.h"

	namespace WebBrowserStuff
	{
		class WebBrowser(){...}
	};

	#include "webbrowserbookmarks.h"
	namespace WebBrowserStuff
	{
		// all bookmark related convenvience functions
	};

# 24. Declare non-member functions when type conversion should apply to all parameters

If you want to multiply an int type varibale with a Rational variable and it's a member function, when an implicit conversion occurs, an error occurs because there is no type conversion from int to Rational

	class Rational
	{
	public:
		const Rational operator*(const Rational& rhs) const;
	};
	Rational oneHalf;
	result = oneHalf * 2;
	result = 2 * oneHalf; // Error, tries to convert int to Rational

	non-member
	class Rational{...};
	const Rational operator*(const Rational &lhs, const Rational &rhs) {...}

Summarization:
- If you need type conversions on all parameters to a function(including the one pointed to by the this pointer), the function must be a non-memeber

# 25. Consider support for a non-throwing swap

It is difficult to write a swap that is effcient, less prone to misunderstandings, and has consistency.

	// old code
	class Widget
	{
	public:
		Widget &operator=(const Widget &rhs)
		{
			*pImpl = *(rhs.pImpl); // inefficient
		}
	private:
		WidgetImpl* pImpl;
	};

	// new code
	namespace WidgetStuff
	{
		template<typename T>
		class Widget
		{
			void swap(Widget &other)
			{
				using std::swap; // this declaration is a specialization of std::swap
				swap(pImpl, other.pImpl);
			}
		}
	};

	...
	template<typename T> // non-memeber swap
	void swap(Widget<T> &a, Widget<T> &b) 
	{
		// does not belong in the std namespace
		a.swap(b);
	}



Swap implementation:

	namespace std
	{
		template<typename T>
		void swap(T& a, T& b)
		{
			T temp(a);
			a = b;
			b = temp;
		}
	}

Summarization:
- Provide a swap member function when std::swap would be inefficient for your type. Make sure your swap doesn't throw exceptions
- If you offer a member swap, also offer a non-member swap that calls the member. For classes(not templates), specialize std::swap, too.
- When calling swap, emply a using declaration for std::swap, then call swap without namespace qualification.
- It's fine to totally specialize std templates for user-defined types, but never try to add something completly new to std

# 26. Postpone variable definitions as long as possible

The main purpose is to prevent variables from not being used after they are defined, which affects efficiency. They should be defined when they are used, and initialized by default constructers instead of trough assignment.

# 27. Minimize casting

There are 6 ways of casting
- C-style casts
	(T) expression
- Function-style casts
	T(expression)
- const_cast is typically used to cast away the constness of objects. It is the only C++-style cast that can do this.
	const_cast<T>(expression)
- dynamic_cast is primarily used to perform "safe downcasting" i.e. to determine wheter an object is of a particular type in an inhgeritance hierarchy. It is the only cast that can't be performed using the old-style syntax . It's also the only cast that may havea significant runtime cost.
	dynamic_cast<T>(expression)
- reinterpret_cast is intended for low-level casts that yield implementation-dependent results, e.g. casting a pointer to an int. Such casts should be rare outside low-level code.
	reinterpret_cast<T>(expression)
- static_cast can be used to force implicit conversion(e.g. non-const object to const object, int to double, etc.). It can also be used to perform the reverse of many such conversions
	static_cast<T>(expression)

Try to not to perform casts, because:
- 1. Switching from int to doubnle is prone to precision errors
- 2. Converting a class to its parent class is also prone to

Summarization:
- Try to avoid casts, especially dynamic_cast in efficiency-consious ccode, and try to use alternative desiugns that don't require casts
- If the transformation is necessary, try to encapsulate it behind a function and let the client call the function without the need to cast in their own code
- If you need to convert, use modern-style C++-casts which are much better than C-style casts

# 28. Avoid returning "handles" to object internals

To prevent the user from misperating the returned value you should avoid returning handles to object internals

	// old code
	class Rectangle
	{
	public:
		Point& upperLeft() const { return pData->ulhc; }
		Point& lowerRight() const { return pData->lrhc; }
	};

	// new code
	class Rectangle
	{
	public:
		const Point& upperLeft() const { return pData->ulhc; }
		const Point& lowerRight() const { return pData->lrhc; }
	};
	// But there could potentionally be still dangling variables
	const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());

boundingBox will return a new, temporary Ractangle object for temp. After this entire statement is executed, temp becomes empty and becomes a dangling variable

Summarization:
- Try not to return pointers, references, etc. to private variables
- If you really want to use it,m try to use const to limit it, and try to avoid the possiblitly of suspension

# 29. Strive for exception-safe code 

Exception-safe functions have one of three characteristics:
- If an exception is thrown, everything within the program remains in a valid state, no objects or data structures are corrupted. Do not leak resources under any circumstances, and do not allow daya destruction under any circumstances.
- If an exception is thrown, the state of the program is not changed, and the program returns to the state before the function was called.
- promise to never throw exceptions

	class PrettyMenu
	{
	public:
		void changeBackground(std::ifstream& imgSrc); // change background image
	private:
		Mutex mutex; // Mutex
	};
	// old code
	void changeBackground(std::ifstream& imgSrc)
	{
		lock(&mutex);
		++changes;
		bgImage = new Image(imgSrc);
		unlock(&mutex);
	}
	/*	This Code has many problems, first it leaks resources and when an exception occurs in new Image(imgSrc), the call
		to unlock the mutex will never be executed and the mutex will be hed forever. Data curruption should be prvented, 
		If an exception occuyrs in tnew Image(imgSrc), bgImage is empty and changes has already been incremented.	
	*/

	// new code
	void PrettyMenu::changeBackground(std::ifstream& imgSrc
	{
		using std::swap;
		Lock ml(&mutex);
		std::shared_ptr<PMImpl> pNe(new PMImpl(*pImpl));
		pNew->bgImage.reset(new Image(imgSrc));
		++pNew->imageChanges;
		swap(pImpl, pNew);
	}

Summarization:
- Exception-safe functions leak no resources and allow no data structures to become corrupted, even when exceptions are thrown. Such functions offer the basic, strong, or nothrow gurantees.
- The strong guarantee can often be implemented via copy-and-swap, but the strong guarentee is not practical for all functions.
- A function can usually offer a guarantee no stronger than the weakes guarentee of the functions it calls.

# 30. Understand the ins and outs of inlining.

Excessive use of inline functions will increase the size of the program and cause high memory usage

The compiler can refuse to inline a function, and if the compiler does not know which function to call, it will report a warining

Try not to set the template or constructor as inline, because template inline may generate corresponding functions for each template, which will make the code too bloated. For the same reason, the constructor will also generate a lot of code in the actual process.

	class Derived : public Base
	{
	public:
		Derived(){} // it looks like an empty constructor but looks can mislead
	}
	
	Derived::Dervied
	{
		// line exception handling code
	}

# 31. Minimize compilation dependencies between files.

This relationship refers to the fact that one file contains the class definition of another file.

So to achieved decoupling, you define the implementation in another class

	// old code
	class Person
	{
	private:
		Dates date;
		Adresses adress;
	}

	// new code decoupling is achieved by seperating the implementation and the interface
	class PersonImpl;
	class Person
	{
	private:
		std::shared_ptr<PersonImpl> pImpl;
	}

Similiar interface classes can also use fully virtual functions

	class Person
	{
	public:
		virtual ~Person();
		virtual std::string name() const = 0;
		virtual std::string birthDate() const = 0;
	}

Then implement the relevant methods through the inheriting subclass

In this case these virtual functions are usually called factory functios

Summarization:
- The file should be made dependent on the declaration and not on the definiton, which can achieved by the two methods above
- Library header files should exist in full and declaration-only forms. This applies regardless of whether templates are involved

# 32. Make sure public inheritance models "is-a"

Public class inheritance refers to "is a"

	class Student : public Persone {...};

A student is a person, but a person is not necessarily a student

A common mistake here is to implement functions that may not exit in the parent class such as:

	class Bird
	{
		virtual void fly();
	}

	class Penguin : public Bird {...}; // Penguins can't fly

It is necessary to eliminate suchg error by design, for example, by defining a FlyBird

Summarization:
- When it comes to public inheritance, every Base class attribute must apply to its derived class

# 33. Avoid hiding inherited names

	class Base
	{
	public: 
		virtual void mf1() = 0;
		virtual void mf1(int);
		virtual void mf2();
		void mf3();
		void mf3(double);
	};

	class Derived : public Base
	{
	public:
		virtual void mf1(); 
		void mf3();
	};

This problem can be solves by doing this:

	using Base::mf1;
	// or
	virtual void mf1()
	{
		Base::mf1(); // forward function
	}

Summarization:
- Be mindful of the cope of functions/variables in derived classes
- You can solve problems like this by forwarding the functions

# 34. Differentiate between inheritance of interface and inheritance of implmentation

Pure virtual functions provide an interface inheritance. When a function is purely virtual, it means that all implementations are implemted in subclasses

	class Shape
	{
		virtual void draw() const = 0;
	};

	ps->Shape::draw();

Summarization:
- Inheritance of interface is different from inheritance of implementation. Under public inheritance derived classes always inherit base class interfaces
- Pure virtual functions specify inheritance of interface only.
- Simple (impure) virtual functions specify inheritance of interface plus inheritance of a default implementation.

# 35. Consider alternatives to virtual functions

NVM method: indirectly call private virtual function through public non-virtual member function, the so-called template method design pattern:

	class GameCharacter
	{
	public:
		int healthValue() const
		{
			// do "before" stuff
			int retVal = doHealthValue(); // do the real work
			// do "after" stuff
		}
	private:
		virtual int doHealthValue() const {...} // default algorithm calculating healthfunc
	}

The advantaghe of this approach is the "before" and "after" statements, that ensure that the virtual function is called separately before and after the actual work

But there are also other methods like e.g. the Strategy design pattern

	class GameCharacter;
	int defaultHealthCalc(const GameCharacter& gc);
	class GameCharacter
	{
	public:
		typedef int (*HealthCalcFunc)(const GameCharacters&); // function pointer
		explicit GameCharacter(HealthCalcFunc hfc = defaultHealthCalc) : healthFunc(hcf) {}
		int healthValue() const { return healthFunc(*this); }
	private:
		HealthCalcFunc healthFunc;
	}

	// you should strive to use a function object instead of a function pointer

	typedef std::function<int (const GameCharacter&)> HealthCalcFunc; // typedef that behaves more like a function pointer

Summarization:
- Alternatives to virtual functions include the NVM idiom and various forms of the Strategy design patter. The NVM idiom is itself an example of the Template Method design pattern.
- A disadvantage of moving functionality from a member function to a function outside the class is that the non-member function lacks access to the class's non-public members
- std::function objects act like generalized function pointers. Such objects support all callable entities compatible with a given target signature.

# 36. Never redefine an inherited non-virtual function

	class B
	{
	public:
		void mf();
	};

	class D : public B
	{
	public:
		void mf();
	};

	D x;
	B* pB = &x;
	D *pD = &x;

	pB->mf(); // calls the B version
	pD->mf(); // calls the D version

Even ignoring this code difference, if it were redefined in this way, it would not meet the previous definiton of "every D is a B"

Summarization:
- Never redefine an inherited non-virtual function

# 37. Never redfine a function's inherited default parameter value

	class Shape
	{
	public:
		enum ShapeColor { Red, Green, Blue };
		virtual void draw(ShapeColor color = Red) const = 0;
	};
	
	class Rectangle : public Shape
	{
	public:
		virtual void draw(ShapeColor color = Green) const; // Different default parameters as the parent class
	};

	Shape *pr = new Rectangle; // static type of pr is Shape, but the dynamic type is Rectangle
	pr->draw(); // The virtual function is dyunamically bound and the default parameter value is statically bound, so Red will be called

Summarization:
- Never redefine an inherited default parameter value, because default parameter values are statically bound, while virtual functions, (the only functions you should overriding) are dynamically bound.

# 38. Model "has-a" or "is-implemented-in-terms-of" through composition

A composition is the relationship between types that arise when objects of one type contain objects of another type.

	template<class T>
	class Set
	{
	public:
		void insert();
		// etc. 
	private:
		std::list<T> rep;
	};

Set is not a list, but has a list

Summarization:
- Composition has meanings completly different from that of public inheritance
- In the application domain, composition means has-a. In the implementation domain, it means is-implemted-in-terms-of.

# 39. Use private inheritance judicously

Private inheritance is not an is-a relationship, i.g. some privatre members of the parent class are inaccessible to subclasses, and after private inheritance, all members of the subclass are private, which mean it's is-implemented-in-terms-of. Most of the time you should use composition instead of private inheritance.

Summarization:
- Privatye inheritance means is-implemented-in-terms-of. It's usually inferior to composition, but makes sense when a derived class needs access to protected base class members or needs to redefine inherited virtual functions.
- Unlike compositon, private inheritance can enable the empty base opimization. This can be importnat for library developers who strive to minimize object sizes.

# 40. Use multiple inheritances judiciously

Multiple inheritance can easily case name conficts

	class BorrowableItem
	{
	public:
		void checkOut();
	};

	class ElectronigGadget
	{
	private:
		bool checkOut() const;
	};

	class MP3Plazer : public BorrowableItem, public ElectronicGadget { ... };
	MP3Player mp;
	mp.checkOut(); // ambiguious the compiler doesn't know which function to call even if one function is private
	// fixable tho
	mp.BorrowableItem::checkOut();

In practical applications, two classes often inherit from the same parent class, and then one class inherits these two classes

	class Parent { ... };
	class First : public Parent { ... };
	class Second : public Parent { ... };
	class last : public First, public Second { ... };

Summarization:
-  Multiple inheritance is more complex than single inheritance. It can lead to new ambiguity issues and to the need for virtual inheritance.
- Virtual inheritance imposes costs in size, speed and complexity of initialization and assignment. It's most practical when virtual base classes have no data.
- Multiple inhertance does have legitimate uses, One scenario involves combing public inheritance from an Interface class with private inheritance from a class that helps with implementation

# 41. Understand implicit interfaces and compile-time polymorphism

Solve problems with explicit interfaces and runtime polymorphism:

	class Widget
	{
	public:
		Widget();
		virtual ~Widget();
		virtual std::size_t() const;
		void swap(Widget& other);
	};

	void doProcessing(Widget& w)
	{
		if (w.size() > 10) { ... }
	}

Since w is declared as an Widget, w must support the Widget interface, and we can find out what this interface looks like in the source code (explicit interface), which means the code is cleary visible in the source code

Since some member functions of Widget are virtual, the calls of w to those functions will show run-time polymorphism, i.e. which function to call during runtime is determined according to the dynamic type of w. 

When it comes to template programming, Implicit interface and compile-time polymorphism are more important:

	template<typename T>
	void doProcessing(T& w)
	{
		if (w.size() > 10) { ... }
	}

Which interface w must support is determnined by the operations performed on w, in the template. For example, T must support functions such as size. This is called an implicit interface.

Any function call involving w, such as operator>, may cause the template to be instantiated, making the call successful, and different functions are instantiated according to different T calls, which is called compile-time-polymorphism

Summarization:
- Both classes and templates support interface and polymorphism.
- For classes, interfaces are explicit and centered on function signatures, Polymorphism occurs at runtime through virtual functions.
- For template parameters, interfaces are implicit aand based on valid expressions. Polymorphism occurs during compilation through template instanitation and function overloading resolution.

# 42. Understand the two meanings of typename

	template <typename C>
	void print2nd(const C& container)
	{
		if (container.size() >= 2)
		{
			typename C::const_iterator iter(container.begin());
		}
	}

The typename there means that C::const_iterator is a typename, because there may be not be a const_iterator in the C type or that there is a variable named const_iterator in type C

Summarization:
- When declaring template parameters, class and typename are interchangable.
- Use typename to identify nested dependednt type names, except in base class list or as a base class identifier in a member initilization list.