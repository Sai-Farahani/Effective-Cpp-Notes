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

Private inheritance is not an is-a relationship, i.g. some privatre members of the parent class are inaccessible to subclasses, and after private inheritance, all members of the subclass are private, which mean it's is-implemented-in-terms-of. Most of the time you should use composition instead of private inheritance.

Summarization:
- Privatye inheritance means is-implemented-in-terms-of. It's usually inferior to composition, but makes sense when a derived class needs access to protected base class members or needs to redefine inherited virtual functions.
- Unlike compositon, private inheritance can enable the empty base opimization. This can be importnat for library developers who strive to minimize object sizes.

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