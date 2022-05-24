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

