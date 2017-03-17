# Python - Mastering OOP

## Snippets

### Chapter 1

* __init__ with static methods as #copy#

```python
class Hand5:
	def __init__( self, dealer_card, *cards ):
	self.dealer_card= dealer_card
	self.cards = list(cards)
	@staticmethod
	def freeze( other ):
		hand= Hand5( other.dealer_card, *other.cards )
		return hand
	@staticmethod
	def split( other, card0, card1 ):
		hand0= Hand5( other.dealer_card, other.cards[0], card0 )
		hand1= Hand5( other.dealer_card, other.cards[1], card1 )
		return hand0, hand1
	def __str__( self ):
		return ", ".join( map(str, self.cards) )
```		
* extending builtin class, e.g. list

```python
class Deck3(list):
	def __init__(self, decks=1):
	super().__init__()
		for i in range(decks):
		self.extend( card6(r+1,s) for r in range(13) for s in 
		(Club, Diamond, Heart, Spade) )
		random.shuffle( self )
		burn= random.randint(1,52)
		for i in range(burn): self.pop()
```

* factory function

```python
def card10( rank, suit ):
	if rank == 1: return AceCard( rank, suit )
	elif 2 <= rank < 11: 
		return NumberCard( rank, suit )
	elif 11 <= rank < 14: 
		return FaceCard( rank, suit )
	else:
		raise Exception( "Rank out of range" )`
```

### Chapter 2

* Collection __str__() and __repr__()

```python
class Hand:
def __init__( self, dealer_card, *cards ):
	self.dealer_card= dealer_card
	self.cards= list(cards)
	def __str__( self ):
		return ", ".join( map(str, self.cards) )
	def __repr__( self ):
		return "{__class__.__name__}({dealer_card!r}, {_cards_str})".format(__class__=self.__class__,_cards_str=", ".join( map(repr, self.cards) ),**self.__dict__ )
```

* The Fundamental Law of Hash(FLH)is this: objects that compare as equal have the same hash value! We can think of a hash comparison as being the first step in an equality test.
The inverse, however, is not true. Objects can have the same hash value but compare as not equal.

* equality and is check

- The "is test" is based on the id()numbers; it shows us that they are indeed separate objects
>>> c1 is c2
- As the hash values are different, they must not compare as equal. This fits the definitions of hash and equality. However, this violates our expectations for this class. The following is an equality check:
>>> print( c1 == c2 )

* overriding __eq__ and __hash for immutable objects

```python
def __eq__( self, other ):
	return self.suit == other.suit and self.rank == other.rank
def __hash__( self ):
	return hash(self.suit) ^ hash(self.rank)`  # ^ == XOR
```
	
The __hash__() method function computes a bit pattern from the two essential values using an exclusive OR of the bits that comprise each value. Using the ^ operator is 
a quick-and-dirty hash method that often works pretty well. For larger and more complex objects, a more sophisticated hash might be appropriate. Start with ziplib
before inventing something that has bugs.

```python
>>> print( id(c1), id(c2) )
4302577040 4302577296
>>> print( c1 is c2 )
False
>>> print( hash(c1), hash(c2) )
1259258073890 1259258073890
>>> print( c1 == c2 )
True
```
* Overriding definitions for mutable objects

```python
def __eq__( self, other ):
	return self.suit == other.suit and self.rank == other.rank
	# and self.hard == other.hard and self.soft == other.soft
def __hash__ = None`

>>> print( hash(c1), hash(c2) )
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'AceCard3'
```
As __hash__is set to None, these Card3objects can't be hashed and can't provide a 
value for the hash()function. This is the expected behavior. The equality test works properly, allowing us to compare cards. They just can't be 
inserted into sets or used as a key to a dictionary.

The following is what happens when we try:
```python
>>> print( set( [c1, c2] ) )
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'AceCard3'
```
We get a proper exception when trying to put these into a set.

* Making a frozen hand from a mutable hand

```python
import sys
class FrozenHand( Hand ):
	def __init__( self, *args, **kw ):
		if len(args) == 1 and isinstance(args[0], Hand):
			# Clone a hand
			other= args[0]
			self.dealer_card= other.dealer_card
			self.cards= other.cards
		else:
			# Build a fresh hand
			super().__init__( *args, **kw )
			def __hash__( self ):
			h= 0
			for c in self.cards:
			h = (h + hash(c)) % sys.hash_info.modulus
			return h
```
			
We can now use these classes for operations such as the following code snippet:

```python
stats = defaultdict(int)
d= Deck()
h = Hand( d.pop(), d.pop(), d.pop() )
h_f = FrozenHand( h )
stats[h_f] += 1
```

We've initialized a statistics dictionary, stats, as a defaultdictdictionary that can 
collect integer counts. We could also use a collections.Counter object for this.
By freezing a Hand class, we can use it as a key in a dictionary, collecting counts of 
each hand that actually gets dealt.

* __bool__

- when wrapping a list
```python
def __bool__( self ):
	return bool( self._cards )
```
- extending a list
```python
def __bool__( self ):
	return super().__bool__( self )
```
* __bytes__

The following is an implementation of __bytes__(), which returns a UTF-8 encoding of the Cardsclass, rank, and suit:
```python
def __bytes__( self ):
	class_code= self.__class__.__name__[0]
	rank_number_str = {'A': '1', 'J': '11', 'Q': '12', 'K': '13'}.
	get( self.rank, self.rank )
	string= "("+" ".join([class_code, rank_number_str, self.suit,] ) + ")"
	return bytes(string,encoding="utf8")`
```	
- decoding from bytes + parse into new Object

```python
def card_from_bytes( buffer ):
	string = buffer.decode("utf8")
	assert string[0 ]=="(" and string[-1] == ")"
	code, rank_number, suit = string[1:-1].split()
	class_ = { 'A': AceCard, 'N': NumberCard, 'F': FaceCard }[code]
	return class_( int(rank_number), suit )`
```
--> It's often better to use the pickleor jsonmodules than to invent a low-level bytes representation of an object.

* comparisons (__eq__, __ne__, __lt__, __le__, __gt__, __ge__)

Standardly the following e.g. the following four are necessary __eq__, __ne__, __lt__, __le__
The @functools.total_orderingdecorator overcomes the default limitation and 
deduces the rest of the comparisons from just __eq__()and one of these: __lt__(), 
__le__(), __gt__(), or __ge__().

```python
class BlackJackCard:
	def __init__( self, rank, suit, hard, soft ):
		self.rank= rank
		self.suit= suit
		self.hard= hard
		self.soft= soft
	def __lt__( self, other ):
		if not isinstance( other, BlackJackCard ): return NotImplemented # explicit type checking
		return self.rank < other.rank
	def __le__( self, other ):
		try: # implicit type checking
			return self.rank <= other.rank
		except AttributeError:
			return NotImplemented
	def __gt__( self, other ):
		if not isinstance( other, BlackJackCard ): return NotImplemented
		return self.rank > other.rank
	def __ge__( self, other ):
		if not isinstance( other, BlackJackCard ): return  NotImplemented
		return self.rank >= other.rank
	def __eq__( self, other ):
		if not isinstance( other, BlackJackCard ): return NotImplemented
		return self.rank == other.rank and self.suit == other.suit
	def __ne__( self, other ):
		if not isinstance( other, BlackJackCard ): return NotImplemented
		return self.rank != other.rank and self.suit != other.suit
```
		
* Implementation of comparison for objects of mixed classes

Given a definition of a total for a Handobject, we can meaningfully define comparisons between the Handinstances and comparisons between Handand int. 
In order to determine which kind of comparison we're doing, we're forced to use sinstance().

```python
def __eq__( self, other ):
	if isinstance(other,int):
		return self.total() == other
	try:
		return (self.cards == other.cards and self.dealer_card == other.dealer_card)
	except AttributeError:
		return NotImplemented
		
def total( self ):
	delta_soft = max( c.soft-c.hard for c in self.cards )
	hard = sum( c.hard for c in self.cards )
	if hard+delta_soft <= 21: return hard+delta_soft
	return hard
```	
* __del__
		
 context manager is often preferable to implementing __del__(), since `del` has some issues with non-predictability.

* Circular references and the weakref module
In the cases where we need circular references but also want __del__()to work nicely, we can use weak references. One common use case for circular references are 
mutual references: a parent with a collection of children; each child has a reference back to the parent. If a Playerclass has multiple hands, it might be helpful for a 
Handobject to contain a reference to the owning Playerclass.

```python
import weakref
class Parent2:
def __init__( self, *children ):
	self.children= list(children)
	for child in self.children:
		child.parent= weakref.ref(self)
def __del__( self ):
	print( "Removing {__class__.__name__} {id:d}".format( __
	class__=self.__class__, id=id(self)) )
```

* __new__
Instead of overriding __init__()when creating a subclass of a built-in immutable type, we have to tweak the object at the time of the creation by overriding __new__
(). The following is an example class definition that shows us the proper way to extend float:
```python
class Float_Units( float ):
	def __new__( cls, value, unit ):
		obj= super().__new__( cls, value )
		obj.unit= unit
		return obj
```

* metaclasses - preserve attr and method order

The following is the example metaclass that will retain the order of the creation of the attribute:
```python
import collections
class Ordered_Attributes(type):
	@classmethod
	def __prepare__(metacls, name, bases, **kwds):
		return collections.OrderedDict()
	def __new__(cls, name, bases, namespace, **kwds):
		result = super().__new__(cls, name, bases, namespace)
		result._order = tuple(n for n in namespace if not 
		n.startswith('__'))
		return result
```

We can use this metaclass instead of typewhen defining a new abstract superclass, as follows:
```python
class Order_Preserved( metaclass=Ordered_Attributes ):
	pass
		
class Something( Order_Preserved ):
	this= 'text'
	def z( self ):
		return False
	b= 'order is preserved'
	a= 'more text'

> Something._order
> ('this', 'z', 'b', 'a')
```

We can consider exploiting this information to properly serialize the object or provide debugging information that is tied to the original source definitions.

* Smart conversion between units see pages 99+

### Chapter 3

* Basic attribute processing

By default, any class we create will permit the following four behaviors with respect to attributes:
•  To create a new attribute by setting its value
•  To set the value of an existing attribute
•  To get the value of an attribute
•  To delete an attribute

The following is a version of Hand.split()that can detect splittable versus unsplittable hands via an optional attribute:
```python
def split( self, deck ):
	assert self.cards[0].rank == self.cards[1].rank
	try:
		self.split_count
		raise CannotResplit
	except AttributeError:
		h0 = Hand( self.dealer_card, self.cards[0], deck.pop() )
		h1 = Hand( self.dealer_card, self.cards[1], deck.pop() )
		h0.split_count= h1.split_count= 1
		return h0, h1
```
-> An optional attribute has the advantage of leaving the __init__()method relatively uncluttered with status flags. It has the disadvantage of obscuring some aspects of 
object state. This use of a try:block to determine object state can be very confusing and should be avoided.

* creating properties

A propertyis a method function that appears (syntactically) to be a simple attribute. We can get, set, and delete property values similarly to how we can get, set, 
and delete attribute values. There's an important distinction here. A property is actually a methodfunction and can process, rather than simply preserve, a reference 
to another object.

We'll take alook at two basic design patterns for properties:
•  Eager calculation: Inthis designpattern, when we set a value via a property, other attributes are also computed
```python
class Hand_Eager(Hand):
	def __init__( self, dealer_card, *cards ):
		self.dealer_card= dealer_card
		self.total= 0
		self._delta_soft= 0
		self._hard_total= 0
		self._cards= list()
		for c in cards:
		self.card = c
	@property
	def card( self ):
		return self._cards
	@card.setter
	def card( self, aCard ):
		self._cards.append(aCard)
		self._delta_soft = max(aCard.soft-aCard.hard, 
		self._delta_soft)
		self._hard_total += aCard.hard
		self._set_total()
	@card.deleter
	def card( self ):
		removed= self._cards.pop(-1)
		self._hard_total -= removed.hard
		# Issue: was this the only ace?
		self._delta_soft = max( c.soft-c.hard for c in self._cards )
		self._set_total()
	def _set_total( self ):
		if self._hard_total+self._delta_soft <= 21:
			self.total= self._hard_total+self._delta_soft
		else:
			self.total= self._hard_total
```
•  Lazy calculation: Inthis design pattern, calculations are deferred until requested via a property
```python
class Hand_Lazy(Hand):
	def __init__( self, dealer_card, *cards ):
		self.dealer_card= dealer_card
		self._cards= list(cards)
	@property
	def total( self ):
		delta_soft = max(c.soft-c.hard for c in self._cards)
		hard_total = sum(c.hard for c in self._cards)
		if hard_total+delta_soft <= 21: return hard_total+delta_soft
		return hard_total
	@property
	def card( self ):
		return self._cards
	@card.setter
	def card( self, aCard ):
		self._cards.append( aCard )
	@card.deleter
	def card( self ):
		self._cards.pop(-1)
```

The advantage of using properties is that the syntax doesn't have to change when the implementation changes. We can make a similar claim for getter/setter method 
functions. However, getter/setter method functions involve extra syntax that isn't very helpful nor informative. 

