# Notes Taken while working through Effective C++, More Effective C++, Effective Modern C++ and Effective STL by Scott Meyers

This file contains detailed knowledge points of Scott Meyers four books.

## Effective C++

### 1. View C++ as a federation of languages

C++ was developed from four languages:

- C
- Object Oriented C++
- Template C++
- The STL

Summarization:

- The rules for efficient programming in C++ may vary from situation to situation depending on which sublanguage is used
- Therefore when theses four sublanguages are switched to each other, the efficiency has to be considered, e.g pass-by-value and pass-by-reference have different efficiencies in different languages.

### 2. Prefer consts, enums, and inlines to #defines

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

### 3. Use const whenever possible

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
	};

	const CTextBlock ccbt("Hello");
	char *cp = &cctb[0];
	*pc = 'J';

	In this case no error will be reported, but on one hand, it's said to be const when it 's declared, and on the other hand, the value is modified. Although this logic is problematic, the compiler will not report an error.

Howerver, there will be sitations where you want to modify a varibale during the use of const, and another part of the code does not need to be modiefied. The first thing that comes to mind at this time is to overload a non-const version.
But there are other ways such as calling the non-copnst version of the code to the const code.

Summarization:
- Declaring something as const helps the compuler to check out errors
- When the const and non-const versions have substantially equivalent implementations, letting the non-const version call the const version helps avoiding code duplication

### 4. Make sure that objects are initialized before they're used

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

### 5. Know what function C++ silently writes and calls

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

### 6. Explicitly disallow the use of compiler-generated functions you do not want

When a class represents a unique transaction record, thjen the copy constructor and the copy assignment operator functions generated by the compiler are uselkess and unwanted.

Summarization:
- You casn set the unncessary default auto-generated function as private or make a private parent class and let your class inherit from it.

### 7. Declare destructors virtual in polymorphic base classes

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

### 8. Prevent exceptions from leaving destructors

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

### 9. Never call virtual functions during construction or destruction

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

### 10. Have assignment operators return a reference to *this

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

### 11. Handle assignment to self in operator=

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

### 12. Copy all parts of an object

Summarization:
- When writing a copy constructor or copy assignment constructor, you should make sure to copy all variables inside the members, as well as all base class memebers
- Don't try to use one copy constructor to call another copy constructor, if you want to somplify the code, you should put all the functions in a third function(init) and call it by both copy constructors
- When a new varibale is added or a class is inherited, it is easy to forget the copy constructor, so each time a varbiale is added, the corresponding method needs to be modified in the copy constructor

### 13. Use objects to manage resources

You should use objects to manage resources to prevent a return before thge delete statement is executes, so objects need to be used to manage these resources. In this way, when the flow of control leaves f, the object's destructor will automaticallyt release those resources. For example shared_ptr and auto_ptr are such objetc's that manage those resources. They do thge delete operation in their own destructor.

Summarization:
- It is recommended to use shared_ptr
- If you need a customized shared_ptr, you should manage resources by defining your own resource management class

### 14. Think carefully about copying behavior in resource-managing classes

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

### 15. Provide accsess to raw resources in resource-managing classes

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

### 16. Use the same form in corresponding uses of new and delete

Summarization:
- When using new[], use delete[], when using new don't use delete[]

### 17. Store newed objetcs in smart pointers in standalone statements

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

### 18. Make interfaces easy to use correctly and hard to use incorrectly

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

### 19. Treat class design as type design

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

### 20. Replce pass-by-value with pass-by-reference-to-const
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

### 21. Don't try to return a reference when you must return an object

It is easy to return a local variable that has been destroyed. If you want to ceeate it with new on the heap, the user can't delete it. Even if you want to use static, there will be a lot of problem.

	inline const Rational operator*(const Rational &lhs, const Rational &rhs)
	{
		return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
	}

Of course, the cost of writing it this way is too high and effciencyt will be relativly low.

### 22. Declare data memeber private

You should make member variables private, and then use public memeber functions to access them. The advantage of this method is that you can control memeber variables more precisely, including controlling read-write, read-only access, etc.

At the same time, if the public variable is changed, and a variable is widely used in the code, then a lot of code is broken and in need to be rewritten.

In addition, protected is not more encapsulating than public, because protected varibales, when changed, their subclasss code will also be destroyed.

### 23. Prefer non-memeber non-friend functions to memeber functions

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

### 24. Declare non-member functions when type conversion should apply to all parameters

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

### 25. Consider support for a non-throwing swap

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

### 26. Postpone variable definitions as long as possible

The main purpose is to prevent variables from not being used after they are defined, which affects efficiency. They should be defined when they are used, and initialized by default constructers instead of trough assignment.

### 27. Minimize casting

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

### 28. Avoid returning "handles" to object internals

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

### 29. Strive for exception-safe code 

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

### 30. Understand the ins and outs of inlining.

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

### 31. Minimize compilation dependencies between files.

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

### 32. Make sure public inheritance models "is-a"

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

### 33. Avoid hiding inherited names

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

### 34. Differentiate between inheritance of interface and inheritance of implmentation

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

### 35. Consider alternatives to virtual functions

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

### 36. Never redefine an inherited non-virtual function

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

### 37. Never redfine a function's inherited default parameter value

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

### 38. Model "has-a" or "is-implemented-in-terms-of" through composition

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

### 39. Use private inheritance judicously

Private inheritance is not an is-a relationship, i.g. some private members of the parent class are inaccessible to subclasses, and after private inheritance, all members of the subclass are private, which mean it's is-implemented-in-terms-of. Most of the time you should use composition instead of private inheritance.

Summarization:
- Private inheritance means is-implemented-in-terms-of. It's usually inferior to composition, but makes sense when a derived class needs access to protected base class members or needs to redefine inherited virtual functions.
- Unlike compositon, private inheritance can enable the empty base opimization. This can be important for library developers who strive to minimize object sizes.

### 40. Use multiple inheritances judiciously

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

### 41. Understand implicit interfaces and compile-time polymorphism

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

### 42. Understand the two meanings of typename

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

### 43. Know how to access names in templatized base classes

	// old code
	class CompanyA
	{
	public:
		void sendCleartext(const std::string& msg);
		// ...
	};

	class CompanyB { ... };

	template <typename Company>
	class MsgSender
	{
	public:
		void sendClear(const MsgInfo& info)
		{
			std::string msg;
			Company c;
			c.sendCleartext(msg);
		}
	};

	template <typename Company>
	class LoggingMsgSender : public MsgSender<Company> // for logging the msg wehn sending it
	{
	public:
		void sendClearMsg(const MsgInfo& info)
		{
			// log
			sendClear(info); // failes to compile because the compiler couldn't find a specialised MsgSender<company>
			// log
		}
	};

Workaround 1:

	template <> // generate a fully specialized template
	class MsgSender<companyZ>
	{
	public:
		void sendEcnrypted(const MsgInfo& info) { ... };
	};

Workaround 2:

	template <typename Company>
	class LoggingMsgSender : public MsgSender<Company>
	{
	public:
		void sencClearMsg(const MsgInfo& info)
		{
			// log
			this->sencClear(info); // supposing that sencClear will be inherited
			// log
		}
	};

Workaround 3:

	template <typename Company>
	class LoggingMsgSender : public MsgSender<Company>
	{
	public:
		using MsgSender<Company>::sendClear; // telling the compiler that it can assume that sendClear is going to be inside the base class

		void sendClearMsg(const MsgInfo& info)
		{
			// log
			sencClear(info); // supposing that sencClear will be inherited
			// log
		}
	};

Workaround 4:

	template <typename Company>
	class LoggingMsgSender : public MsgSender<Company>
	{
	public:
		void sencClearMsg(const MsgInfo& info)
		{
			MsgSender<Company>::sendClear(info);
		}
	};

Summarization:
- In derived class templates, refer to names in base class templates via a "this->" prefix, via using declarations, or via an explicit base class qualification.

### 44. Factor parameter-independent ocde out of templates

To not make the compiler compile long and bloated binary code, you should extract the parameters.

	template <typename T, std::size_t n>
	class SquareMatrix
	{
	public:
		void invert(); // invert the matrix
	};

	SquareMatrix<double, 5> sm1;
	SquareMatrix<double, 10> sm2;
	sm1.invert();
	sm2.invert(); // will have two inverts that are basically indentical

	// new code
	template <typename T>
	class SquareMatrix : private SquareMatrixBase<T>
	{
	private:
		using SquareMatrixBase<T>::invert; // avoids shadowing the base version of invert
	public:
		void invert()
		{
			this->invert(n); // An inline call that calls the base class version of invert
		}
	};

Because the matrix data may be different, e.g. the it's a 5x5 or 10x10 matrix, the input matrix data will also be different. It is better to use a pointer to point to the matrix data.

	template <typename T, std::size_t n>
	class SqureMatrix : private SquareMatrixBase<T>
	{
	public:
		SquareMatrix::SquareMatrix<T>(n, 0), pData(new T[n*n])
		{
			this->setDataPtr(pData.get());
		}
	private:
		boost::scoped_array<T> pData; // exists in heap
	};

Summarization:
- Templates generate multiple classes and multiple functions, so any template code not dependent on a template parameter causes bloat.
- Bloat due to non-type template parameters can often be eliminated by replacing template parameters with function parameters or class data members.
- Bloat due to type parameters can be reduced by sharing implementations for instiantiation types with identical binary representations.

### 45. Use member function templates to accept "all compatible types"

	Top* pt2 = new Bottom; // Converting Bottom to Top is simple
	template <typename T>
	class SmartPtr
	{
	public:
		explicit SmartPtr(T* realPtr);
	};

	SmartPtr<Top> pt2 = SmartPtr<Bottom>(new Bottom); // Converting SmartPtr<Bottom> to SmartPtr<Top> is a bit troublesome

	// better way
	template <typename T>
	class SmartPtr
	{
	public:
		template <typename U>

		SmartPtr(const SmartPtr<U>& other) : // generate copy constructor
		heldPtr(other.get())
		{}

		T* get() consat
		{
			return heldPtr;
		}
	private:
		T* heldPtr; // the built in raw pointer held by this SmartPtr
	};

Summarization:
- Use member function templates to generate functions that accept all compatible types.
- If you declare member templates for generalized copy construction or generalized assignment, you'll still need to declare the normal copy contructor and copy assignment operator, too.

### 46. Define non-memeber functions inside templates when type conversions are desired

When we perform mixed-type arithmetic operations, there will be cases where the compilation fails

	template <typename T>
	const Rational<T> operator* (const Rational<T>& lhs, const Rational<T>& rhs) { ... };

	Rational<int> oneHalf(1, 2);
	Rational<int> result = oneHalf * 2;

To fix this problem you can just declare a friend function and make a mixed call

	template <typename T>
	class Rational
	{6
	public:
		friend const Rational operator* (const Rational& lhs, const Rational& rhs)
		{
			return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
		}
	};

	template <typename T>
	const Rational<T> operator* (const Rational<T>& lhs, const Rational<T>& rhs) { ... }

Summarization:
- When writing a clas template that offers functions related tot the template that support implicit type conversions on all parameters, define those functions as friends inside the class template.

### 47. Use traits classes for information about types

Traits are a techique that allows you to obtain certain types of information at compile time, or a protocol. One of the requirements of this technique is that it must behave the same for built-in types and user-defined types.

	template <typename T>
	struct iterator_traits; // information about iterator classification
	// The way iterator_traits works is that for a certain type IterT, it must be declared in struct iterator_traits<IterT> a typedef named iterator_category. This typedef is used to confirm the iterator classification of IterT. And a class designed for deque could look like this:

	template <...>
	class deque
	{
	public:
		class iterator
		{
		public:
			typedef random_access_iterator_tag iterator_category;
		}
	};

	// For user-defined iterator_traits, there is a IterT, which says what it means by itself
	template <typename IterT>
	struct iterator_traits
	{
		typedef typename IterT::iterator_category iterator_category;
	}

	// The iterator type specified by iterator_traits for pointer are:
	template <typename IterT>
	struc iterator_traits?<IterT*>
	{
		typedef random_access_iterator_tag iterator_category;
	}

Here's how to design and implement a traits-class:
- Confirm some type_related information that you want to be available in the future, e.g. for iterators we want to have their classification available in the future.
- Choose a name for this information (e.g. iterator_category)
- Provide a template and a set of specializations (such as iterator_traits) containing information about the types you want to support

After designing and implementing a traits class, we need to use this traits-class:

	template <typename IterT, typename DistT>
	void doAdvance(IterT& iter, DistT d, std::random_access_iterrator_tag) { iter+= d;} // used to implement random access iterators

	template <typename IterT, Disd d, std::bidirectional_iterator_tag>
	{
		// used to implement bidirectional access iterators
		if (d >= 0)
		{
			while (d--)
				++iter;
		} else
		{
			while(d++)
				--iter;
		}
	}

	template <typename IterT, typename DistT>
	void advance(IterT& iter, DistT d)
	{
		doAdvance(iter, d, typename std::iterator_traits_traits<IterT>)::iterator_category());
	}

Use a traits class:
- Create a set of overloaded functions or function templates, the only difference between them is their respective traits parameters, so that each function implementation code corrosponds to the traits information it accepts
- Create a control function or function templete that calls the above overloaded function and passes the information provided by the traits class

Summarization:
- Traits classes make information about types available during compilation. They're implemented using templates and template specializations.
- In conjunction with overloading, traits classes make it possible to perform compile-time if...else tests on types.

### 48. Be aware of template metaprogramming

Writing programs that are executed during compilation is metaprogramming. Because these codes run in the compiler instead at runtime, the efficiency will be very high and some problems that are easy to occur during runtime are also exposed.

	// meta programming for calculating Ffactorials
	template <unisnged n>
	struct Factorial
	{
		enum
		{
			value = n * Factorial<n-1>::value;
		};
	};
	template <>
	struct Factorial<0>
	{
		enum ( value = 1);
	};

Summarization:
- Template metaprogramming can shift work from runtime to compile-time, thus enabling earlier error detection and higher runtime performance.
- TMV can be used to generate custom code based on combinations of policy choices, and it can also be used to avoid generating code inappropriate for particular types.

### 49. Understand the behavior of the new-handler

When new can't allocate new memory, it will keep callingthe new-handler until enough memory is found, new_handl;er is an error handling function:

	namespace std
	{
		typedef void(*new_handler)();
		new_handler set_new_handler(new handler p ) throw();
	}

A well-designed new-handler does the following:
- make more memory available
- install another new-handler. If this new-handler can't obtain more availbable memory at present, it knows which other new-hand;ler has the ability, and then replaces itself with that new-handler.
- remove new-handler
- throws an exception when bad_alloc
- Doesn't return, call abort or exit

The new-handler can't be customized for each class, but it can rewrite the new operator and design its own new-handler. At this time, the new should bne similar to the following implementation.

	void* Widget::operator new(std::size_t size) throw(std::bad_alloc)
	{
		NewHandlerHold h(std::set_new_handler(currentHandler));
		return ::operator new(size); // Allocate memory or throw an exception, restore global new-handler
	}

Summarization:
- set_new_handler allows clients to specify a function to be called when a memory allocation can't be satisfied
- Nothrow new is of limited utility , because it applies only to memory allocation; subsequent constructor calls may still throw exceptions.

### 50. Understand when it makes sense to teplcae new and delete

- It is used to detect errors in use. If the new memory fails tyo delte, it will cause memory leaks. When customizing, it can detect and locate the corrsponding failure location.
- In order to strengthen efficiency(traditional new is made to meet varous needs, so the efficiency is very moderate)
- Statistics on usage can be collected
- To increase the speed of allocating and returning memory.
- In order tyo reduce the space overhead caused by the dafault memory manager
- To compensate for non-optimal alignment bits in the dafault allocator
- To cluster related objects together

Summarization:
- There are many valid reasons for writing custom versions of new and delete, including improving performance, debugging heap usage erros, and collecting heap usage information.

### 51. Adhere to convention when writing new and delete

When rewiring new, it is necessary to ensure conditions, and to be able to handle all unexpected situations such as the 0-bytes memory application

When overriding delete, make sure that it is always safe to delete a null pointer

Summarization:
- operator new should contain an infinite loop tring to allocate memory, should call the new handler if it can't satistya memory request, and should handle requests for zero bytes. Clas specific versions should handle requests for larger blocks than expected.
- operator delete should do nothing if passed by a pointer that is null. Class-specific versions should handle blocks that are larger than expected

### 52. Write placement delete if you write placement new

If the parameters accepted by operator new have other parameters besides the certain size_t, this is the so-called place,emt new

	void* operator new(std::size_t, void *pMemory) throw(); // placement new
	static void operator delete(void* pMemory) throw(); // placement delete

Summarization:
- When you write a placement version of operator new, be sure to write the corresponmding placement version of operator delete. If you don't your program may experience subtle, intermittent memory leaks.
- When you delcare placement versions of new and delete, be sure not to unintentionally hide the normal versions of those functions

### 53. Pay attention to compiler warnings

Take warnings issued by the compiler serously, and strive to have no warnings at the highest warning level of the compiler

At the same time, don't rely too much on the compiler's warnings, because different compilers may have diffeernet attitudes towards things, and there may be no warnings if you change a compiler.

### 54. Familiarize yourself with the standard library including TR1

This one is outdates, but it's still (kinda) useful to know

- smart pointers
- tr1::function : representsany callable entity
- tr1::bind is a stl binder
- Hash tables such as sets, multisets, maps, etc.
- regular expressions
- tuples variable group
- tr1::array: Essentiallyt an STL array
- tr1::mem_fn: Essentially a function pointer
- tr1::reference_wrapper: something that makes referneces behave more like objects
- random number generator
- type traits

Summarization:
- The primary standard c++ library functionality consists of the STL, iostream and locals, The C99 standard library is also included
- TR1 adds support for smart pointers, generalized function poiinters, hash-based cntainers, regular expressions, and 10 other components
- TR1 itself is only a specification. To take advantage of TR1, you need an implementation. One source for implementations of TR1 components in Boost.

### 55. Familiarize yourself with Boost.

- Boost is a community and web site for the development of free, open source, peer-reviewed C++ libraries. Boost plays an influential role in C++ standardization.
- Boost offers implementations of many TR1 components, but it also offers many other libraries, too

## More Effective C++

### 1. Distinguish pointers and references

References must point to an object, they can't refer to a null value.

	char* pc = 0; // set pointer to null
	char& rc = *pc; // undefined behavior can arise now because the reference is pointing to null

Summarization:
- Use pointers when there is a possibility that it does not point to any object and it needs to be able to point to different objects at different times.

### 2. Prefer C++ style casts

Was mentioned in the previous book (item 27)

### 3. Never use polymorphism with Arrays

	class BST { ... };
	class BalancedBST : public BST { ... };

	void printBSTArray(const BST array[])
	{
		for (auto i : array)
		{
			std::cout << *i << std::endl;
		}
	}

	BalancedBST bstArray[10];
	printBSTArray(bstArray);

The compiler, has no emmits no warning in this case, and the object is passed according to the declared size during the transfer process, so the interval between each element is size of(BST) and the pointer points to the wrong place.

### 4. Avoiud unnecessary default constructors

To prevent the occurence of objects but with no necessary data.

	class EP
	{
	public:
		EP(int ID);
	}

This can be a problem especially when Ep is a virtual class

	EP bestEP[] = {
		EP(ID1),
		EP(ID2),
		...
	}

A method of an array function or a pointer:

	typedef EP* PEP;
	PEP bestPieces[10];
	PEP *bestPieces = new PEP[10]; // then re-initialize with new when you use it

### 5. Be wary of user-defined coversion functions

	class Rational
	{
	public:
		Rational(int numerator = 0, int denominator = 0);
		operator double() const;
	};

	Rational r (1, 2);
	double d = 0.5 * r; // converts r to a double for the calculation

	std::cout << r << std::endl; // Will call the closest type conversion function, converts r to a double and will print it

The Solution to this problem is the following:

	double asDouble() const;

But even if you do this, there may still be an implicit type conversion:

	template <class T>
	class Array
	{
	public:
		Array(int size);
		T& operator[](int index);
	};

	bool operator==(const Array<int> &lhs, const Array<int> &rhs);
	Array<int> a(10), b(10);
	if (a == b[3]) ... // if you accidentially wanted to write a[3] == b[3], the compiler will not report an error and tries to cnvert the array to an int

	explicit Array(int size); // that prevents the compiler from making implicit conversions
	if (a == b[3]) // error!!!

Here is another bad case:

	class Array
	{
	public:
		class ArraySize
		{
		public:
			ArraySize(int numElements) : theSize(numElements) {}
			int size() const { return theSize; }
		private:
			int theSize;
		};
		Array(int lowBound, int highBound);
		Array(ArraySize size);
		...
	};

When you write it this way, ( a(10); ) the compiler will first convert the type to ArraySize, and then construct it. Although it can preent implicit conversion, it will make your code slower.

### 6. Distinguish between prefix and postfix forms of increment and decrement operators

Overloading forms of prefixes and postfix is different. And you should use prefixes whenever possible because a postfix creates an "oldvalue" that has to be constructed and destructed which makes it less effcient.

	// postfix
	const UPint UPint::operator++(int)
	{
		const UPint oldValue = *this;
		++(*this);

		return oldValue;
	}

### 7. Never overload &&, ||, or ,.

Just because you can overload them doesn't mean you should. Overloading those functions disables short circuting.

More special is the comma operator ",", it's most commenly used in for loops:

	for (int i = 0, j = strlen(s) -1; i < j; ++i, ++j);

Because the last part of this function uses an expression, it is illegal to separate the expression to change the values of i and j in this for loop. Using a comma expression will first calculate ++i on the left,m and then calculate ++j on the right of the comma and you can't really implement that when you overload the comma operator.

### 8. Understand the meanings of new and delete in different situations

There are two kinds of new: there is the new operator and there is operator new

	string *ps = new String("memory managment"); // new operator

	void * operator new (size_t size); // operator new, you can override this function to change how memory is allocated

Operator new is generally not called directly, buit it can be called like any other functions:

	void* rawMemory = operator new(sizeof(String));

There is also placement new, placement new is used for memory that has been allocated but not preocessed yet. It is necessary to construct an object in your memory.

	class Widget
	{
	public:
		Widget(int widgetSize);
		....
	};

	Widget* constructWidgetInBuffer(void *buffer, int widgetSize)
	{
		return new(buffer) Widget(widgetSize);
	}

This returns a pointer to a Widget object, which was already allocated in the buffer that was passed to the function

delete buffer; // referst to calling the destructor of the buffer first, and then releases the memory
operator delete(buffer); // releases the memory but doesn't call the destructor

For the memory generated by placement new, the delete operator shouldn't be called, because the delete operator uses operator delete to free the memory, but the memory containing the object was not originally allocated by operator new.

new[] and delete[] are equivalent to calling the constructor and destructor for each array element

### 9. Use destructors to prevent resource leaks

	// old code
	void processAdoptions(istream& dataSource)
	{
		while (dataSource)
		{
			ALA *pa = readALA(dataSource);
			try {
				pa->processAdoption();
			}
			catch ()
			{
				delete pa; // avoid leaks when pa->processAdoption() throws an exception
				throw;
			}
			delete pa;
		}
	}

Code duplication often means that you're doing something wrong or that there is a better way to write this. We should use a smart pointer;

	void processAdoptions(istream& dataSource)
	{
		while (dataSource)
		{
			auto pa = std::make_unique<ALA>(dataSource);
			pa->processAdoption();
		}
	}

The Idea behind using a unique_ptr is to use an object to store resources that need to be automatically released, and then to rely on the objects destructor to release the resources.

### 10. Prevent resource leaks in constructors.

This item is about how to prevent resource leaks caused by exceptions in the constructor. 

	BookEntry::BookEntry(const imageFileName imgName, const soundFileName souName)
	:	thisImage(imgName),
		thisAudio(souName)
	{}

	BookEntry::~BookEntry()
	{
		delete thisImage;
		delte thisAudio;
	}

If there occurs an exception in the constructor, then the destructor will not be called, and it will cause memory leaks
There are two good solutions to this problem.

1. Let the initializer list call a private function that catches the exception with a try and catch block

	BookEntry::BookEntry(const imageFileName imgName, const soundFileName souName)
	:	thisImage(initImage(imgName)),
		thisAudio(initAudio(souName))
	{}

2. Make use of smart pointers so that the memory gets automatically freed

	class BookEntry
	{
	public:
		.....
	private:
		const std::unique_ptr<Image> thisImage;
		const std::unique_ptr<Audio> thisAudio;
	};

### 11. Prevent exceptions from leaving destructors

If the destructor throws an exception, it will cause the program to directly terminate without releasing the object, so the exception should not be passed to the outside of the destructor, but should be caught directly in the destructor and should be disposed of.

Also, if the destructor throws an exception, then the destructor won't fully run, and thus won't be able to do other you want it to do.

	Session::~Session()
	{
		logDestruction((this));
		endTransaction(); // If the statement above fails this statement will not be executed.
	}

### 12. Understand how throwing an exception differs from passing a parameter or calling a virtual function

A function that passes one parameter:

	void f1(Widget w);

catch clause:

	catch(Widget w)...

They have some similarities and differences, they are similar in the way that they can pass-by-value, pass-by-reference and pass-by-pointer. But the process that the program needs to complete the operation is completly different. When a function is called, the control of the program will also return to the caller of the function, but when an exception is thrown, control will never return to the place where the exception was thrown.

There are three ways to catch exceptions:

	catch(Widget w);
	catch(Widget& w);
	catch(const Widget& w);

If an object is thrown it can be catched by a normal reference, it does not need to be catched by a reference to a const object. Another difference is that in the try block, the thrown exception will not be converted as long as it's not a conversion of a derived class to a base class.

	void f(int value)
	{
		try
		{
			throw value; // it doesn't matter wheter value is an int or double or another type
		}
		catch(double d) // Only exceptions of type double are handled here, if its an int or any other type it will be ignored
		{
			...
		}
	}

### 13. Catch exceptions by reference

Using pointers to catch exceptions is way faster, because it doesn't copy the object. However, programmers can easily forget to make them static. If they forget to make them static, if will cause the exception to fail because it leaves the scope after it is thrown.

	void someFunction()
	{
		static exception ex;
		throw &ex;
	}
	void doSomething()
	{
		try
		{
			someFunction();
		}
		catch(exception *ex) { ... }
	}

Creating exception objects on the heap is problematic because there is a slicing problem, a derived object will be catched as the base class and it's also troublesome to decide at which level the object should be deleted.

	class Exception
	{
	public:
		virtual const char *what() throw();
	};

	class runtimeError : public exception { ... };

	void someFunction()
	{
		if (true)
		{
			throw runtimeError();
		}
	}

	void doSomething()
	{
		try
		{
			someFunction();
		}
		catch(exception ex)
		{
			std::cerr << ex.what(); // calls what() of the base class instead of the one in runtimeError
		}
	}

You can avoid all the problems above if you catrch exceptions by reference:

	void someFunction() { ... }
	void doSomething()
	{
		try { ... }
		catch(exception& ex)
		{
			std::cerr << ex.what(); // calls the right what()
		}
	}

### 14. Use exception specifications judiciously

An exception specification refers to a function specifiying the only types of exceptions that can be thrown:

	extern void f1(); // Any type of exceptions can be thrown
	void f2() throw(int); // Only ints can be thrown
	
	void f2() throw(int)
	{
		f1(); // Legal even though f1 might throw something besides an int
	}

This is made more obvious when using templates:

	templace <class T>
	bool operator==(const T& lhs, const T& rhs) throw()
	{
		return &lhs == &rhs;
	}

This template defines an operator function for pairs of any type, if they have the same adress, it returns true, otherwise it returns false. Such a function alone may not throw an exception, but if the operator& is overloaded, it may throws an exception, which violates the exception rule and makes the program halt unexpectetly.

There are three ways to prevent the program from terminating unexpectantly:

Replace all unexpected exceptions with Unexpected-Execption-Objects:

	class UnexpectedException{}; 
	void convertUnexpected()
	{
		throw UnexpectedException(); // If an unexpected exception is thrown, this function will be called
	}
	set_unexpected(conveertUnexpected);

Replace the unexpected function:

	void convertUnexpected()
	{
		throw;
	}
	set_unexpected(convertUnexpected);

An exception specification should carefully consider whether the behavior it brings is what we want before adding it.

### 15. Understand the cost of exception handling

The overhead of try catch blocks usually involve a 5-10% speed reduction and add the same amount to the code size.

### 16. Remember the 80-20 rule

20% of the code consumes 80% of the program resources, running time, memory disk and 80% of the mainenance is invested in 20% of the code. The profiler tool can be used to analyze the program

### 17. Consider using lazy evaluation

An example of delayed computation:

	class String { ... };
	String s1 = "Hello";
	String s2 = s1;

Read and write operations using lazy computation:

	String s = "Homer's Illiad;
	std::cout << s[3];
	s[3] = 'x';

The operator[] is called first to read part of the string, but the function is called a second tiome to complete the write operation. The read efficiency is high, and the write efficiencey is low because of the need for copying. At this time, the decision of whether to read or write can be postponed.

Put the data in the memory and the database, and only update the memory when it's used.

Dalay expression:

	Matrix<int> m1(1000, 1000), m2(1000, 1000);
	m3 = m1 + m2;
	// Because the addition of the matrices is so computationally expensivem you can just calculate what you really need

### 18. Amorize the cost of expected computation

If max and mnin functions are called frequently, the min and max can be cached as m_min and m_max, so that they can be returned directly each time they are called, and there would be no need to recalculate.

Prefetching is another method. When reading data from disk, read the whole block or an entire sector of data at a time, because reading a large block at a time, is faster than reading several small blocks at different times.

### 19. Understand the origin of temporary objects

Temporary objects in the usual sense refer to temp in temp = 1; a = b; b = temp; but in C++ temporarty objects refer to those invisible things such as:

	size_t countChar(const string& str, char ch);
	char buffer[MAX_STRING_LEN];
	std::cout << countChar(buffer, c);

For the call of countChar, buffer is a char data type, but its formal parameter is const string, then you need to create a temporary object of the string type, and then use buffer as a parameter to initialize this temporary object.

In addition, when it comes to operator+ overloaded functions, the return value of the function is temporary because it isn't named.

So whenever you see a constant reference parameter in a function, there is the possibility of it creating a temporary object.

### 20. Facilitate the return value opimization

A function that returns an entire object is inefficient because it needs to call the obnject's constructor and destructor.
But sometimes the comnpiler will optimize the code.

	inline const Rational operator*(const Rational& lhs, const Rational& rhs)
	{
		return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
	}

At first glance, it seems that a temporary object of the type Rational will be created, but the compiler will optimize this temporary object, so that the overhead of construction and destruction are eliminated, and the inline will also reduce the function call overhead.

### 21. Overload to avoid implicit type conversions

	// old code
	class UPInt
	{
	public:
		UPInt();
		UPInt(int value);
	}

	const UPInt operator+(const UPInt& lhs, const UPInt& rhs);

	// new Code
	const UPInt operator+(const UPint& lhs, const UPint& rhs);
	const UPInt operator+(const UPint& lhs, int rhs);
	const UPInt operator+(int lhs, const UPint& rhs);

### 22. Consider using op= instead of stand-alone op

Operator+ is different from Operator+=, so if you want to overload the + sign, it is better to overload +=, than the + sign to implement +=, you can also use templates.

	template <class T>
	const T operator+(const T& lhs, const T& rhs)
	{
		return T(lhs) += rhs;
	}

	template <class T>
	const T operator-(const T& lhs, const T& rhs)
	{
		return T(lhs) -= rhs;
	}

### 23. Consider using other equivalent libraries

Both the stdio and iostream libraries have input and output functions, but the stdio library is faster and iostream is safer. You should use alternative libraries if it helps with your codes requirements.

### 24. Understand the costs of virtual functions, multiple nheritance, virtual base classes and RTTI

The fetures you use of C++ and the compiler will greatly affect the efficiency of the program, so it is necessary to know what the compiler does behind a C++ feature

For example virtual functions, the type of pointers to objects or references is not important, most compilers use virtual tables (vtbl) and virtual table pointers (vptr) for implementing those.

vtbl:

	class C1
	{
	public:
		C1();
		virtual ~C1();
		virtual void f1();
		virtual int f2(char c) const;
		virtual void f3(const String& s);
		void f4() const;
	};

The virtual table of vbtl is similar to the following, only the virtual functions are in it.

	 ___
	|___| → ~C1()
	|___| → f1()
	|___| → f2()
	|___| → f3()

If according to the above, each virtual function needs space and has an adress, then if a class with a large number of virtual functions will need a large number opf addresses to store these things, where this vtbl is places varies depending on the compiler.

vptr:

	 __________
	|__________| → data for class
	|__________| → stores the vptr

Only one pointer is stored per object, but when the objects are small, more than one vptr will seem to take up a lot of space. When using vptr, the compiler will first find the corresponding vtbl through the vptr, and then start to find the pointed function through vtbl.

	pC1->f1();

is technically:

	(*pC1->vptr[i])(pC1);

When using multiple inheritance, the vptr will take up a lot of space, so you should try to not use multiple inheritance wherever possible.

RTTI:
Allows us to find the class information of objects at runtime, so there must be a place to store this information. This feature can also be implemented using a vtbl, adding an invisible data member type_info to each object to store these things, alas it can take up a lot of space.

### 25. Virtualize constructors and non-memeber functions

	class NewsLetter
	{
	private:
		static NLComponent *readComponent(istream& str);
		virtual NLComponent *clone() const = 0;
	};

	NewsLetter::NewsLetter(istream& str)
	{
		while(str)
		{
			components.push_back(readComponent(str));
		}
	}

	class TextBlock : public NLComponent
	{
	public:
		virtual TextBlock* clone() const
		{
			return new TextBlock(*this);
		}
	};

Readfunction is a function that behaves like a constructor, because it can create new objects we cal;l it a virtual constructor.

clone() is called a virtual copy constructor, which is equivalent to copying a new object

	NewsLetter::NewsLetter(const NewsLetter& rhs)
	{
		while (str)
		{
			for (list<NLComponent*>::const_iterator it = rhs.component.begin(); it != rhs.component.end(); ++it)
			{
				components.push_back((*it)->clone());
			}
		}
	}

### 26. Limit the number of objects of a class

The easiest way to liomit the number of objects, is to make the constructor private

	class Printer
	{
	public:
		friend Printer& thePrinter();
	priavate:
		Printer();
		Printer(const Printer& rhs);
	};

	Printer& thePrinter()
	{
		staticPrinter p;
		return p;
	}

The constructor of the Printer class is private, which can prevent hte creation of objects. The global function thePrinter is declared asa  afriend of the class, allowing thePrinter to avoid the restrictions caused by the private constructor.

There is also another way of limiting the number of objects, you can add a static variable named numObjects to record the number of objects. Of course this method will have problems when inheritance occurs e.g. when Printer and a colorPrinter inherit from Printer exist at the same time, the number of numObjects will exceed the number of maxObjects.

A better way is to encapsulate all the counting functions at once, if you have a large number of classes like Printer that need to be limited, and you need to ensure that each instance counting class has an iosolated counter. You should use a template for this.

	template <class BeingCounted>
	class Counted
	{
	public:
		class TooManyObjects{};
		static int objectCount()
		{
			rteturn numObnjects;
		}
	protected:
		Counted();
		Counted(const Counted& rhs);
		~Counted()
		{
			--numObjects;
		}
	private:
		static int numObjects;
		static const size_t maxObjects;
		vopid init()l
	};

	template <BeingCounted
	Counted<BeingCounted>>::Counted()
	{
		init();
	}

	template <class BeingCounted>
	Counted<BeingCounted>::Counted(const Counted<BeingCounted>&)
	{
		init();
	}

	template <class BeingCounted>
	void Counted<BeingCounted>::init()
	{
		if (numObjects >= maxObjects) throw TooManyObjects;
		++numObjects;
	}

	class Printer : private Counted<Printer>
	{
	public:
		static Printer* maxPrinter();
		using Counted<Printer>::objectCount;
		using Counted<Printer>::TooManyObjects;
	};

### 27. Requiring or prohibiting heap-based objects

If you want to force objects to be created on the heap you must disable implicit constrctors and destructors, then create a public destroy() method to call the destructor. If you encounter a problem with inheriting the destructor, (it can't be inherited), you could also just declare the destructor as protected.

It is impossible to distinguish whether an object is on the heap or not.

	class UPNumber
	{
	public:
		class HeapConstraintViolation{};
		static void* operator new(size_t size);
		UPNumber();
	private:
		static bool onTheHeap;
	};

This code doesn't work with new[] and delete[] so it's not optimal and shouldn't be used

Another method is to determine wheere the variable is located, because the stack goes down from high, while the heap goes up.

	bool onHeap(const void* address)
	{
		chjar onTheStack;
		return address < &onTheStack;
	}

If you want to forbit heap objects you just have to rewrite the operator new or make it private.

### 28. Smart Pointers

	template <class T>
	T* SmartPtr<T>::operator->()const
	{
		return pointee;
	}

Testing whether the smart pointer is null(don't use the following code)

	SmartPtr<TreeNode> ptn;
	if (ptn == 0) // error
	if (ptn) // error
	if (!ptn) // error

Therefore, an implicit type conversion operator is required to be able to perform the above operation

	template <class T>
	class SmartPtr
	{
	public:
		operator void*();
	};
	SmartPtr<TreeNode> ptn;
	if (ptn == 0) // works now
	if (ptn) // works now
	if (!ptn) // works now

Smart pointers and type conversions for inherited/base classes:

	class MusicProduct { ... };
	class Cassette : public MusicProduct { ... };
	class CD : public MusicProduct { ... };
	displayAndPlay(const SmartPTr<MusicProduct>& pmp, int numTimes);

	SmartPtr<Cassette> funMusic(new Cassette("1234));
	SmartPtr<CD> nightmareMusic(new CD("124));
	displayAndPlay(funMusic, 10); // error
	displayAndPlay(nightmareMusic, 0); // error

What we can see is that there is no way to convert without an implicit conversion operator:

	class SmartPtr<Cassette>
	{
	public:
		operator SmartPtr<MusicProduct>()
		{
			return SmartPtr<MusicProduct>(pointee);
		}
	};

SmartPtrs and const:

	SmartPtr<CD> p; // non const ptr non const object
	SmartPtr<const CD> p; // non const ptr const object
	const SmartPtr<CD> p = &goodCD; // const ptr non const object
	const SmartPtr<const CD> p = &goodCD; // const ptr const object

	templace <class T> // points to const object
	class SmartPtrToConst
	{
		...
	protected:
		union
		{
			const T* constPointee;
			T* pointee;
		};
	};

	templace <class T>
	class SmartPtr : public SmaartPtrToConst<T>
	{
		...
	};

### 29. Reference Counting

Basically a smart Ptr.

	template <class T>
	class Array2D
	{
	public:
		Array2D(int dim1, int dim2);
		class Array2D{
		public:
			T& operator[](int index);
			const T& operator[](int index) const;
		};
		Array1D operator[](int index);
		const Array1D operator[](int index) const;
	};

	Array2D<int> data(10, 20);
	std::cout << data[3][6]; // The [][] operator is implemented by overloading it twice

### 30. Proxy classes

The proxy class distinguisihes between reads and writes of the [] operator

Using the delayed calculation method, modify the operator[] so that it returns a proxy character, instead of the character object itself, and judge how the proxy character is used later, so as to judge whether it is a read or write operation.

	class String
	{
	public:
		class CharPrxy
		{
		public:
			CharProxy(String& str, int index);
			CharProxy& operator=(const CharProxy& rhs);
			CharProxy& operator=(char c);
			operator char() const;
		private:
			String& theString;
			int charIndex;
		};
		const CharPrxy operator[](index) const;
		CharProxy operator[](int index);
	};

### 31. Making functions virtual with respect to more than one object

	class GameObject { ... };
	class SpaceShip : public GameObject { ... };
	class SpaceStation : public GameObject { ... };
	class Asteroid : public GameObject { ... };

	void checkForCollision(GameObject& object1, GameObject& object2)
	{
		processCollision(object1, object2);
	}

When we call processCollision, the function depends on two different objects, but this function does not know the real types of object1 and object2. At this time, you need to design virtual functions based on multiple objects.

You could use virtual functions + RTTI:

	class GameObject
	{
	public:
		virtual void collide(GameObject& otherObject) = 0;
	};

	class SpaceShip : public GameObject
	{
	public:
		virtual void collide(GameObject& otherObject);
	};

	void SpaceShip::collide(GameObject& otherObject)
	{
		const type_info& objectType= typeid(otherObject);
		if (objectType == typeid(SpaceShip))
		{
			SpaceShip& ss = static_cast<SpaceShip&>(otherObject);
		} else if { ... }
	}

You can also just use virtual functions:

	class SpaceShip; // forward declaration
	class SpaceStation;
	class Asteriod;
	class GameObject
	{
	public:
		virtual void collide(GameObject& otherObject) = 0;
		virtual void collide(SpaceShip& otherObject) = 0;
		virtual void collide(SpaceStation& otherObject) = 0;
		virtual void collide(Asteriod& otherObject) = 0;
		...
	};

Simulate the virtual function table:

	class SpaceShip : public GameObject
	{
	public:
		virtual void collide(GameObject& otherObject);
		virtual void hitSpaceShip(SpaceShip& otherObject);
		virtual void hitSpaceStation(SpaceStation& otherObject);
		virtual void hitAsteriod(Asteriod& otherObject);
		...
	};

Initializing the simulated function table:

	class GameObject
	{
	public:
		virtual void collide(GameObject& otherObject) = 0;
		...
	};

	class SpaceShip : public GameObject
	{
	public:
		virtual void collide(GameObject& otheROject);
		// functions now all take GameObject as a parameter
		virtual void hitSpaceShip(GameObject& spaceShip);
		virtual void hitSpaceStation(GameObject& spaceStation);
		virtual void hitAsteriod(GameObject& asteriod);
		...
	};

	SpaceShip::HitMap* SpaceShip::initialzeCollisionMap()
	{
		HitMap *phm = new Hitmap;
		(*phm)["SpaceShip"] = &hitSpaceShip;
		(*phm)["SpaceStation"]= &hitSpaceStation;
		(*phm)["Asteroid"] = &hitAsteroid;
		return phm;
	}

### 32. Program in the future tense

New functions will be added to the function library, new overloads will appear in the future so you should be aware of how ambiguous function calls behave as a result. New classes will be added to the inheritance, new environments to run in etc.

### 33. Make non-leaf calsses abstract

	class Animal
	{
	public:
		virtual Animal& operator=(const Animal& rhs);
		...
	};

	class Lizard : public Animal
	{
	public:
		virtual Lizard& operator=(const Animal& rhs);
	};

	class Chicken : public Animal
	{
	public:
		virtual Chicken& operator=(const Animal& rhs);
	};

There will be type conversions and assignments that we usually don't want to happen:

	Animal *pAnimal1 = &liz;
	Animal *pAnimal = &chick;
	*Animal1 = *pAnimal2;

But we also hope that the following operations are feasible:

	Animal *pAnimal1 = &liz1;
	Animal *pAnimal2 = &liz2;
	// assigning a lizard to a lizard

The easiest way to solve this problem is to use dynamic_cast for type checking, but another way is to make Animal an absttract class or create an AbstractAnimal class:

	class AbstractAnimal
	{
	protected:
		AbstractAnimal& operator=(const AbstractAnimal& rhs);
	public:
		virtual !AbstractAnimal() = 0;
	};

	class Animal : public AbstractAnimal
	{
	public:
		Animal& operator=(const Animal& rhs);
	};

	class Lizard : public AbstractAnimal
	{
	public:
		virtual Lizard& operator=(const Animal& rhs);
	};

	class Chicken : public AbstractAnimal
	{
	public:
		virtual Chicken& operator(const Animal& rhs);
	};

### 34. Understand how to combine C++ and C in the same program

The compiler gives C++ and C different prefixes. In C, because there is no function overloading, the compiler doesn't specifically chjange the names of functions, but in C++, the compiler gives different names to function.

C++'s extern can disbale name mangling

	extern 'C'
	void drawLine(int x1, int y1, int x2, int y2);

In C++, static clas objects and definitions are exceuted before main is executed. In the compiler, this approach is usually to call a function by default in main.

	int main(int argc, char *argv[])
	{
		performStaticInitialization();

		realmain();

		performStaticDestruction();
	}

C++ uses new and delete, C uses malloc and free

C has no way of knowing about C++ features and data structures

Make sure C++ and C compilers produce compatible obj files, declare functions that are used in both languages as extern 'C', and whenever possible, use delete for new, and free for malloc

### 35. Familiarize yourself with the C++ language standard

Get familiar with STL and some new C++ features.

Almost anything in the STL is a template, and almost everything is in the namespace std

## Effective Modern C++

### 1. Understand template type deduction

The type deduced for T is dependent not just on the type of expression, but also on the form of ParamType. There are three cases:

- Pointer or reference
- Universal reference
- Neither a pointer nor a reference

	template <typename T>
	void f(T& param); // param is a reference

	int x = 27;
	const int cx = x;
	const int& rx = x;

	f(x); // T's and param's types are both int
	f(cx); // T's and param's type are still both int
	f(rx); // T's and param's type are still both int

	template <typename T>
	void f(T&& param); // param is now a generic reference

	template <typename T>
	void f(T param); // param is now pass-by-value

If you use an array or a function pointer to call, the template will automatically abstract into a pointer. If the template itself is the first case (pointer or reference), then it is automatically compiled into an array.

### 2. Understand auto type deduction

Type type of the auto keyword is similar to that of an template, and auto is equivalent to the T in the template:

	auto x = 27; // neither ptr nor reference
	const auto cx = x; // ditto
	const auto& rx = x; // rx is a non generic reference

	auto&& uref1 = x; //  uref1 is of type int
	auto&& uref2 = cs; //  uref2 is of type const int&
	auto&& uref3 = rx; // uref3 is of type int&&

At the time of braced initalization, the type of auto is an instance of std::initializer+list, but the same type when initialized by a template results in an error.

	auto x = { 11, 23, 9 }; // x is of type initializer_list
	
	template <typename T>
	void f(T param);
	f({ 11, 23, 9}); // error

	template <typename T>
	void f(std::initializer_list<T> initList);
	f({ 11, 23, 9});

### 3. Understand decltype

	template <typename container, typename index> // works but requires refinements
	auto authAndAccess(container& c, index i) - decltype(c[i])
	{
		authenticateUser();
		return c[i];
	}

If you want to return a reference, you need to rewrite the code above into the following:

	template <typename container, typename index>
	decltype(auto) authAndAccess(container&& c, indedxd i)
	{
		authenticateUser()l
		return std::forward<container>(c)[i];
	}

Some interesting application of decltype:

	decltype(auto) f2()
	{
		int x = 0;
		return x; // returns int
	}

	decltype(auto) f2()
	{
		int x = 0;
		return (x); // returns int&
	}

### 4. Know how to view deduced types

You can use the IDE to view the type, but the data types reported by the compiler are not necessarily correct, so you still need to be familiar with the types of the C++ standard.

### 5. Prefer auto to explicit type deduction

When auto is not initalized, an error will be reported from the compiler stage, it can make the lambda expression more stable, faster, require less resources, avoid trhe problem of type truncatioon and the ambiguity caused by variable delcation:

	std::vector<int> v;
	unsigned sz = v.size();
	auto s2 = v.size();

### 6. Use the explicitly typed initializer ideom when auto deduces undesired types

	std::vector<bool> features(const Widget& w);
	Widget w;
	auto highPriority = features(w)[5];

	processWidget(w, highPriority);

When using:

	bool highPriority = features(w)[5];

There is also a way to force it to become bool:

	auto highPriority = static_cast<bool>(features(w)[5]);

### 7. Distinguish between () and {} when creating objects

Braced {} initialization is the most widely usable initialization syntax, it prevents narrowing conversions, and it's immune to C++'s most vexing parse

	double x, y, z;
	int sum{x + y + z}; // results in an error because braces forbid implicit conversions

Parentheses () and the equal sign = are most often unusable, and prentheses can easily be mistaken as function. And those two don't type check

	class Widget
	{
		int x{0};
		int y = 0l
		int z(0); // error
	};

	std::atomic<int> ai1(0);
	std::atomic<int> ai2 = 0; // error

Another difference between curly braces and parentheses is that with std::initializer_list, braced initialization calls the constructor/function with the initializer_list:

	class Widget
	{
	public:
		Widget(int i, bool b);
		Widget(std::initializer_list><long double> il);
	};

	Widget w1(10, true); // calls first constructor
	Widget w2{10, true}; // calls second constructor

### 8. Prefer nullptr to 0 and NULL

Using nullptr can not only avoid some ambiguities, but also make the code clearer, and the most important thing is that nullptr can't be interpreted as an int, which avoids many problems.

### 9. Prefer alias declarations to typedefs

Alias declarations can make function pointers easier to understand:

	// FP is equivalent to a function pointer whose arguments are of type int and const std::string with no return value
	typedef void (*FP)(int, const std::string&);

	using FP = void (*)(int, const std::string&); // Alias declaration

And alias declarations allow for alias templates, while typedefs don't

	template <typename T>
	using MyAllocList = std::list<T, MyAlloc<T>>; // Equivalent to std::list<T, MyAlloc<T>>

	MyAllocList<Widget> lw;

Template aliases also avoid the ::type suffix, and in templates, typedefs also often require the typename prefix:

	template <class T>
	using remove_const_t = typename remove_const<T>::type;

### 10. Prefer scoped enums to unscoped enums

	enum Color { blac, white, red};
	auto white = fasle; // error because white has already been declared in this scope

	enum class Color { black, white, red};
	auto white = false; // fine because there's no other "white" in this scope

A C++98-style enum is an enum without scope

Enumaterion elemts of a scoped enumerator are only visible inside the enumerator and can only be converted to other types by type casting.

Bot scoped and unscoped enums support specifying potential types. The default latent type of a scoped enum is int. An unscoped enum has no default underlying type.

A scoped enum can always be forward-declared. An unscoped enum can only be forward-declared if the underlying type is specified.

### 11. Prefer deleted functions to private undefined ones

	template <class charT, class traits = char_traits<charT>>
	class basic_ios : public ios_base
	{
	public:
		basic_ios(const basic_ios&) = delete;
		basic_ios& operator=(const basic_ios&) = delete;
	};

The deleted function can't be used in any way. even by other memeber or friend functions, but if it's only private the compiler will complain that it's private

Another advantage of delete is that any function can be deleted, but only member functions can be private:

	bool isLucky(int num);
	bool isLucky(char) = delete; // reject the char type

If the code above is merly declared private it can be overloaded

### 12. Declare overriding functions override

Override occurs only when the virtual function of the base class and the subclass are exactly the same. If they are not exactly the same, they will be overloaded which can result in a lot of errors:

	class Base
	{
	public:
		virtual void doWork();
	};

	class Derived : public Base
	{
	public:
		virtual void doWork();
	};

	class Derived : public Base
	{
	public:
		virtual void doWork()&&;
	};

That's why you should add "override" after the function declaration

### 13. Prefer const_iterators to iterators

const_iterator wasn't very easy to use in C++98, but it's very convient in C++11

### 14. Declare functions noexcept if they won't emit exceptions

Because for the exception itself, whether an exception will occur or not is often what people care about, but what kind of exception is not so important, so noexcept and const are equally important informations and adding the noexcept keyword will optimize your code at compile time.

For functions like swap that require exception checking, as well as move operator functions, memory releases and destructors, putting noexcept in your code will greatly improve its effciency, but you have to ensure that the function does not throw an exception.

### 15. Use constexpr whenever possible

You should use constexpr if the value represented is not only const, but also when its value is known at compile time.

	constexpr auto arraySize = 10;
	std::array<int, arraySize> data;

But for consexpr to work all the functions arguments have to be known at compile time. Using consexpr will improve the efficiency of the program imensly.

### 16. Make const member functions thread safe

	class Polynomial
	{
	public:
		using RootsType = std::vector<double>;
		RootsType roots() const
		{
			if (!rootsAreValid)
			{
				rootsAreValid = true;
			}
			return rootVals;
		}
	private:
		mutable bool rootsAreValid{false};
		mutable RootsType rootVals{};
	};

Although roots is a const member function, the member variables are mutable and can be changed. If this is done, thread safety can't be achieved. At this time, only the mutex can be added

	std::lock_guard; 
	std::mutex g(m); 
	mutable std::mutex m;

Of course, in addition to the above approach of adding a mutex, a cheaper approach is to perform std::atomic operations:

	class Point
	{
	public:
		double distanceFromOriginal() const noexcept
		{
			++callCount;
			return std::sqrt((x * x) + (y * y));
		}
	private:
		mutable std::atomic<unsigned> callCount{0};
		double x, y;
	};

### 17. Understand special member function generation

The sepcial member functions are those compilers may generate on their own:
- default constructor
- default destructor
- copy operations
- move operations

Move operations are generated only for classes lacking explicitly declared move operations, copy operations and a destructor

The copy constructor is geenerated only for classes lacking an explicitly declared copy constructor, and it's deleted if a move operation is declared. The copy assignment operator is generated only for classes lacking an xplicitly declared copy assignment operator, and it's deleted if a move operator is declared. Generation of the copy operations in classes with an explicitly declared destructor is deprecated.

Member function templates never suppress generation of special member functions

### 18. Use std::unique_ptr for exclusive ownership resource management

- std::unique+ptr is a small, fast, move-only smart pointer for managhing resources with exclusive-ownership semantics

- By default, resource destruction takes place via delete, but custom delters can be specified. Stateful deleters and function pointers as deleters increase the size of std::unique_ptr objects.

- Converting a std::unique_ptr to a std::shared_ptr is easy

### 19. Use std::shared_ptr for shared ownershiip resource management

- std::shared_ptr is twice the size of a native pointer, because they contain a reference count in addition to a native pointer
- Reference-counted memory must be dynamically allocated, and using make_shared to create a shared_ptr will avoid the overhead of dynamic memory
- Incrementing and decrtementing a reference count must be atomic
- std::shared_ptr provides the convenience of automatic garbage collection in order to manage shared memory managment of arbitrary resourcess
- std::shared_ptr is twice as large as std::unique_ptr, in addition to control blocks, there is also the overhead caused by the need for atomic reference counting operations
- The default destruction of resources is generally carried out by delete, but custom deleters are also supported. The type of deleter has no effect on the type of std::shared_ptr
- You should avoid creating std::shared_ptr from raw pointer variables

### 20. Use std::weak_ptr for std::shared_ptr like pointers that can dangle

weak_ptr is usually created by a std::shared_ptr, they point to the same place, but weak_ptr does not affect the reference count of sharted_ptr:

	auto spw = std::make_shared<Widget>(); // RC (ref count) is 1

	std::weak_ptr<Widget> wpw(spw); .. wpw points to same Widget as spw but the RC remains 1

	spw = nullptr; // RC goers to 0 and the Widget is destroyed. wpw now dangles

std::weak_ptrs that dangle are said to have expired. You can test for this directly:

	if (wpw.expired()) ... // if wpw doesn't point to an object

So although weak_ptr looks useless, it has an application e.g. for caching

	std::shared_ptr<const Widget> fastLoadWidget(WidgetID id)
	{
		static std::unordered_map<WidgetID, std::weak_ptr<const Widget>> cache;

		auto objPtr = cache[id].lock(); // objPtr is std::shared_ptr to cached object or null if object's not in cache

		if (!objPtr) {
			// if not in cahce, load it cache it
			objPtr = loadWidget(id);
			cache[id] = objPtr;
		}
		
		return objPtr;
	}

- std::weak_ptr is used to mimic dangling pointers
- Potential uses of std::weak_ptr include caches, watcher lists, preventing std::shared_ptr from forming rings

### 21. Prefer std::make_unique and std::make_shared to direct use of new

- Compared to direct use of new, make functions eliminate source code duplication, improve exception safety, and, or std::make_shared and std::allocate_shared, generate code that's smaller and faster

- Situations where use of make functions is inappropriate include the need to specify custom delters and a desire to pass braces initializers

- For std::shared_ptrs, additional sitations where make functions may be ill advised include, classes with custom memory managment and systems with memory concerns, very large objects, and std::weak_ptr that outlive the corresponding std::shared_ptrs

### 22. When using the Pimpl idiom define special member functions in the implementation file

To reduce the number of builds, we can write a pimpl class, to replace the object's member variables with a pointer to an already implemented class

	// Widget.h
	/*
		If you have to include a lot of headers, it increases
	*/
	class Widget
	{
	public:
		Widget();
		~Widget();
		...
	private:
		struct Impl; // declare implementation struct and pointer to it, for the Objects that we need
		std::unqiue_ptr<Impl> pImpl;
	};

Now we dynamically allocate and deallocate the data members that used to be in the original class:

	// Widget.cpp
	#include "Widget.h"
	#include "Gadget.h"
	#include <string>
	#include <vector>

	struct Widget::Impl
	{
		std::string name;
		std::vector<double> data;
		Gadget g1, g2, g3;
	};

	Widget::Widget()
	: pImpl(std::make_unique_ptr<Impl>())
	{}

	Widget::~Widget()
	{}

Client code:

	Widget w1;
	auto w2(std::move(w1)); // move construct w2

	w1 = std::move(w2); // move assign w1

- Th ePimpl Idiom decreases build times by reducing compilation dependencies between class clients and class implementations

- For std::unique_ptr pImpl pointers, declare special member functions in the class header, but implement them in the implementation file. Do this even if the default function implementations are acceptable

- The above advice applies to std::uniqyue_ptr, but not to std::shared_ptr

### 23. Understand std::move and std::forward

Move does not move anything, and forward does not forward anything. At runtime no exceutable code is generated. These two are just functions that perform casts. std::move unconditionally casts its paramters into a rvalue, and forward will perform its casting only for references to which rvalues have been bound. The following is the pseudo code of std::move:

	template <typename T>
	decltype(auto) move(T&& param)
	{
		using ReturnType = remove_reference_t<T>&&;
		return static_cast<ReturnType>(param);
	}

- std::move performs an unconditional cast to an rvalue. In and of itself, it doesn't move anything.
- std::forward casts its argument to an rvalue only if that argument is bound to an rvalue.
- Netiher std::move nor std::forward do anything at runtime

### 24. Distinguish universal references from rvalue references

Universal references and rvalue references can be mistaken for each other in code (T&&):

Rvalue reference:

	void f(Widget&& param);
	Widget&& var1 = Widget();

	template <typename T>
	void f(std::vector<T>&& param);

Universal reference:

	template <typename T>
	void f(T&& param);

	auto&& var2 = var1;

  - If the form of the type declaration isn't precisely type&&, or if type deduction does not occur, type&& denotes an rvalue reference.
- Universal references correspond to rvalue references, if they're initialized with rvalues. They correspond to lvalue references if they're initialized with lvalues.

### 25. Use std::move on rvalue refernces, std::forward on universal references

Rvalue references are only bound to objects that can be moved. If the parameter type is an rvalue reference, the object bound to it should be movable.

- Apply std::move to rvalue reference and std::forward to universal references the last time each is used
- Do the same thing for rvalue references and universal references being returned from functions that return by value
- Never apply std::move or std::forward to local objects if they would otherwise be eligble for the return optimization

	Widget makeWidget()
	{
		Widget w;
		return w;
	}

	Widget makeWidget()
	{
		Widget w;
		return std::move(w); // causes negative optimization
	}

The compiler will enable return value opimization (RVO), this optimization needs to meet two conditions:

- The local object type is the same as the return type of the function
- Returns the local object itself

std::move and std::forward should not be used if the local object can be optimized usign RVO

### 26. Avoid overloading on universal references

Universal references will produce functions that exactly match the calling function:

	template <typename T>
	void log(T&& name) {}

	void log(int name) {}

	short a;
	log(a);

An exact matching log method will be generated, and then the template function will be called.

The universal reference template will also compete with the copy constructor.

	class Person
	{
	public:
		template<typename t> explicit PErson(T&& n): name(std::forward<T>(n)) {}
		explicit Person(int idx);

		Person(const Person& rhs);
		Person(Person&& rhs);
	};

	Person p("Nancy);
	auto cloneOfP(p);

### 27. Familiarize yourself with alternatives to overloading on universal references

- Avoid overloading and adopt a scheme of subsituting names
- Replace references with pass-by-value
- Use the Pimpl method
- Restrict generic reference templates

### 28. Understand reference collapsing

When an argument is passed to a function template , the decued template parameter encodes information ahout whether the argument is an lvalue or an rvalue:

	template <typename T>
	void func(T&& param);

	Widget WidgetFactory();
	Widget w;

	func(w);
	func(WidgetFactory);

In C++, reference by reference is illegal, but when the result of pushing T above is Widget&, void func(Widget& &&param), lvalue refenece + rvaluie refence will appear

The compiler itself will indeed hjave referneces to referneces.

- If either reference is an lvalue reference, the result is an lvalue reference, otherwise an rvalue refernece
- Reference folding occurs in four contexts: template instantiaton, auto type generation, creation and use of typedef and alias declarations, and decltype

### 29. Assume that move operators are not present, not cheap and not used

- Assume that move operations don't exist, are expensive and aren't used
- This assumption can't be made for code whose type or support for move semantics is known

Move before C++11 is inefficient, but the C++11 and newer versions made the move operation faster, but when you don't know for which C++ version you're writing you should shouldn't use it

### 30. Familiarize yourself with perfect forwarding failure cases

	template <typename>
	void fwd(T&& param)
	{
		f*std::forward<T(param)>;
	}

	template <typename... Ts>
	void fwd(Ts&&... param)
	{
		f(std::forward<Ts>(param)...);
	}

Perfect forwarding fails:

	f({1, 2, 3}); // no problem, {1, 2, 3} will be implicitly converted to std::vector<int>
	fwd({1, 2, 3}); // error, because to the brace-initialized variable was passed to a function template parameter of type std::iunitializer_list

	class Widget
	{
	public:
		static const std::size_t minVals = 28;
	};

	fwd(Widget::minVals); // error shouldn't be able to link, because usually references are treated as pointers, and also need to specify a piece of memory for the pointer to refer to

	void f(int (*fp)(int));
	int processValue(int value);
	int processValue(int value, int priority);
	fwd(processVal); // error the bare processVal doesn't have a type

	struct IPv4 HEader
	{
		std::uint32_t version4;
		IHL:4.
		DSCP:6,
		ECN:2,
		totalLenght:16
	};

	void f(std::size_t sz);
	IPv4Header h;
	fwd(h.totalLength); // error

- In the end, all failure sencarios boil down to the template type failing to push, or them pushing to a result that's wrong

### 31. Avoid default capture modes

When using explicit capture, you can clearly let the user know the declaration cycle of the variable:

	void addDivisorFilter()
	{
		auto divisor = computeDivisor*cal1, cal2;
		filters.emplace_back([&](int value) { return value % divisor == 0; });
	}

The best way to do this is to pass a *this* in:

	void Widget::addFilter() const
	{
		auto currentObjectPtr = this;
		filters.emplace_back([currentObjectPtr](int value)
		{
			return valuie & currentObjectPtr->advisor == 0;
		});
	}

### 32. Use init capture to move objects into closures

	auto func = [pw = std::make_unique<Widget>()]
	{
		return pw->isValidated() && pw->isArchived();
	};

	auto func = std::bind([](const std::unique_ptr<Widget>& data){}, std::make_unique<Widget>());

### 33. Use decltype on auto&& parameters to std::forward them

In C++14, we can use auto in lambda expressions:

	class SomeClass
	{
	public:
		template <typename T>
		auto operator()(T x) const
		{
			return func(normilize(x));
		}
	}

	auto f=[](auto&& x)
	{
		retuyrn func(normalize(std::forward<declytype(x)>(x)));
	}


### 34. Prefer lambdas to std::bin

- lambda expressiions are more readable, expressive and potentially more efficient
- Only in C++11 does bind still have a role to play

### 35. Prefer taskbased programming to thread based

Thread based code:

	int doAsyncWork();
	std::thread t(doAsyncWork);

Task based code:

	auto fut = std::async(doAsyncWork);

You should prefer task-based methods. Async is able to get the return value doAsyncWork and catch exceptions if there are any and more importantly with the latter approacgm you can leave the responsibility of thread managment to the STL and don't need to solve deadlocks, load balancing new platformadaptions, etc. yourself. On top of that, software threads are cross process managment threads of the operating system capable of creating more than hardware threads but are a finite resource and when a software thread isn't avalable, an exception is thrown directly, even if it's noexcept.

There are a few situations where threads are uised directly:

- Need to acccess the API of the underlyng thread implementations
- Thread optimization is required and the developed is capable
- Need to implement technieues that are not found in the C++ concurrency API

### 36. Specify std::launch::async if asynchronicity is essential

- std::launch::async: When it's specified it means that the runction f must be executed asynchronously, i.e. exectued on another thread

- std::launch::deferred means that f is only executed synchonously when the get or wait function is called. If get or wait isn't called, f isn't executed

- If you don't specify a strategy, the system will infer what strategy needs to be carried out according to its own estimation, which will bring uncertainty. So try to specify whether it is asynchonous or synchrous, when using it.

	auto fut = std::async(f);
	if (fut.wait_for(0s) == std::future_staticLLdeferred) { ... } // determine whether it's synchrounous (deferred)
	else {
		while (fut.wait_for(100ms) != std::future_status::ready) { ... }
	}


### 37. Make std::threads unjoinable on all paths

Every object of type std::thread is in two states: joinable and unjoinable

- joinable: Corresponds to the thread that is blocked or waiting for scheduling that is already running, runnable or finished at the bototm layer
- unjoinable: default construced std::thread, moved std::threadjoined stD::thread, detached std::thread
- If std::thread is joinable and then it's destroyed, it can have serious consequences, such as causign implicit joings, whcih can cause performance expections that are hard to debug and implicit detach whci can cause undefined behavior that is also hard to debu, so you want to make sure that the thread is unjoinable on all paths.

	class ThreadRAII
	{
	public:
		enum class DtorACtion
		{
			join,
			detach
		};
		ThreadRAII(std::thread&& t, DtorAction a)
		:
		action(a),
		t(std::move(t)) {} // hands over the thread to ThreadRAII Process
		~ThreadRAII()
		{
			if (t.joinable())
			{
				if (action == DtorAction::join)
				{
					t.join();
				} else {
					t.detach(); // Ensure that all paths can't go out connected
				}
			}
		private:
			DtorAction action;
			std::thread t;
		}
	};

### 38. Be aware of varing thread handle destructor behavior

Since the return value of the called function may be executed before the caller executes get, the return value of the called threead is stored in one place, so there is a shared state.

So the last return value of the shared state of a thread started in the synchronous state is kept blocking until the end of the task and the destructor of the return value will under normal circumstances, only destruct the member variable of the return value.

### 39. Consider void futures for one-shot even communication

Thread locks can be used instead of flag bits.

	while (!flag) {}

	bool flag(false);
	std::lock_guard<std::mutex> g(m);
	flag = true;

It's not a good practise to use the flag bit. IUf you use an object of type std::prmise, the above problem can be solved, but this method needs to use heap memory for shared state, and it's limited to one-time communication.

	std::promise<void> p;
	void react();
	void detect()
	{
		std::thread t([]
		{
			p.get_future().wait();
			react();
		});
		p.set_value();
		t.join();
	}

### 40. Use std::atomic for concurrency, volatile for special memory

atomic operation:

	std::atomic<int> ai(0);
	ai = 0;

volatile values:

	volatile int vi(0);
	vi = 10;

Tells the compiler that the variable that's being processed should use special memory and should not be opimized

	volatile int x;
	auto y = x;
	y = x;

### 41. Consider pass by value for copyable parameters that are cheap to move and always copied

When we write functions, we don't need to pass them by value, generally speaking, but if the parameters themselves are to be copied or moved, they can be passed by value. The cost of those three operations are as follows:

- Overloaded operation: means one copy for lvalue, and one move for rvalue
- Use universal reference: an lvalue means one copy, and rvalue means a move
- Pass by value: lvalue means one copy plus one move, rvalue means two moves

The cost of moving operations is inexpensive, and the parameters can be copied, because when theses two conditions are satisfied at the same the time, the efficiency of pass-by-value will not be too low

### 42. Consider emplacement instead of insertion

	std::vector<std::string> vs;
	vs.push_back("xyz");

Three things are happening here:
- A temporary variable temp get created, gets casted from const char to string, temp is an rvalue
- temp is passed to the rtvalue overloaded version of push_back, it constructs a copy of x in memory for the vector, creating a new object
- When push_back is executed, temp gets destructed

If emplace_back is used, no temporary object will be enerated, because emplace_back forwards it and that greatly improves the efficiency of the code

Insering is always better than inserting unless it's not possible to add a new value. It is more efficient than inserting in the following cases

- The value to be added is added to the container by construction rather than copying
- The type of the actual parameter passed is different from the type in the container itself
- The container is less likely to reject a new value added, (such as a map) due to duplication

## Effective STL

### 1. Choose your containers with care