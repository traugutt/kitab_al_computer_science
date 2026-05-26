# Python Classes — A Breezy Guide
### From the problem of organising things to the machinery that makes it possible
### Every feature introduced only when the previous feature creates a new need

---

## Chapter One: The Problem of Belonging

Imagine you are writing software for a library. You have books. Each book has a title, an author, a year, and a number of pages. You might write this:

```python
title  = "The Name of the Wind"
author = "Patrick Rothfuss"
year   = 2007
pages  = 662
```

This works for one book. But a library has thousands of books. So you reach for a dictionary:

```python
book = {
    "title":  "The Name of the Wind",
    "author": "Patrick Rothfuss",
    "year":   2007,
    "pages":  662
}
```

Better. Now you have a second book, a third, a list of books. You write functions that operate on them:

```python
def is_long(book):
    return book["pages"] > 500

def describe(book):
    return f"{book['title']} by {book['author']} ({book['year']})"
```

This seems fine until the library grows. Now you have functions scattered across your codebase, all operating on book dictionaries. Some functions expect a key called `"pages"`, others call it `"page_count"` because a different developer wrote them. Someone creates a book with `"autor"` instead of `"author"` and the code silently does the wrong thing. There is no single place that says "a book has exactly these fields and can do exactly these things."

The functions do not belong to the books. They float freely, unattached, waiting to be called with the wrong thing.

This is the problem classes solve.

---

## Chapter Two: The Class — Belonging Given Form

A class is a declaration that a certain kind of thing exists, what it contains, and what it can do. Here is the same library, with books given their proper form:

```python
class Book:
    pass
```

This is a class with nothing in it yet. But already something meaningful has happened — we have declared that a thing called `Book` exists in our program. We can now create individual books:

```python
book1 = Book()
book2 = Book()
```

`book1` and `book2` are called instances. The class is the blueprint. The instance is the building made from that blueprint. Every building made from the same blueprint is the same kind of thing, but each building is its own physical object at its own location.

Right now our buildings are empty. Let us put things in them:

```python
book1.title  = "The Name of the Wind"
book1.author = "Patrick Rothfuss"
book1.pages  = 662

book2.title  = "Dune"
book2.author = "Frank Herbert"
book2.pages  = 412
```

This is already better than floating dictionaries — `book1.title` is clearer than `book1["title"]` — but we still have the problem that no one enforces which attributes a book must have. Someone could create a Book without a title. We need a way to set up a book properly at the moment of creation.

---

## Chapter Three: `__init__` — The Setup Ceremony

Python calls a special method called `__init__` automatically whenever you create a new instance. This is where you declare what a book must have and set it up:

```python
class Book:
    def __init__(self, title, author, year, pages):
        self.title  = title
        self.author = author
        self.year   = year
        self.pages  = pages
```

Now creating a book requires all four pieces of information:

```python
book = Book("The Name of the Wind", "Patrick Rothfuss", 2007, 662)
```

If you forget one, Python tells you immediately. The book cannot exist in an incomplete state.

The word `self` is the first thing that confuses people. Let us be precise about it. When you write `book = Book("The Name of the Wind", ...)`, Python creates a new empty object and then calls `__init__` with that object as the first argument. The name `self` is just the conventional name for that first argument — the object being initialised. When you write `self.title = title`, you are saying "on this specific book object, store the title I was given."

You could name it anything. `def __init__(this_book, title, author, year, pages)` would work identically. The convention `self` exists purely so that every Python programmer's code reads the same way.

---

## Chapter Four: Methods — Functions That Know Where They Live

Now that books are properly formed, let us add the functions. But this time, we put them inside the class:

```python
class Book:
    def __init__(self, title, author, year, pages):
        self.title  = title
        self.author = author
        self.year   = year
        self.pages  = pages

    def is_long(self):
        return self.pages > 500

    def describe(self):
        return f"{self.title} by {self.author} ({self.year})"
```

These functions inside the class are called methods. The difference between a method and a plain function is exactly this: a method knows which object it belongs to, because Python passes the object as the first argument automatically.

When you write `book.describe()`, Python does something precise: it finds `describe` in the Book class, and calls it with `book` as the `self` argument. So `book.describe()` is exactly equivalent to `Book.describe(book)`. The dot notation is simply a readable shorthand.

This means the functions now belong to the data. `book.describe()` is impossible to call on the wrong thing — if you try to call it on a dictionary, Python will tell you dictionaries do not have a `describe` method.

The library code becomes coherent:

```python
books = [
    Book("The Name of the Wind", "Patrick Rothfuss", 2007, 662),
    Book("Dune", "Frank Herbert", 1965, 412),
    Book("Flowers for Algernon", "Daniel Keyes", 1966, 311),
]

long_books = [b for b in books if b.is_long()]
for b in long_books:
    print(b.describe())
```

---

## Chapter Five: Class Attributes — What All Books Share

Our library adds a new rule: books must be one of two formats — paperback or hardcover. Every book has a format. But also, there is a library-wide policy: books with more than 800 pages are considered "extended works" regardless of format.

The 800-page threshold is not specific to any one book. It belongs to the concept of a book, or perhaps to the library's policy. We should not store it 10,000 times, once per book instance. We should store it once, on the class itself:

```python
class Book:
    EXTENDED_THRESHOLD = 800    # this belongs to the class, not any instance
    VALID_FORMATS = ("paperback", "hardcover")

    def __init__(self, title, author, year, pages, format="paperback"):
        self.title  = title
        self.author = author
        self.year   = year
        self.pages  = pages
        self.format = format

    def is_extended(self):
        return self.pages > Book.EXTENDED_THRESHOLD
```

`EXTENDED_THRESHOLD` lives on the class. Every instance can read it via `Book.EXTENDED_THRESHOLD` or via `self.EXTENDED_THRESHOLD` — Python checks the instance first, then the class.

This distinction is important. An instance attribute is the book's own private data — its title, its page count. A class attribute is shared data that belongs to the type of thing, not to any specific instance of it.

The practical consequence: if you change a class attribute, every instance sees the change. If you change an instance attribute, only that one instance is affected. This becomes a source of bugs when you accidentally put mutable data — a list, a dictionary — as a class attribute and then modify it. You think you are modifying one instance's data; you are actually modifying data shared by all instances.

---

## Chapter Six: Inheritance — The Copy-Paste Problem

The library now also holds academic papers and magazines. Papers have a title, authors (plural), a journal name, and a year. Magazines have a title, a publisher, a frequency (weekly, monthly), and a year.

Without inheritance, you write three separate classes. All three have title and year. When you want to add a `describe()` method to all of them, you write it three times, slightly differently each time. When the requirement changes — all items now need an `available` flag — you update it in three places and inevitably forget one.

The underlying problem: you have a general concept (a library item) and specific variations of it. The specific variations share the general behaviour but each adds or changes something.

Inheritance says: define the general concept once, then let specific concepts say "I am one of those, plus these differences."

```python
class LibraryItem:
    def __init__(self, title, year):
        self.title     = title
        self.year      = year
        self.available = True

    def check_out(self):
        if not self.available:
            raise ValueError(f"{self.title} is not available")
        self.available = False

    def return_item(self):
        self.available = True

    def describe(self):
        return f"{self.title} ({self.year})"


class Book(LibraryItem):
    def __init__(self, title, year, author, pages):
        super().__init__(title, year)    # set up the LibraryItem part
        self.author = author
        self.pages  = pages

    def describe(self):
        return f"{self.title} by {self.author} ({self.year})"


class Magazine(LibraryItem):
    def __init__(self, title, year, publisher, frequency):
        super().__init__(title, year)
        self.publisher = publisher
        self.frequency = frequency

    def describe(self):
        return f"{self.title} ({self.frequency}), published by {self.publisher}"
```

`Book` and `Magazine` inherit everything from `LibraryItem`. They both have `check_out()`, `return_item()`, and `available` without writing any of that code themselves. They each provide their own `describe()` because the description of a book is different from a magazine. When they do provide their own version of a method that the parent also has, the child's version wins — this is called overriding.

The relationship is described as "is-a": a Book is a LibraryItem. A Magazine is a LibraryItem. This means anywhere your code expects a LibraryItem, it can receive a Book or a Magazine and everything will work — because they are LibraryItems with extra features, not something else entirely.

---

## Chapter Seven: `super()` — Cooperation Between Levels

In the example above, `Book.__init__` calls `super().__init__(title, year)`. Let us understand exactly what this does and why it matters.

`super()` does not simply mean "my parent class." It means "the next class in the method resolution order." This distinction matters only when you have multiple inheritance — a class that inherits from more than one parent — but the habit of using `super()` correctly is important from the start.

Consider what happens without it:

```python
class Book(LibraryItem):
    def __init__(self, title, year, author, pages):
        # BAD: calling parent by name directly
        LibraryItem.__init__(self, title, year)
        self.author = author
        self.pages  = pages
```

This works for simple cases but breaks with multiple inheritance. With `super()`, Python follows the method resolution order — the precise sequence in which classes are searched when looking for a method. This order guarantees that in any inheritance hierarchy, each class's `__init__` is called exactly once, in the right order, without any class being skipped or called twice.

The practical rule: always use `super()` when calling a parent method. Never call the parent class by name.

---

## Chapter Eight: `__str__` and `__repr__` — Talking to People and to Developers

Your library system is working well. Then a bug appears. A librarian calls you over to their terminal and shows you:

```
>>> book
<__main__.Book object at 0x7f8b1c234d90>
```

This is meaningless. The librarian wanted to see the book's title and author.

The problem is that Python does not know how to turn your Book into text. You have not told it. There are two distinct ways Python turns an object into text, and they serve different purposes.

`__str__` is for people. It is called when you print something, when you include it in an f-string, when you pass it to `str()`. It should be clear and readable.

`__repr__` is for developers. It is called in the interactive console, in debug output, in logging. It should ideally be a string that, if you pasted it back into Python, would recreate the object. At minimum it should uniquely identify the object.

```python
class Book(LibraryItem):
    # ... __init__ as before ...

    def __str__(self):
        # For people: clear and readable
        status = "available" if self.available else "checked out"
        return f'"{self.title}" by {self.author} — {status}'

    def __repr__(self):
        # For developers: unambiguous and reconstructable
        return f'Book(title={self.title!r}, year={self.year}, author={self.author!r}, pages={self.pages})'
```

Now:

```python
book = Book("Dune", 1965, "Frank Herbert", 412)
print(book)           # "Dune" by Frank Herbert — available
book                  # Book(title='Dune', year=1965, author='Frank Herbert', pages=412)
```

If you implement only one, implement `__repr__`. Python uses `__repr__` as the fallback for `__str__` when `__str__` is not defined, but not the other way around.

---

## Chapter Nine: The Dunder Protocol — How Python Talks to Your Objects

`__init__`, `__str__`, and `__repr__` are examples of dunder methods — methods whose names begin and end with double underscores. They are Python's way of asking your objects to participate in the language's own syntax and built-in operations.

This is a profound idea. Python does not have special cases for specific types. Instead, it defines a set of protocols — agreements — that any object can choose to participate in. When you write `len(something)`, Python calls `something.__len__()`. When you write `a + b`, Python calls `a.__add__(b)`. When you write `if something:`, Python calls `something.__bool__()`. When you write `for item in something:`, Python calls `something.__iter__()` and then repeatedly calls `__next__()` on the result.

Your classes can participate in all of these. Here is a Book that knows its own length and can be compared with other books:

```python
class Book(LibraryItem):
    # ... as before ...

    def __len__(self):
        # len(book) returns the number of pages
        return self.pages

    def __eq__(self, other):
        # book1 == book2 is True if titles and authors match
        if not isinstance(other, Book):
            return NotImplemented
        return self.title == other.title and self.author == other.author

    def __lt__(self, other):
        # book1 < book2 compares by year, enabling sorted(list_of_books)
        if not isinstance(other, Book):
            return NotImplemented
        return self.year < other.year

    def __contains__(self, word):
        # 'magic' in book checks if the word appears in the title
        return word.lower() in self.title.lower()
```

Now:

```python
books = [Book("Dune", 1965, ...), Book("Foundation", 1951, ...), Book("Neuromancer", 1984, ...)]

print(len(books[0]))           # 412 — pages
print(sorted(books))           # sorted by year
print("dune" in books[0])      # True — title contains 'dune'
```

The important thing to understand: Python did not add special cases for Book objects. It simply calls the agreed-upon methods. Your Book is participating in Python's language as a first-class citizen. This is why Python code can be so readable — the language is designed so that user-defined types feel as natural as built-in ones.

When a dunder method returns `NotImplemented` (not to be confused with raising `NotImplementedError`), it is telling Python "I don't know how to handle this comparison with that type, try asking the other object instead." This is the graceful way to handle operations between incompatible types.

---

## Chapter Ten: Properties — The Validation Problem

The library wants to enforce that book years are reasonable — not in the future, not before books existed. Someone tries to set `book.year = -500` and the system accepts it silently.

The naive solution is to write a method: `book.set_year(2007)`. But this makes the API awkward — everywhere you used `book.year = 2007` now needs to change to `book.set_year(2007)`. And you still have no protection against direct assignment to `book.year`.

A better solution would be to keep the clean `book.year = 2007` syntax but run validation code when it is used. This is what `property` does.

A property is a way to make a method look like an attribute. When someone reads `book.year`, Python calls a function you wrote. When someone writes `book.year = 2007`, Python calls a different function you wrote. The caller sees a simple attribute. You get to intercept reads and writes.

```python
class Book(LibraryItem):
    def __init__(self, title, year, author, pages):
        super().__init__(title, year)
        self.author = author
        self.pages  = pages

    @property
    def year(self):
        return self._year    # the actual value stored with an underscore

    @year.setter
    def year(self, value):
        if not isinstance(value, int):
            raise TypeError("Year must be an integer")
        if value < 1440 or value > 2100:
            raise ValueError(f"Year {value} is outside reasonable range")
        self._year = value
```

The underscore in `_year` is a convention meaning "internal implementation detail, do not touch directly." It is not enforced by Python — it is a message to other developers.

Now `book.year = -500` raises `ValueError`. But `book.year = 2007` and `print(book.year)` work exactly as before. The caller's code does not change. The API stays clean.

Properties are also used for computed attributes — attributes whose value is calculated from other data rather than stored:

```python
@property
def age(self):
    from datetime import date
    return date.today().year - self.year
```

`book.age` works like an attribute access but runs the calculation fresh each time. There is no `_age` stored anywhere.

---

## Chapter Eleven: Class Methods and Static Methods — Functions That Belong to the Type

The library needs to create books from two sources: a database record (a tuple of values) and a barcode scan (a string like `"978-0-06-093546-0|Dune|Frank Herbert|1965|412"`).

These are both ways of creating a Book, so they belong on the Book class. But they do not operate on an existing book instance — they create new ones. They are factory functions. They belong to the class itself, not to any instance.

A class method receives the class as its first argument (by convention called `cls`) instead of an instance:

```python
class Book(LibraryItem):
    # ... as before ...

    @classmethod
    def from_database_record(cls, record):
        # record is a tuple: (title, year, author, pages)
        return cls(record[0], record[1], record[2], record[3])

    @classmethod
    def from_barcode_string(cls, barcode):
        # format: "isbn|title|author|year|pages"
        parts = barcode.split("|")
        return cls(parts[1], int(parts[3]), parts[2], int(parts[4]))
```

The reason `cls` is used instead of writing `Book(...)` directly: if someone inherits from Book and creates `AudioBook(Book)`, calling `AudioBook.from_database_record(record)` will correctly create an `AudioBook` instance, not a `Book` instance. The class method works correctly for the entire inheritance hierarchy.

A static method is even simpler. It does not receive the class or the instance — it is just a plain function that happens to be logically related to the class and belongs in its namespace:

```python
class Book(LibraryItem):
    # ...

    @staticmethod
    def is_valid_isbn(isbn):
        # Validate ISBN format - doesn't need to know about any specific book
        import re
        return bool(re.match(r'^978-\d{1,5}-\d{1,7}-\d{1,7}-\d$', isbn))
```

The distinction between the three: regular methods work on instances, class methods work on the class, static methods are just utility functions that live in the class for organisational reasons.

---

## Chapter Twelve: Descriptors — The Mechanism Behind Properties

You have used properties. Now let us understand what they actually are, because the understanding unlocks something more powerful.

Consider this problem: the library system has many classes — Book, Magazine, Paper, DVD — and all of them need a `year` attribute with the same validation. With properties, you write the same getter and setter code in every class. This violates the principle of writing things once.

What if you could write the validation logic once and attach it to any class?

A descriptor is an object that controls what happens when an attribute on another object is read, written, or deleted. It implements three methods: `__get__`, `__set__`, and `__delete__`. When Python finds that a class attribute is a descriptor, it calls these methods instead of doing a normal attribute access.

A property is simply a built-in descriptor. When you write `@property`, Python creates a property object (a descriptor) and stores it as a class attribute.

Here is a reusable year validator, written as a descriptor:

```python
class YearAttribute:
    def __init__(self, min_year=1440, max_year=2100):
        self.min_year = min_year
        self.max_year = max_year
        self.name = None    # will be set when attached to a class

    def __set_name__(self, owner, name):
        # Python calls this when the descriptor is assigned to a class attribute.
        # 'name' is what the attribute is called ('year', 'publication_year', etc.)
        self.name = name
        self.storage = f'_{name}'    # where we actually store the value

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self    # accessed from the class itself, return the descriptor
        return getattr(obj, self.storage, None)

    def __set__(self, obj, value):
        if not isinstance(value, int):
            raise TypeError(f"{self.name} must be an integer")
        if not (self.min_year <= value <= self.max_year):
            raise ValueError(f"{self.name} must be between {self.min_year} and {self.max_year}")
        setattr(obj, self.storage, value)
```

Now any class can use this:

```python
class Book(LibraryItem):
    year = YearAttribute()    # one line, same validation everywhere
    # ...

class Magazine(LibraryItem):
    year = YearAttribute()    # same validation, no code duplication

class AncientManuscript(LibraryItem):
    year = YearAttribute(min_year=-3000, max_year=1440)    # different range
```

`book.year = 2007` goes through `YearAttribute.__set__`. `book.year` goes through `YearAttribute.__get__`. The book object does not know there is a descriptor. The descriptor does not know it is being used in a Book. They cooperate through the protocol.

This is how Python's own built-in features are implemented. `property`, `classmethod`, `staticmethod` are all descriptors. Understanding descriptors means understanding how Python works at the level below the surface you normally see.

---

## Chapter Thirteen: `__slots__` — The Memory Problem

The library system has grown. There are now two million book records loaded into memory for a search index. Memory usage is enormous.

By default, every Python instance stores its attributes in a dictionary — `__dict__`. This dictionary is flexible (you can add any attribute at any time) but expensive. A dictionary for even a few keys consumes several hundred bytes. Two million books means hundreds of gigabytes just for the attribute dictionaries.

`__slots__` tells Python: this class will have exactly these attributes and no others. Do not create a `__dict__` per instance. Store the attributes in a fixed, compact array.

```python
class BookIndexEntry:
    __slots__ = ('title', 'author', 'year', 'book_id')

    def __init__(self, title, author, year, book_id):
        self.title   = title
        self.author  = author
        self.year    = year
        self.book_id = book_id
```

The gain is significant. A slotted instance uses roughly 50-100 bytes. An unslotted instance uses 50-100 bytes plus 200-300 bytes for the dictionary. For two million instances, this saves roughly 400-600 megabytes.

The cost is rigidity. You cannot add new attributes at runtime. `entry.new_field = "something"` raises `AttributeError`. You also lose some flexibility with multiple inheritance — all classes in the hierarchy need to declare their slots correctly.

The rule of thumb: use `__slots__` when you will create many instances of the same class and memory is a concern. Do not use it for normal application code where flexibility matters more.

---

## Chapter Fourteen: Abstract Base Classes — The Contract Problem

The library system has grown. There are now many types of items: books, magazines, DVDs, audiobooks, digital articles. All of them must support certain operations: `check_out()`, `return_item()`, `describe()`.

The `LibraryItem` base class provides `check_out()` and `return_item()`. But `describe()` must be different for each type — a book's description includes the author, a magazine's includes the publisher. The base class cannot provide a useful default.

Right now, if a developer creates a new type and forgets to implement `describe()`, it silently inherits the base class version (or raises `AttributeError` if there is no base version). The error is discovered at runtime, when a librarian tries to describe an item.

An abstract base class says: "if you inherit from me, you must implement these methods. If you do not, Python will refuse to even instantiate your class." The contract is enforced at the moment the object is created, not when the missing method is called.

```python
from abc import ABC, abstractmethod

class LibraryItem(ABC):
    def __init__(self, title, year):
        self.title     = title
        self.year      = year
        self.available = True

    def check_out(self):
        if not self.available:
            raise ValueError(f"{self.title} is not available")
        self.available = False

    def return_item(self):
        self.available = True

    @abstractmethod
    def describe(self):
        # Every subclass MUST implement this.
        # This method has no body — it is purely a declaration.
        pass
```

Now:

```python
class DVD(LibraryItem):
    def __init__(self, title, year, director):
        super().__init__(title, year)
        self.director = director
    # Forgot to implement describe()!

dvd = DVD("Blade Runner", 1982, "Ridley Scott")
# TypeError: Can't instantiate abstract class DVD with abstract method describe
```

The error is immediate, at the point of instantiation, with a clear message. This is much better than discovering the missing method when a user tries to view a DVD's information.

Abstract base classes also enable `isinstance` checks that are based on capability rather than inheritance. Python's own `Iterable`, `Sized`, `Mapping`, and similar abstract classes from `collections.abc` work this way. An object does not need to inherit from `Iterable` — if it implements `__iter__`, `isinstance(obj, Iterable)` returns `True`. The check is about what the object can do, not what it inherits from.

---

## Chapter Fifteen: Dataclasses — The Boilerplate Problem

You have a new task: create a simple data container for a book reservation — a record that holds a book, a member, and a date. This is pure data, no complex behaviour.

Without dataclasses, you write:

```python
class Reservation:
    def __init__(self, book, member, date):
        self.book   = book
        self.member = member
        self.date   = date

    def __repr__(self):
        return f"Reservation(book={self.book!r}, member={self.member!r}, date={self.date!r})"

    def __eq__(self, other):
        if not isinstance(other, Reservation):
            return NotImplemented
        return (self.book, self.member, self.date) == (other.book, other.member, other.date)
```

This is thirty lines of tedious, error-prone boilerplate. The `__eq__` method is especially easy to get wrong — forget to include one field and you have silent incorrect comparisons.

The `@dataclass` decorator generates all of this from the type-annotated field declarations:

```python
from dataclasses import dataclass, field
from datetime import date

@dataclass
class Reservation:
    book:   Book
    member: str
    date:   date = field(default_factory=date.today)
```

Python generates `__init__`, `__repr__`, and `__eq__` automatically. The code does exactly what the longer version does, with four lines instead of thirty, and is less likely to have bugs.

Dataclasses have options for common needs:

`@dataclass(frozen=True)` creates an immutable instance — all attributes are read-only after creation, and the object becomes hashable (can be used as a dictionary key or in a set). This is the right choice for value objects — things where identity is determined by content, not by which specific object it is.

`@dataclass(order=True)` generates comparison methods — `<`, `>`, `<=`, `>=` — based on the fields in declaration order. This makes your objects sortable without writing comparison methods.

`field(compare=False)` excludes a field from equality comparisons and sorting. Useful for metadata that does not define identity.

The rule: use dataclasses for data containers. Use regular classes when behaviour is complex or when you need full control over initialisation.

---

## Chapter Sixteen: Metaclasses — The Class-Creation Problem

This chapter is for completeness. In practice you will encounter metaclasses, and understanding what they are prevents confusion. You will rarely write one.

Everything we have discussed so far is about creating instances of classes. But classes themselves are also objects — they are created from something. That something is a metaclass.

The default metaclass is `type`. When Python sees:

```python
class Book(LibraryItem):
    ...
```

It is essentially doing:

```python
Book = type('Book', (LibraryItem,), {'__init__': ..., 'describe': ..., ...})
```

`type` is called with the class name, the tuple of base classes, and a dictionary of the class body. It returns the class object.

A metaclass lets you intercept and customise this class-creation step. The classic use case is a registry — automatically recording every class that is created:

```python
class RegistryMeta(type):
    registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if bases:    # do not register the base class itself
            RegistryMeta.registry[name] = cls
        return cls

class LibraryItem(metaclass=RegistryMeta):
    pass

class Book(LibraryItem):
    pass

class Magazine(LibraryItem):
    pass

# Every subclass is automatically registered
print(RegistryMeta.registry)   # {'Book': <class 'Book'>, 'Magazine': <class 'Magazine'>}
```

This pattern appears in web frameworks (registering URL handlers), serialisation libraries (registering types), and ORM systems (Django's Model class uses a metaclass to build the database schema from class attributes).

Before reaching for a metaclass, consider `__init_subclass__`. Python 3.6 added this simpler alternative — it is a class method that the parent calls whenever a new subclass is created:

```python
class LibraryItem:
    _registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        LibraryItem._registry[cls.__name__] = cls
```

This is simpler, more readable, and covers most of the cases where you might otherwise reach for a metaclass.

---

## Chapter Seventeen: The Complete Mental Model

We have built an entire edifice. Let us stand back and see its shape.

A class is two things at once: a factory for creating objects, and an object itself. When Python sees `class Book(LibraryItem):`, it creates the `Book` object (an instance of `type`) and stores it as a name. `Book` is not a blueprint that disappears after use — it is a living object in your program with its own attributes, methods, and identity.

When you write `book = Book(...)`, Python calls `Book.__new__(Book, ...)` to create a bare instance, then calls `Book.__init__(instance, ...)` to initialise it. In almost all cases you only need `__init__`. You reach for `__new__` when you need to control the creation itself — singleton patterns, immutable types, caching instances.

When you access `book.title`, Python follows a precise lookup sequence: first check the class hierarchy for a data descriptor (something with both `__get__` and `__set__`, like `property`), then check the instance's own `__dict__`, then check the class hierarchy for non-data descriptors (like methods) and plain class attributes. This ordering explains why `property` can intercept attribute access even when the instance has a value stored directly in `__dict__` — the property is a data descriptor and wins the priority contest.

When you call `book.describe()`, Python finds `describe` in the class's dictionary (not the instance's), calls `describe.__get__(book, Book)` to get a bound method, and calls that. The binding is what makes `self` work — the function becomes bound to the specific instance.

The method resolution order governs which class's method is found first in an inheritance hierarchy. Python computes it using the C3 linearisation algorithm, which guarantees that a class always comes before its parents and the order of multiple parents is preserved. `super()` does not mean "my parent" — it means "the next class in the method resolution order." This distinction matters for multiple inheritance and is the reason cooperative inheritance works correctly.

The dunder methods are contracts with Python itself. By implementing them, your objects participate in Python's syntax and built-in functions as first-class citizens. This is why Python code can be so expressive — the language has no special cases for specific types; it simply follows the protocol.

The whole class system reduces to this: a way of binding data and behaviour together, a way of sharing behaviour across related types, and a way of plugging your types into the language's own mechanisms. Everything else is a specific answer to a specific problem that arose from trying to do those three things well.

---

## Appendix: The Quick Reference for Real Work

**When should I use a dataclass?** When you primarily need a container for data with little behaviour. For simple value objects, configuration, API response models.

**When should I use a regular class?** When behaviour is central — when the methods are more important than the data structure. When you need complex initialisation logic. When inheritance and polymorphism matter.

**When should I use a property?** When you want attribute-style access but need validation, computation, or logging. When the value is derived from other data.

**When should I use a class method?** For alternative constructors — `from_json`, `from_database`, `from_config_file`. For factory methods that need to return the correct subclass.

**When should I use a static method?** For utility functions that are logically related to the class but do not need access to the class or instance. If it could be a module-level function, ask whether it belongs in the class for organisational reasons.

**When should I use `__slots__`?** When creating many instances and memory matters. When you want to prevent accidental attribute creation.

**When should I use an abstract base class?** When you are defining a protocol that subclasses must follow. When you want `isinstance` checks based on capability.

**Why does modifying a list class attribute affect all instances?** Because class attributes are shared. The list is one object. When you do `instance.list_attr.append(item)`, you are not setting the instance's attribute — you are reading the shared class attribute and mutating it. The fix is to create a new list in `__init__`: `self.list_attr = []`.

**What does `NotImplemented` mean versus `NotImplementedError`?** `NotImplemented` (not an exception) is returned from dunder methods to say "I do not know how to do this operation with this type, ask the other object." `NotImplementedError` (an exception) is raised from abstract methods to say "this method must be implemented by a subclass."

**What does the underscore prefix convention mean?** A single underscore prefix (`_name`) means "internal implementation detail, treat as private, do not use from outside the class." A double underscore prefix (`__name`) triggers name mangling — Python renames it to `_ClassName__name` to avoid accidental override in subclasses. Neither is enforced by the language; both are agreements between developers.

---

*A class is not a bureaucratic requirement. It is the answer to the question: where does this thing belong? When you find yourself passing the same data to many functions, or copying logic between similar structures, or discovering that your code has no clear home for certain behaviour — that is the moment a class earns its place.*
