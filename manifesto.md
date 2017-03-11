# Python - Mastering OOP

## Snippets

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
	
* Collection __str__() and __repr__()

class Hand:
def __init__( self, dealer_card, *cards ):
	self.dealer_card= dealer_card
	self.cards= list(cards)
	def __str__( self ):
		return ", ".join( map(str, self.cards) )
	def __repr__( self ):
		return "{__class__.__name__}({dealer_card!r}, {_cards_str})".format(__class__=self.__class__,_cards_str=", ".join( map(repr, self.cards) ),**self.__dict__ )

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
		

